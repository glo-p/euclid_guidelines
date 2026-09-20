# Prop. IX.5 - Sunset on evidence; retirement removes the contract

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A Deprecated version MUST NOT enter the Sunset stage ([Def. IX.4](../definitions.md#Def.%20IX.4%20-%20Lifecycle%20Stage)) before both the sunset date ([Def. IX.6](../definitions.md#Def.%20IX.6%20-%20Sunset%20Date)) has passed and traffic evidence ([Def. IX.9](../definitions.md#Def.%20IX.9%20-%20Traffic%20Evidence)) from the RED metrics of [Prop. VI.3](../../06-observability/propositions/03-red-metrics.md) shows zero traffic for that version for 30 consecutive days in `prod`. A Sunset version MUST be Retired by removing it from every environment, removing its routes, rules and schema entries, and marking it `retired` in the catalogue; the identifier is never reused.

## Given

* [Def. I.21](../../01-foundations/definitions.md#Def.%20I.21%20-%20Environment), [Def. IX.4](../definitions.md#Def.%20IX.4%20-%20Lifecycle%20Stage), [Def. IX.6](../definitions.md#Def.%20IX.6%20-%20Sunset%20Date), [Def. IX.9](../definitions.md#Def.%20IX.9%20-%20Traffic%20Evidence)
* [Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution), [Post. IX.2](../postulates.md#Post.%20IX.2%20-%20Minimum%20Deprecation%20Period)
* [CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed), [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated)
* [Cor. I.4.1](../../01-foundations/method.md#Prop.%20I.4%20-%20Change%20is%20by%20ADR%20against%20the%20foundations), [Prop. VI.3](../../06-observability/propositions/03-red-metrics.md), [Prop. IX.4](04-producers-know-their-consumers.md)

## Demonstration

Until the sunset date the version is owed ([CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed), [Def. IX.4](../definitions.md#Def.%20IX.4%20-%20Lifecycle%20Stage)), so the date is the earliest possible boundary. The date alone is not sufficient: consumers lag indefinitely ([Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution)), and a consumer still sending traffic on the sunset date would fail the moment the version is withdrawn, which is a producer defect. The only evidence that no consumer remains is the absence of observed traffic ([CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated)), measured by the metrics that [Prop. VI.3](../../06-observability/propositions/03-red-metrics.md) already requires and joined to consumers by [Prop. IX.4](04-producers-know-their-consumers.md); 30 days covers monthly batch consumers. Once Sunset, the version is not owed, and keeping unowed surfaces deployed contradicts [Post. IX.1](../postulates.md#Post.%20IX.1%20-%20Two%20Majors%20at%20Most)'s purpose; so retirement removes them, and [Cor. I.4.1](../../01-foundations/method.md#Prop.%20I.4%20-%20Change%20is%20by%20ADR%20against%20the%20foundations) keeps the number. ∎ Q.E.D.

## Corollaries

* **Cor. IX.5.1** - Non-zero traffic after the sunset date does not extend the obligation; it triggers direct contact with the identified consumer, and the producer MAY choose to move the sunset date later ([Def. IX.6](../definitions.md#Def.%20IX.6%20-%20Sunset%20Date)).
* **Cor. IX.5.2** - Retirement of an event type removes the producer's emission, the registry entry and the archive replay capability for that type ([Prop. III.13](../../03-events/propositions/13-archive-and-replay.md)); the archived events themselves are retained per [Book VII](../../07-data-ownership/README.md).

## Construction

* CloudWatch metric math over `RequestCount{contract, major}` and `MessagesPublished{eventType}` with a 30-day period; alarm `sunset-eligible` that flips to `OK` when the sum is zero.
* Scheduled Lambda `contract-lifecycle-advancer` that, for each Deprecated entry past its sunset date, evaluates the alarm and moves the catalogue stage to `sunset`; it never moves to `retired` on its own.
* Retirement pull request template in the service repository: delete route prefix, delete `Asp.Versioning` controller set, delete EventBridge rule and schema registry version via Terraform/CDK, set catalogue stage `retired`.
* Architecture test in `Company.Architecture.Tests`: a controller or event type whose catalogue stage is `sunset` for more than 30 days fails the build.

## Conformance

Catalogue lint rejecting a transition to `sunset` without a linked zero-traffic alarm state and a passed sunset date; architecture test for lingering Sunset artefacts.

## Scholium

"Zero traffic" is measured in `prod` because `prod` is the only environment where the contract is owed to anyone but the producing team; whether to include other environments is an open question. The 30-day window is also an open question.
