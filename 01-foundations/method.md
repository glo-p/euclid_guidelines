# Book I - Method

The propositions of [Book I](README.md) are about the guidelines themselves: how a rule is written, how strong it is, how it is checked and how it changes.

---

## Prop. I.1 - Every rule is a proposition with a demonstration

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

### Statement

Every normative rule in these guidelines MUST be written as a proposition using [`templates/proposition.md`](../templates/proposition.md), with a *Statement*, a *Given* list citing only definitions, postulates, common notions and earlier propositions, and a *Demonstration* deriving the statement from them.

### Given

[CN 3](common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 8](common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts), [Post. I.3](postulates.md#Post.%20I.3%20-%20Autonomy).

### Demonstration

Teams are autonomous ([Post. I.3](postulates.md#Post.%20I.3%20-%20Autonomy)) and will only follow a rule whose reason they can see; a rule with no visible derivation is a preference, and preferences are argued about indefinitely. By [CN 8](common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts) the architecture is what is written down, and by [CN 3](common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit) what is not written does not exist. Therefore each rule must be written, and written with its derivation, so that disagreement is directed at a postulate or definition rather than at the rule itself. ∎ Q.E.D.

### Scholium

The demonstration need not be long. Most are three or four sentences. What matters is that every step points at something already agreed.

---

## Prop. I.2 - Levels and their obligations

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |

### Statement

Every proposition MUST carry exactly one level ([Def. I.19](definitions.md#Def.%20I.19%20-%20Level)). A **MUST** binds every service without exception unless an exception ADR exists ([Def. I.20](definitions.md#Def.%20I.20%20-%20Exception)). A **SHOULD** binds every service unless the owning team records, in the service repository, a one-paragraph reason for deviating. A **MAY** binds no one and exists to name a sanctioned option.

### Given

[Def. I.19](definitions.md#Def.%20I.19%20-%20Level), [Def. I.20](definitions.md#Def.%20I.20%20-%20Exception), [Post. I.3](postulates.md#Post.%20I.3%20-%20Autonomy), [Post. I.6](postulates.md#Post.%20I.6%20-%20Machine%20Verification).

### Demonstration

Autonomous teams ([Post. I.3](postulates.md#Post.%20I.3%20-%20Autonomy)) need to know which rules are negotiable locally and which are not; a single vocabulary ([Def. I.19](definitions.md#Def.%20I.19%20-%20Level)) makes that unambiguous. A MUST without an exception path would be ignored silently, which [Post. I.6](postulates.md#Post.%20I.6%20-%20Machine%20Verification) tells us happens within a year; therefore MUST is paired with the recorded exception ([Def. I.20](definitions.md#Def.%20I.20%20-%20Exception)). A SHOULD with no recording requirement is indistinguishable from a MAY; therefore SHOULD requires a local, lightweight record. ∎ Q.E.D.

---

## Prop. I.3 - Every MUST has a machine check

| | |
|---|---|
| **Level** | SHOULD |
| **Status** | Accepted |
| **Since** | 2026-09-20 |

### Statement

Every **MUST** proposition SHOULD name, in its *Conformance* section, at least one automated check (OpenAPI linter rule, JSON Schema validation, architecture test, AWS Config rule, CI policy) that detects violation. A MUST with no check is marked `Draft` until one exists.

### Given

[Post. I.6](postulates.md#Post.%20I.6%20-%20Machine%20Verification), [CN 7](common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated).

### Demonstration

By [Post. I.6](postulates.md#Post.%20I.6%20-%20Machine%20Verification) an unchecked rule is violated within a year. A violated MUST that is not detected is, by [CN 7](common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated), operationally absent - the guideline says one thing and the estate does another, which is worse than having no guideline. Therefore each MUST needs a detector, and until it has one it has not earned the status *Accepted*. ∎ Q.E.D.

### Scholium

This is a SHOULD rather than a MUST because some rules (for example, "events are named in the past tense") are only partly checkable. The intent is pressure, not paralysis.

---

## Prop. I.4 - Change is by ADR against the foundations

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |

### Statement

A change to any definition, postulate, common notion or `Accepted` proposition MUST be proposed as an ADR ([Book X](../10-governance/README.md), [`templates/adr.md`](../templates/adr.md)) that names the items changed and shows that every proposition citing them still holds or is amended in the same ADR.

### Given

[Prop. I.1](method.md#Prop.%20I.1%20-%20Every%20rule%20is%20a%20proposition%20with%20a%20demonstration), [CN 4](common-notions.md#CN%204%20-%20Compatibility%20Is%20Compositional), [CN 8](common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts).

### Demonstration

Propositions cite their foundations ([Prop. I.1](method.md#Prop.%20I.1%20-%20Every%20rule%20is%20a%20proposition%20with%20a%20demonstration)), so a change to a foundation has consequences that are enumerable: exactly the propositions that cite it. By [CN 4](common-notions.md#CN%204%20-%20Compatibility%20Is%20Compositional) applied to the guideline itself, the change is sound only if each consequence is sound. By [CN 8](common-notions.md#CN%208%20-%20The%20Whole%20Is%20Not%20Greater%20Than%20Its%20Contracts) the change is only real when recorded. Therefore the change is recorded as an ADR and the ADR carries the enumeration. ∎ Q.E.D.

### Corollaries

* **Cor. I.4.1** - Identifiers are never reused. A withdrawn item is marked `Deprecated` and keeps its number, so that historical ADRs remain readable.
* **Cor. I.4.2** - A team may propose a change; only the principal engineers accept one. This is the sole point of central control in the guidelines and is deliberately narrow.
