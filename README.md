# launch-runbook

Plan and execute a client website go-live, covering DNS cutover, SSL, redirect mapping, analytics, pre-flight gates, and a written rollback plan.

An original skill by Silicon Valley Academy.

## What it does

Anyone can list launch tasks. What sinks launches is ordering: doing a slow-to-undo step before a fast-to-verify one. This skill produces a dated, T-minus runbook and enforces the parts that have to happen days early:

- Lower DNS TTL to 300 seconds, 24 to 48 hours before cutover, so your rollback window is minutes instead of a day
- Build the redirect map while the old site is still live, because after cutover you cannot enumerate what existed
- Provision SSL before pointing DNS, so there is never a reachable insecure window

It then runs blocking pre-flight gates (no stray `noindex`, MX records preserved, forms actually delivering, analytics actually firing) and requires the rollback plan in writing before anything is switched.

Output is a runbook on a T-48h through T+24h timeline that records what was done, what failed, and what was skipped.

## Install

Paste this into Claude Code:

```
Install the launch-runbook skill from github.com/sva-admin/launch-runbook
```

It triggers on phrases like "launch runbook", "go live", "DNS cutover", "launch checklist", or any time a site is about to move from staging to a real domain.

## Pairs with

Run [seo-aeo](https://github.com/sva-admin/seo-aeo) before launch, while metadata and sitemap changes are still cheap. This skill starts where your deploy workflow finishes: it owns domains and traffic, not code and CI.

## Learn free

Learn free at https://loop.sv-academy.org/tutorials

---

More skills: https://github.com/sva-admin/claude-skills
Silicon Valley Academy: https://sv-academy.org
