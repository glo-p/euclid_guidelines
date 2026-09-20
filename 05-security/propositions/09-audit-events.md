# Prop. V.9 - Security-relevant actions emit an audit event

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every security-relevant action MUST emit an audit event ([Def. V.11](../definitions.md#Def.%20V.11%20-%20Audit%20Event)) as a CloudEvents message ([Prop. III.1](../../03-events/propositions/01-envelope.md)) on the audit bus, carrying principal, action, resource, tenant, outcome, source address and the trace identifier ([Def. I.22](../../01-foundations/definitions.md#Def.%20I.22%20-%20Correlation)). Security-relevant actions include at least: authentication outcomes, permission denials, changes to roles or permissions, access to `Restricted` data, secret retrieval, and any cross-tenant operation ([Cor. V.7.2](07-tenant-isolation-at-data-access.md#Corollaries)).

## Given

[Def. V.11](../definitions.md#Def.%20V.11%20-%20Audit%20Event), [Def. V.1](../definitions.md#Def.%20V.1%20-%20Principal), [Def. I.9](../../01-foundations/definitions.md#Def.%20I.9%20-%20Event), [Def. I.22](../../01-foundations/definitions.md#Def.%20I.22%20-%20Correlation), [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated), [Prop. III.1](../../03-events/propositions/01-envelope.md), [Prop. III.5](../../03-events/propositions/05-transactional-outbox.md), [Prop. VI.2](../../06-observability/propositions/02-traceparent-propagation.md), [Prop. VI.7](../../06-observability/propositions/07-no-pii-or-secrets-in-logs.md).

## Demonstration

By [CN 7](../../01-foundations/common-notions.md#CN%207%20-%20What%20Cannot%20Be%20Observed%20Cannot%20Be%20Operated) an action that leaves no record is, for the purpose of investigation, absent; an unrecorded permission change cannot be reversed or explained. The record must be immutable and attributable, which is the definition of an event ([Def. I.9](../../01-foundations/definitions.md#Def.%20I.9%20-%20Event)) and of an audit event in particular ([Def. V.11](../definitions.md#Def.%20V.11%20-%20Audit%20Event)). Using the standard envelope ([Prop. III.1](../../03-events/propositions/01-envelope.md)) and the outbox ([Prop. III.5](../../03-events/propositions/05-transactional-outbox.md)) gives durability without a second mechanism, and carrying the trace identifier ([Prop. VI.2](../../06-observability/propositions/02-traceparent-propagation.md)) joins the audit record to the operational record. Audit events are separate from logs because logs are redacted and short-lived ([Prop. VI.7](../../06-observability/propositions/07-no-pii-or-secrets-in-logs.md)) whereas audit needs full identity and long retention. ∎ Q.E.D.

## Corollaries

* **Cor. V.9.1** - Audit events are `Internal` as a whole but MAY contain `Confidential` identity fields that ordinary events may not ([Prop. V.6](06-pii-minimisation-in-events.md)); they are therefore published only to the audit bus, never to the general bus.
* **Cor. V.9.2** - Failure to write an audit event fails the action where the outbox is transactional; it never silently proceeds.

## Construction

* Dedicated EventBridge bus `audit` per account with an archive ([Prop. III.13](../../03-events/propositions/13-archive-and-replay.md)) and a rule forwarding to a locked-down S3 bucket with Object Lock and to the SIEM.
* Event type family `company.audit.*` ([Prop. III.2](../../03-events/propositions/02-naming.md)) with a shared schema `AuditEvent` proposed for [Book IV](../../04-shared-schemas/README.md).
* `Company.Audit` package: `IAuditWriter` backed by the transactional outbox ([Prop. III.14](../../03-events/propositions/14-dotnet-construction.md)), ASP.NET Core authorisation middleware emitting denials automatically.
* AWS CloudTrail for the AWS control plane; Secrets Manager access logged there and correlated by role.

## Conformance

Architecture test: every authorisation policy failure path and every role/permission mutation endpoint calls `IAuditWriter`. Contract test ([Prop. III.12](../../03-events/propositions/12-consumer-contracts-and-testing.md)) that the audit event validates against the `AuditEvent` schema. Security Hub control that the audit bus archive exists and the S3 bucket has Object Lock.

## Scholium

The list of security-relevant actions will grow. The principals own the list; teams propose additions by ADR. [Book VI](../../06-observability/README.md) defines how audit events are stored and queried alongside logs and traces without being confused with them.
