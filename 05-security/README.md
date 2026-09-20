# Book V - Security

Book V covers identity, authorisation, secrets, data classification and audit. It rests on Book I, cites the API contract rules of Book II and the event rules of Book III, and is cited by Book VI (observability) and Book VII (data ownership).

**Depth:** Scaffold. Every proposition is `Draft`. The statements are settled enough to build against; the constructions name building blocks rather than give code.

## Definitions

| Item | Summary |
|---|---|
| `Def. V.1` Principal | Any authenticatable party to which an action is attributed: user, service or job. |
| `Def. V.2` Identity Provider | The one system that authenticates principals and issues tokens. |
| `Def. V.3` Access Token | Signed, time-limited JWT carrying identity, tenant, scopes and roles. |
| `Def. V.4` Scope | Coarse, contract-level grant evaluated at the edge. |
| `Def. V.5` Permission | Fine-grained grant to act on a resource, evaluated in the service. |
| `Def. V.6` Role | Named bundle of permissions; an administrative convenience, not a decision input. |
| `Def. V.7` Tenant Isolation | No principal of one tenant can read, change or infer another tenant's data. |
| `Def. V.8` Secret | Any value whose disclosure allows impersonation, decryption or unauthorised reach. |
| `Def. V.9` Data Classification | Four levels: Public, Internal, Confidential, Restricted; untagged means Confidential. |
| `Def. V.10` Attack Surface | The set of points through which untrusted input reaches a service. |
| `Def. V.11` Audit Event | Immutable record of a security-relevant action: who, what, which, for whom, when, outcome. |

See [`definitions.md`](definitions.md).

## Postulates

| Item | Summary |
|---|---|
| `Post. V.1` One Identity Provider | A single OIDC-compliant IdP issues every token; no service issues its own. |
| `Post. V.2` Single Ingress | API Gateway is the sole ingress for synchronous traffic. |
| `Post. V.3` Managed Secret Stores | Secrets live only in Secrets Manager or SSM Parameter Store. |
| `Post. V.4` Least Privilege | Every IAM principal holds exactly the permissions its function needs. |
| `Post. V.5` Untrusted Clients | Browsers and mobile apps are untrusted; only the IdP signature is trusted. |

See [`postulates.md`](postulates.md).

## Propositions

| Item | Level | Summary |
|---|---|---|
| [`Prop. V.1`](propositions/01-every-api-behind-the-edge.md) Every API is behind the edge with JWT validation | MUST | API Gateway validates every token; services refuse traffic that skipped the edge. |
| [`Prop. V.2`](propositions/02-coarse-scopes-fine-authorisation.md) Coarse scopes at the edge, fine-grained authorisation in the service | MUST | Scopes per operation at the edge; permission decisions inside the owning service; never on role names. |
| [`Prop. V.3`](propositions/03-service-to-service-authentication.md) Service-to-service authentication | MUST | Client credentials by default; IAM SigV4 as an open alternative; never forward a user token. |
| [`Prop. V.4`](propositions/04-secrets-never-in-source.md) Secrets are never in code, configuration files or committed environment | MUST | Secrets are read at runtime from the managed stores; scanners block commits. |
| [`Prop. V.5`](propositions/05-data-classification-in-contracts.md) Confidential and Restricted fields are tagged in the contract | MUST | `x-data-classification` on every sensitive field in OpenAPI and JSON Schema. |
| [`Prop. V.6`](propositions/06-pii-minimisation-in-events.md) PII minimisation in events | MUST | Events carry identifiers, not personal data; Restricted never appears in a payload. |
| [`Prop. V.7`](propositions/07-tenant-isolation-at-data-access.md) Tenant isolation is enforced at the data access layer | MUST | Tenant from the token or envelope only; the constraint lives in the data layer. |
| [`Prop. V.8`](propositions/08-scanning-gates-ci.md) Dependency, image and IaC scanning gate CI | MUST | Three scans before deploy; findings fixed or recorded as time-bound exceptions. |
| [`Prop. V.9`](propositions/09-audit-events.md) Security-relevant actions emit an audit event | MUST | CloudEvents audit records on a dedicated bus, joined to traces, retained immutably. |

## Open questions

Decisions the principal engineers still need to make before any proposition here moves from `Draft` to `Accepted`:

1. **Identity provider product** (Post. V.1). Amazon Cognito, Microsoft Entra ID, Okta or Auth0. The postulate is product-neutral; the construction sections are not.
2. **Service-to-service default** (Prop. V.3). Confirm OAuth2 client credentials as the default and decide whether IAM SigV4 becomes a sanctioned MAY, an exception-only path, or is withdrawn.
3. **Tenant claim name** (Prop. V.7). Which JWT claim carries the tenant identifier (`tid`, `tenant_id`, a custom namespaced claim) and whether it is a ULID.
4. **Scope naming scheme** (Prop. V.2). `resource:verb` versus `service.resource.verb`; whether scopes are generated from the OpenAPI document or the reverse.
5. **Central policy engine** (Prop. V.2). Whether fine-grained permissions stay in each service or move to a shared decision point (OPA, Cedar via Amazon Verified Permissions) in a later revision.
6. **Scan severity threshold and exception expiry** (Prop. V.8). Proposed: High and above block, Medium warns, exceptions expire in 90 days.
7. **Audit retention and SIEM** (Prop. V.9). Retention period per classification, and which SIEM receives the audit bus.
8. **`AuditEvent` shared schema** (Prop. V.9). Resolved: defined as Prop. IV.12 (`audit-event.json`). Remaining question is only which fields the SIEM requires beyond it.
9. **Classification dictionary** (Prop. V.5). Who maintains the field-name heuristics the linter uses, and where they live.
10. **Public endpoints** (Prop. V.1). Whether third-party webhook receivers get a standing exception pattern or individual ADRs.
