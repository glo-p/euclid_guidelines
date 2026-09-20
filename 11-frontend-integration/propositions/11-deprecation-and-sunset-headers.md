# Prop. XI.11 - Front-ends surface Deprecation and Sunset headers in telemetry

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A front-end MUST detect the `Deprecation` and `Sunset` response headers ([Prop. IX.3](../../09-versioning-and-deprecation/propositions/03-deprecation-is-declared-in-the-contract.md)) on every response received through a generated client and MUST record each occurrence as a warning in client telemetry ([Prop. XI.8](08-traceparent-and-rum.md)), naming the operation, the API major and the sunset date. A front-end MUST NOT alter its behaviour towards the user on the basis of these headers.

## Given

* [Def. I.4](../../01-foundations/definitions.md#Def.%20I.4%20-%20Consumer), [Def. XI.1](../definitions.md#Def.%20XI.1%20-%20Front-end), [Def. XI.6](../definitions.md#Def.%20XI.6%20-%20Generated%20Client)
* [Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution), [Post. XI.2](../postulates.md#Post.%20XI.2%20-%20Untrusted%20Clients)
* [CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed), [CN 6](../../01-foundations/common-notions.md#CN%206%20-%20The%20Producer%20Pays%20for%20Stability%3B%20the%20Consumer%20Pays%20for%20Tolerance), [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated)
* [Prop. II.10](../../02-api-guidelines/propositions/10-headers-and-metadata.md), [Prop. IX.3](../../09-versioning-and-deprecation/propositions/03-deprecation-is-declared-in-the-contract.md), [Prop. XI.1](01-generated-clients.md), [Prop. XI.8](08-traceparent-and-rum.md)

## Demonstration

By [Prop. IX.3](../../09-versioning-and-deprecation/propositions/03-deprecation-is-declared-in-the-contract.md) deprecation is declared in the contract and signalled on the wire with `Deprecation` and `Sunset` ([Prop. II.10](../../02-api-guidelines/propositions/10-headers-and-metadata.md)). A front-end lags its producers indefinitely ([Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution)) and is installed on devices the company does not control ([Post. XI.2](../postulates.md#Post.%20XI.2%20-%20Untrusted%20Clients)), so the only way to know which front-end versions still call a deprecated operation is for those front-ends to say so; by [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated) a migration that is not observed cannot be managed. The producer owes the deprecation process ([CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed)) and the consumer owes tolerance ([CN 6](../../01-foundations/common-notions.md#CN%206%20-%20The%20Producer%20Pays%20for%20Stability%3B%20the%20Consumer%20Pays%20for%20Tolerance)), so the front-end continues to work and merely reports. The generated client ([Prop. XI.1](01-generated-clients.md)) sees every response, so it is the one place the check need be implemented. ∎ Q.E.D.

## Corollaries

* **Cor. XI.11.1** - A front-end build whose generated client's major is itself marked deprecated in the package registry fails its CI with a warning after the deprecation date and an error after the sunset date.
* **Cor. XI.11.2** - Telemetry from this proposition is the input to the producer's decision to sunset ([Book IX](../../09-versioning-and-deprecation/README.md)); a producer may not sunset while front-end telemetry shows live callers without an exception ADR.

## Construction

* Detection: a response interceptor in the generated client's `fetch` middleware ([Prop. XI.1](01-generated-clients.md)) reads `Deprecation` (RFC 9745) and `Sunset` (RFC 8594); both are in the CORS exposed headers ([Prop. XI.7](07-cors-allow-list.md)).
* Telemetry: one RUM custom event or OpenTelemetry span event `api.deprecated` with attributes `api`, `major`, `operationId`, `sunset`, `frontend.version`; rate-limited to once per operation per session to avoid noise.
* Dashboard: a CloudWatch or Datadog widget per API listing deprecated operations by calling front-end and version; owned by the API's team.
* Registry: `npm deprecate` on the generated client major at the deprecation date; CI in front-end repositories surfaces the `npm` warning.
* Server-rendered front-ends: the same interceptor runs in the server process; the warning goes to structured logs ([Prop. VI.1](../../06-observability/propositions/01-structured-json-logs.md)).

## Conformance

Contract test in the generated client package: a stubbed response carrying `Deprecation` and `Sunset` produces exactly one telemetry event with the required attributes. Front-end CI: `npm ls` reports no dependency on a generated client major past its sunset date.

## Scholium

The header check is cheap and lives in one shared place; what it buys is a migration that can be seen from the producer's side, which is what [Book IX](../../09-versioning-and-deprecation/README.md) needs to work at all. Showing the user a "this feature will stop working" banner is a product decision and not required here; hiding the signal from operators is what this proposition forbids.
