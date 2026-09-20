# Engineering Guidelines - The Elements

> *"Let it be postulated that a contract, once published, is owed."*

This repository is the company-wide set of engineering guidelines that make a shared architecture possible across autonomous teams. It is written in the manner of Euclid's *Elements*: every rule is a **proposition** that is *demonstrated* from a small number of **definitions**, **postulates** and **common notions**. Nothing is asserted merely because someone prefers it; every rule can be traced back to something we have all agreed is true.

If you disagree with a proposition, the productive move is to find the definition, postulate or common notion it rests on and argue with *that*. If you cannot, the proposition stands.

## How to read this repository

| Term | Meaning here | Example |
|---|---|---|
| **Definition** (`Def. II.3`) | Fixes the meaning of a word so we stop arguing about vocabulary. Definitions are not rules. | "A *resource* is …" |
| **Postulate** (`Post. I.4`) | Something we accept as true about *our* world without proof: our cloud, our languages, our organisation, the nature of networks. Postulates can change when the world changes. | "All workloads run in AWS." |
| **Common Notion** (`CN 3`) | A self-evident principle of reasoning that applies to every book. | "Explicit is greater than implicit." |
| **Proposition** (`Prop. III.5`) | A rule. Each has a *Statement*, a *Demonstration* deriving it from the above, *Corollaries*, a *Construction* (how to build it in .NET / AWS) and a *Scholium* (commentary, examples, known exceptions). | "Every service publishes events through a transactional outbox." |
| **Corollary** (`Cor. III.5.1`) | A consequence that follows immediately from a proposition. | |
| **Lemma** | A small helper result proved on the way to a proposition. | |
| **Scholium** | Commentary that is *not* normative. | |

Propositions carry a **level** using RFC 2119 words: **MUST** (requirement), **SHOULD** (recommendation, deviation needs a written reason), **MAY** (option). Every proposition also carries a **status**: `Draft`, `Accepted`, `Deprecated`.

Propositions ending in **Q.E.D.** ("which was to be demonstrated") establish a rule. Propositions ending in **Q.E.F.** ("which was to be done") establish a *construction* - a concrete, reusable way of building something (a NuGet package, a Terraform module, a middleware).

## The Books

| Book | Folder | Depth | Subject |
|---|---|---|---|
| I | [`00-foundations/`](00-foundations/README.md) | Complete | Definitions, postulates and common notions shared by every other book. Method for writing propositions. |
| II | [`01-api-guidelines/`](01-api-guidelines/README.md) | **Detailed** | Synchronous HTTP/JSON APIs: naming, methods, errors, pagination, versioning, idempotency, concurrency, security, .NET construction. |
| III | [`02-events/`](02-events/README.md) | **Detailed** | Asynchronous communication: event envelope, naming, schemas, outbox, idempotent consumers, ordering, retries, EventBridge/SQS topology, .NET construction. |
| IV | [`03-shared-schemas/`](03-shared-schemas/README.md) | **Detailed** | The shared schema catalogue: Money, Problem Details, Page, Metadata, Identifier, Temporal types, Event Envelope - as JSON Schema and as C# types. |
| V | [`04-security/`](04-security/README.md) | Scaffold | Identity, authorisation, secrets, data classification. |
| VI | [`05-observability/`](05-observability/README.md) | Scaffold | Logs, metrics, traces, correlation, health, SLOs. |
| VII | [`06-data-ownership/`](06-data-ownership/README.md) | Scaffold | Bounded contexts, data ownership, reporting, PII. |
| VIII | [`07-infrastructure-aws/`](07-infrastructure-aws/README.md) | Scaffold | Accounts, networking, compute choices, IaC, environments. |
| IX | [`08-versioning-and-deprecation/`](08-versioning-and-deprecation/README.md) | Scaffold | Lifecycle of any contract: introduce, evolve, deprecate, sunset. |
| X | [`09-governance/`](09-governance/README.md) | Scaffold | How these guidelines change; ADRs; exceptions; conformance. |
| XI | [`10-frontend-integration/`](10-frontend-integration/README.md) | Scaffold | How front-ends of any technology consume the APIs and schemas above. |

Templates for new definitions, propositions and ADRs are in [`templates/`](templates/). The normative JSON Schemas are in [`03-shared-schemas/schemas/`](03-shared-schemas/schemas/README.md) and the Spectral ruleset that checks Book II is in [`rulesets/spectral/`](rulesets/spectral/README.md).

## Numbering

* Books are Roman numerals; folders carry a two-digit sort prefix.
* `Def. II.3` = third definition of Book II. `Post. I.2` = second postulate of Book I.
* Common notions are global and live only in Book I: `CN 1 … CN n`.
* `Prop. II.4` = fourth proposition of Book II. `Cor. II.4.1` = its first corollary.
* Identifiers are permanent. A withdrawn item keeps its number and is marked `Deprecated`; numbers are never reused.

## Status of this repository

| Field | Value |
|---|---|
| Created | 2026-09-20 |
| Owners | The three principal engineers (see Book X for the change process) |
| Canonical stack | AWS · .NET (current LTS) · REST/JSON over HTTP · Amazon EventBridge + SQS · JSON Schema 2020-12 · OpenAPI 3.1 |

*Each Book's `README.md` is its table of contents. Start with Book I.*
