# Prop. V.9 - Security-relevant actions emit an audit event

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

Every security-relevant action MUST emit an audit event (Def. V.11) as a CloudEvents message (Prop. III.1) on the audit bus, carrying principal, action, resource, tenant, outcome, source address and the trace identifier (Def. I.22). Security-relevant actions include at least: authentication outcomes, permission denials, changes to roles or permissions, access to `Restricted` data, secret retrieval, and any cross-tenant operation (Cor. V.7.2).

## Given

Def. V.11, Def. V.1, Def. I.9, Def. I.22, CN 7, Prop. III.1, Prop. III.5, Prop. VI.2, Prop. VI.7.

## Demonstration

By CN 7 an action that leaves no record is, for the purpose of investigation, absent; an unrecorded permission change cannot be reversed or explained. The record must be immutable and attributable, which is the definition of an event (Def. I.9) and of an audit event in particular (Def. V.11). Using the standard envelope (Prop. III.1) and the outbox (Prop. III.5) gives durability without a second mechanism, and carrying the trace identifier (Prop. VI.2) joins the audit record to the operational record. Audit events are separate from logs because logs are redacted and short-lived (Prop. VI.7) whereas audit needs full identity and long retention. ∎ Q.E.D.

## Corollaries

* **Cor. V.9.1** - Audit events are `Internal` as a whole but MAY contain `Confidential` identity fields that ordinary events may not (Prop. V.6); they are therefore published only to the audit bus, never to the general bus.
* **Cor. V.9.2** - Failure to write an audit event fails the action where the outbox is transactional; it never silently proceeds.

## Construction

* Dedicated EventBridge bus `audit` per account with an archive (Prop. III.13) and a rule forwarding to a locked-down S3 bucket with Object Lock and to the SIEM.
* Event type family `company.audit.*` (Prop. III.2) with a shared schema `AuditEvent` proposed for Book IV.
* `Company.Audit` package: `IAuditWriter` backed by the transactional outbox (Prop. III.14), ASP.NET Core authorisation middleware emitting denials automatically.
* AWS CloudTrail for the AWS control plane; Secrets Manager access logged there and correlated by role.

## Conformance

Architecture test: every authorisation policy failure path and every role/permission mutation endpoint calls `IAuditWriter`. Contract test (Prop. III.12) that the audit event validates against the `AuditEvent` schema. Security Hub control that the audit bus archive exists and the S3 bucket has Object Lock.

## Scholium

The list of security-relevant actions will grow. The principals own the list; teams propose additions by ADR. Book VI defines how audit events are stored and queried alongside logs and traces without being confused with them.
