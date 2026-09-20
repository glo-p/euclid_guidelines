# Prop. IV.12 - Security-relevant actions are an `AuditEvent`

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

The payload of every audit event (Prop. V.9) MUST conform to [`audit-event.json`](../schemas/shared/v1/audit-event.json) and MUST be published with event type `com.company.<context>.audit.action-performed.v1` (Prop. III.2). It records `actor`, `action`, `resource`, `outcome` (closed: `success`, `denied`, `failed`), `occurredAt`, and optionally `tenantId`, `clientId`, `sourceIp`, `dataClassification` and a flat `attributes` map of short strings. It MUST NOT carry the data acted upon.

## Given

CN 3, CN 5, CN 7, Prop. III.1, III.2, III.10, V.9, VI.7, IV.7.

## Demonstration

Audit is consumed centrally (a security information store) and must be queryable across every service; that requires one shape (CN 5). Carrying it as an ordinary event reuses the outbox, envelope and archive guarantees (Prop. III.1, III.13), so an audit record is exactly as durable as any other fact (CN 7). Excluding the data itself keeps the audit stream at a fixed classification and avoids copying Restricted data into a long-retained store (Prop. III.10, VI.7); references suffice because the resource can be fetched by an authorised investigator. `Actor` is reused by CN 5. ∎ Q.E.D.

## Corollaries

* **Cor. IV.12.1** - `outcome: denied` events are published for authorisation failures at the service (Prop. V.2); the edge's own denials are logged by the edge.
* **Cor. IV.12.2** - The audit subscriber is a platform service; each context's `audit.action-performed.v1` is routed to it by a rule owned by that service (Prop. III.9).

## Construction

`Company.Platform.Security.Audit` provides `IAuditWriter.RecordAsync(...)` which writes an `AuditEvent` to the outbox with the current `Actor`, tenant and trace.

## Conformance

Catalogue CI validates examples. Security review checklist (Prop. X.4) confirms each service's list of audited actions.
