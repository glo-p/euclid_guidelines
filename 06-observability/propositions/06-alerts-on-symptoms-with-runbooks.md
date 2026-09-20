# Prop. VI.6 - Alerts page on symptoms, not causes, and every alert links a runbook

| | |
|---|---|
| **Level** | SHOULD |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A paging alert (Def. VI.13) SHOULD be defined on a symptom visible to a consumer of a contract, principally SLO burn rate (Prop. VI.4), consumer lag, or readiness failure, and SHOULD NOT page on an internal cause such as CPU, memory, a single host, or a dependency's own health. Every alert, paging or not, MUST link a runbook (Def. VI.14) owned by the team that is paged (Post. VI.3).

## Given

Def. VI.13, Def. VI.14, Def. VI.9, Post. VI.3, Post. I.4, CN 2, Prop. VI.4.

## Demonstration

What is owed to a consumer is the contract's behaviour (CN 2), so what must wake a human is a threat to that behaviour; a cause that does not produce a symptom is not yet a problem, and Post. I.4 guarantees many transient causes that never do. Paging on causes therefore produces alerts that require no action, which Def. VI.13 calls noise, and noise trains responders to ignore pages. The error budget (Def. VI.9) is the quantitative form of "a symptom that matters". A page without a runbook asks the responder to rediscover the service at three in the morning; Post. VI.3 puts the knowledge and the page in the same team, so the link is cheap to maintain. ∎ Q.E.D.

## Corollaries

* **Cor. VI.6.1** - Cause-based signals (CPU, connection pool, GC) are dashboard panels or non-paging notifications, not pages.
* **Cor. VI.6.2** - An alert that fires and needs no action twice in a month is re-tuned or deleted; the review is part of the on-call handover.

## Construction

* Datadog multi-window, multi-burn-rate monitors generated from `slo.yaml` (Prop. VI.4); SQS `ApproximateAgeOfOldestMessage` monitors per consumer queue; DLQ depth monitors (Prop. III.8).
* Monitor message template requiring a `runbook:` URL; Datadog Service Catalog `on-call` and `runbooks` fields.
* Runbooks in the service repository under `docs/runbooks/` or in the company wiki, one per alert, following the runbook template.
* Datadog On-Call (or PagerDuty) schedules per team; escalation to the principals only for cross-team incidents.

## Conformance

Terraform policy (`conftest`) rejecting a `datadog_monitor` whose message lacks a runbook URL or whose `priority` is paging without an SLO or lag query. Monthly report of pages with no follow-up action.

## Scholium

SHOULD because some services have a legitimate cause-based page (a certificate expiry, a licence limit). The written reason names the cause and why no symptom precedes it.
