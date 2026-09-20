# Prop. IX.5 - Sunset on evidence; retirement removes the contract

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A Deprecated version MUST NOT enter the Sunset stage (Def. IX.4) before both the sunset date (Def. IX.6) has passed and traffic evidence (Def. IX.9) from the RED metrics of Prop. VI.3 shows zero traffic for that version for 30 consecutive days in `prod`. A Sunset version MUST be Retired by removing it from every environment, removing its routes, rules and schema entries, and marking it `retired` in the catalogue; the identifier is never reused.

## Given

* Def. I.21, Def. IX.4, Def. IX.6, Def. IX.9
* Post. I.5, Post. IX.2
* CN 2, CN 7
* Cor. I.4.1, Prop. VI.3, Prop. IX.4

## Demonstration

Until the sunset date the version is owed (CN 2, Def. IX.4), so the date is the earliest possible boundary. The date alone is not sufficient: consumers lag indefinitely (Post. I.5), and a consumer still sending traffic on the sunset date would fail the moment the version is withdrawn, which is a producer defect. The only evidence that no consumer remains is the absence of observed traffic (CN 7), measured by the metrics that Prop. VI.3 already requires and joined to consumers by Prop. IX.4; 30 days covers monthly batch consumers. Once Sunset, the version is not owed, and keeping unowed surfaces deployed contradicts Post. IX.1's purpose; so retirement removes them, and Cor. I.4.1 keeps the number. ∎ Q.E.D.

## Corollaries

* **Cor. IX.5.1** - Non-zero traffic after the sunset date does not extend the obligation; it triggers direct contact with the identified consumer, and the producer MAY choose to move the sunset date later (Def. IX.6).
* **Cor. IX.5.2** - Retirement of an event type removes the producer's emission, the registry entry and the archive replay capability for that type (Prop. III.13); the archived events themselves are retained per Book VII.

## Construction

* CloudWatch metric math over `RequestCount{contract, major}` and `MessagesPublished{eventType}` with a 30-day period; alarm `sunset-eligible` that flips to `OK` when the sum is zero.
* Scheduled Lambda `contract-lifecycle-advancer` that, for each Deprecated entry past its sunset date, evaluates the alarm and moves the catalogue stage to `sunset`; it never moves to `retired` on its own.
* Retirement pull request template in the service repository: delete route prefix, delete `Asp.Versioning` controller set, delete EventBridge rule and schema registry version via Terraform/CDK, set catalogue stage `retired`.
* Architecture test in `Company.Architecture.Tests`: a controller or event type whose catalogue stage is `sunset` for more than 30 days fails the build.

## Conformance

Catalogue lint rejecting a transition to `sunset` without a linked zero-traffic alarm state and a passed sunset date; architecture test for lingering Sunset artefacts.

## Scholium

"Zero traffic" is measured in `prod` because `prod` is the only environment where the contract is owed to anyone but the producing team; whether to include other environments is an open question. The 30-day window is also an open question.
