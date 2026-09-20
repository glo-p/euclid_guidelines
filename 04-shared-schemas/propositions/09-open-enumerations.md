# Prop. IV.9 - Enumerations are open unless proven closed

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

A string-valued field with a known set of values MUST be declared as an open enumeration (Def. IV.6): `type: string`, a `pattern` of `^[a-z0-9]+(-[a-z0-9]+)*$`, and `x-known-values` listing the current values with a description each. It MUST be declared closed (`enum`) only when the schema's description states why the set cannot grow (a legal or mathematical fixed set, or a value the *consumer* sends and the producer must reject if unknown). Adding a value to an open enumeration is a compatible change (Def. I.15); consumers MUST handle an unknown value without failing, typically by treating it as "other" and logging.

## Given

Def. I.14, Def. I.15, Def. IV.6, Post. I.5, CN 3, CN 4, CN 6.

## Demonstration

Almost every enumeration in a business system grows (a new order status, a new payment method). If it is closed, each growth is a breaking change (Def. I.14) and, by CN 4, a new major of every contract that embeds it, which under Post. I.5 means a migration for every consumer for the sake of one string. If it is open, growth is compatible, and CN 6 already requires consumers to tolerate what they do not know. The cost is that generated types cannot be exhaustive `switch`es, which is exactly the right cost: the consumer's code is forced to have a default branch. The set is still published (CN 3) so that consumers know what to expect today. Input enumerations are the exception because there the *producer* is the one that must reject the unknown. ∎ Q.E.D.

## Corollaries

* **Cor. IV.9.1** - Generated C# types for open enumerations are `readonly record struct` string wrappers with static known members, not `enum`. TypeScript types are `KnownStatus | (string & {})`.
* **Cor. IV.9.2** - Removing a value from `x-known-values` is a breaking change unless the value has not been emitted for the deprecation period (Book IX).
* **Cor. IV.9.3** - `Operation.status`, `Health.status`, `AuditEvent.outcome` and `EventEnvelope.dataclassification` are closed, with the reason stated in their descriptions.

## Construction

```json
"status": {
  "type": "string",
  "pattern": "^[a-z0-9]+(-[a-z0-9]+)*$",
  "x-known-values": [
    { "value": "placed", "description": "Accepted, not yet paid." },
    { "value": "paid", "description": "Payment captured." },
    { "value": "cancelled", "description": "Cancelled before fulfilment." }
  ]
}
```

## Conformance

Spectral / catalogue rule `enum-is-open-or-justified`: any `enum` keyword on an output schema must be accompanied by a description containing the phrase "closed because". Consumer contract tests (Prop. III.12) deliver an unknown value for every open enumeration.
