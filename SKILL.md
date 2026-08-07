---
name: launch-runbook
description: Plan and execute a client website go-live. Covers DNS cutover, SSL, redirect mapping, analytics and Search Console, pre-flight gates, and a written rollback plan. Trigger on "launch runbook", "go live", "going live", "DNS cutover", "launch checklist", "ship this site to the client domain", "point the domain at", or when a client site is about to move from staging to a real domain. This owns DOMAINS AND TRAFFIC. Your deploy and CI workflow owns code and builds; the two compose rather than replace each other.
---

# Launch Runbook

## Objective

Get a client site onto its real domain without losing traffic, breaking email, or shipping a `noindex` to production. Produce a dated runbook the user can follow step by step, and leave behind a record of what was actually done.

The value here is **ordering and irreversibility**, not the checklist. Anyone can list launch tasks. What sinks launches is doing a slow-to-undo step before a fast-to-verify one.

## The three steps that must happen days early

Treat these as blocking. If they have not happened, the launch is not ready regardless of how finished the site looks.

1. **Lower DNS TTL to 300 seconds, 24 to 48 hours before cutover.** TTL changes only take effect after the *old* TTL expires. Skip this and your rollback window equals the old TTL, which is often 24 hours of a dead client site. This is the single most common cause of a launch that cannot be undone.
2. **Build the redirect map while the old site is still live.** Crawl it, export every indexed URL, and map each to its new destination. After cutover you have lost the ability to enumerate what existed. Every unmapped URL that had traffic or backlinks becomes a hard 404.
3. **Provision SSL for the real domain before pointing DNS at it**, where the host allows pre-validation. Otherwise there is a window where the site is reachable and insecure, and browsers will have cached the warning.

## Pre-flight gates (each one blocks cutover)

- `noindex`, `robots.txt` disallow, and any staging password are removed. **Check the built output, not the source.** This is the classic silent launch killer: the site goes live, Google honors the noindex, and nobody notices for weeks.
- Redirect map exists and every entry resolves to a real destination, not a redirect chain.
- Analytics and any pixels fire on the production build, verified by loading a page and watching the network request, not by trusting that the tag is in the code.
- Forms actually deliver to a real inbox. Submit one.
- MX and mail records are preserved in the new DNS zone. Changing nameservers without copying MX records takes down the client's email, which is a far worse incident than a broken page.
- The rollback plan is written down (below) before anything is switched.

## The runbook you produce

Save to `docs/launches/<client>/launch-<YYYY-MM-DD>.md` (or wherever the project keeps its written record) with frontmatter `title`, `client`, `domain`, `status` (planned, executing, live, rolled-back), `created`. Structure it on a T-minus timeline, because the schedule is the product:

- **T-48h**: TTL lowered, redirect map built, SSL provisioned, DNS zone drafted with MX preserved, stakeholders told the window.
- **T-24h**: pre-flight gates run and recorded pass or fail, rollback plan written, final content freeze agreed with the client.
- **T-0**: cutover. Record the exact change made and the timestamp.
- **T+15m**: propagation spot-check from at least two networks (a phone on cellular counts), SSL valid, home page and one deep page load, one redirect verified.
- **T+1h**: full redirect sweep, forms tested live, analytics receiving real hits.
- **T+24h**: Search Console property verified on the new domain, sitemap submitted, crawl errors checked, old-domain redirects still resolving.

## The rollback plan (write before cutover, never during)

It must answer three things in advance, because during an incident nobody reasons well: what exact DNS values restore the previous state, who is authorized to make that call and at what threshold, and how long propagation will take given the current TTL. If the honest answer to the last one is "hours," the launch is not ready and you say so.

## Done looks like

- The site resolves on the real domain over HTTPS from more than one network.
- Old URLs with traffic land on their mapped destinations, not on a generic home page.
- The production build is indexable, and Search Console confirms it.
- Analytics shows real sessions.
- Client email still works.
- The runbook file records what was done, what failed, and what was skipped.

## Verify by observing, not asserting

Do not mark a step done because the config looks right. Load the page, watch the network tab, submit the form, follow the redirect. Use a headless browser (Playwright or equivalent) for automated sweeps across many URLs, a real logged-in browser session when a check needs authentication, and a broader functional QA pass on top of that. A launch is the one place where "it should work" has repeatedly meant it did not.

## Pair with

The `seo-aeo` skill (github.com/sva-admin/seo-aeo) before launch, to get metadata, structured data, and sitemap right while changes are still cheap. Whatever ship or deploy workflow you use handles the code side. This skill starts where those finish.

## When this is overkill

An internal tool, a staging URL nobody links to, or a personal project on a subdomain does not need a T-minus schedule. Say so and give the short version rather than performing the full runbook on a low-stakes deploy.
