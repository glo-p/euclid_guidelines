# Prop. IV.12 - Security-relevant actions are an `AuditEvent`

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

The payload of every audit event ([Prop. V.9](../../05-security/propositions/09-audit-events.md)) MUST conform to [`audit-event.json`](../schemas/shared/v1/audit-event.json) and MUST be published with event type `com.company.<context>.audit.action-performed.v1` ([Prop. III.2](../../03-events/propositions/02-naming.md)). It records `actor`, `action`, `resource`, `outcome` (closed: `success`, `denied`, `failed`), `occurredAt`, and optionally `tenantId`, `clientId`, `sourceIp`, `dataClassification` and a flat `attributes` map of short strings. It MUST NOT carry the data acted upon.

## Given

[CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape), [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated), [Prop. III.1](../../03-events/propositions/01-envelope.md), [III.2](../../03-events/propositions/02-naming.md), [III.10](../../03-events/propositions/10-payload-design.md), [V.9](../../05-security/propositions/09-audit-events.md), [VI.7](../../06-observability/propositions/07-no-pii-or-secrets-in-logs.md), [IV.7](07-resource-metadata-and-actor.md).

## Demonstration

Audit is consumed centrally (a security information store) and must be queryable across every service; that requires one shape ([CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape)). Carrying it as an ordinary event reuses the outbox, envelope and archive guarantees ([Prop. III.1](../../03-events/propositions/01-envelope.md), [III.13](../../03-events/propositions/13-archive-and-replay.md)), so an audit record is exactly as durable as any other fact ([CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated)). Excluding the data itself keeps the audit stream at a fixed classification and avoids copying Restricted data into a long-retained store ([Prop. III.10](../../03-events/propositions/10-payload-design.md), [VI.7](../../06-observability/propositions/07-no-pii-or-secrets-in-logs.md)); references suffice because the resource can be fetched by an authorised investigator. `Actor` is reused by [CN 5](../../01-foundations/common-notions.md#CN%205%20-%20One%20Concept%2C%20One%20Shape). ∎ Q.E.D.

## Corollaries

* **Cor. IV.12.1** - `outcome: denied` events are published for authorisation failures at the service ([Prop. V.2](../../05-security/propositions/02-coarse-scopes-fine-authorisation.md)); the edge's own denials are logged by the edge.
* **Cor. IV.12.2** - The audit subscriber is a platform service; each context's `audit.action-performed.v1` is routed to it by a rule owned by that service ([Prop. III.9](../../03-events/propositions/09-topology.md)).

## Construction

`Company.Platform.Security.Audit` provides `IAuditWriter.RecordAsync(...)` which writes an `AuditEvent` to the outbox with the current `Actor`, tenant and trace.

## Conformance

Catalogue CI validates examples. Security review checklist ([Prop. X.4](../../10-governance/propositions/04-architecture-review-by-event.md)) confirms each service's list of audited actions.
