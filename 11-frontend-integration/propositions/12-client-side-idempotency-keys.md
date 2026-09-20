# Prop. XI.12 - Idempotency keys are generated client-side for every non-idempotent submit

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A front-end MUST generate an `Idempotency-Key` ([Prop. II.8](../../02-api-guidelines/propositions/08-idempotency.md)) before every request that creates or otherwise non-idempotently changes state, MUST send the same key on every retry of that request until a definitive response is received, and MUST generate a new key only when the user intentionally initiates a new submission.

## Given

* [Def. I.16](../../01-foundations/definitions.md#Def.%20I.16%20-%20Idempotent%20Operation), [Def. XI.1](../definitions.md#Def.%20XI.1%20-%20Front-end), [Def. XI.6](../definitions.md#Def.%20XI.6%20-%20Generated%20Client)
* [Post. I.4](../../01-foundations/postulates.md#Post.%20I.4%20-%20Unreliable%20Network), [Post. XI.2](../postulates.md#Post.%20XI.2%20-%20Untrusted%20Clients)
* [Prop. II.3](../../02-api-guidelines/propositions/03-methods-and-status-codes.md), [Prop. II.8](../../02-api-guidelines/propositions/08-idempotency.md), [Prop. XI.1](01-generated-clients.md), [Prop. XI.8](08-traceparent-and-rum.md)

## Demonstration

By [Post. I.4](../../01-foundations/postulates.md#Post.%20I.4%20-%20Unreliable%20Network) a request may succeed without the caller learning of it, and a front-end on a mobile network meets this case daily. A POST is not idempotent ([Def. I.16](../../01-foundations/definitions.md#Def.%20I.16%20-%20Idempotent%20Operation), [Prop. II.3](../../02-api-guidelines/propositions/03-methods-and-status-codes.md)), so a naive retry creates the effect twice. [Prop. II.8](../../02-api-guidelines/propositions/08-idempotency.md) gives producers a way to make such a request safe to repeat, keyed on a value the caller supplies; the caller is the front-end, so the key must be born there, before the first attempt, and survive the failure that prompts the retry. A key generated per attempt rather than per intent would defeat the mechanism. The generated client ([Prop. XI.1](01-generated-clients.md)) sees every request, so the key is attached there and no operation is missed. ∎ Q.E.D.

## Corollaries

* **Cor. XI.12.1** - The key is bound to the user's intent: it is created when the form or action is opened, reused across retries, and discarded on a 2xx, on a 4xx other than 409 or 429, or when the user abandons the action.
* **Cor. XI.12.2** - Retries use the trace-id of the original attempt ([Cor. XI.8.1](08-traceparent-and-rum.md#Corollaries)) so that the producer's idempotency log and the trace agree.

## Construction

* Key: a ULID ([Prop. IV.2](../../04-shared-schemas/propositions/02-identifier.md)) or UUIDv4 generated with `crypto.randomUUID()` or the `ulid` package; opaque; not derived from form content.
* Attachment: request middleware in the generated client ([Prop. XI.1](01-generated-clients.md)) adds `Idempotency-Key` to every operation the OpenAPI document marks as requiring it ([Prop. II.8](../../02-api-guidelines/propositions/08-idempotency.md)); operations not so marked are not touched.
* Retry policy: a shared, framework-neutral retry helper in `@company/contracts/http` with bounded exponential backoff and jitter, retrying on network failure, 408, 429 (honouring `Retry-After`, [Prop. II.13](../../02-api-guidelines/propositions/13-rate-limiting.md)) and 5xx; never on other 4xx.
* Persistence: the in-flight key is kept in memory with the pending action; for native clients that may be suspended, in the platform secure store alongside the queued request.
* `Idempotency-Key` is in the CORS allowed headers ([Prop. XI.7](07-cors-allow-list.md)).

## Conformance

Contract test in the generated client package: a simulated network failure followed by a retry sends the same `Idempotency-Key`; a fresh submission sends a different one. Front-end lint rule: direct construction of an `Idempotency-Key` header outside the shared middleware is forbidden.

## Scholium

Double-submitted orders and payments are the most expensive class of front-end defect and the cheapest to prevent. Disabling the submit button is a courtesy, not a guarantee; the key is the guarantee. The producer side of this contract, including how long keys are remembered and how a conflicting replay is reported, is [Prop. II.8](../../02-api-guidelines/propositions/08-idempotency.md) and is not restated here.
