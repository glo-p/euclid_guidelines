# Book V - Definitions

These definitions fix the meaning of the security vocabulary used in Book V and cited by later books. They are not rules. Where a definition refines one in Book I it says so.

---

**Def. V.1 - Principal.** A *principal* is any party that can be authenticated and to which an action can be attributed: a human user, a service (Def. I.1), or an automated job. Every request and every message is performed on behalf of exactly one principal.

**Def. V.2 - Identity Provider.** The *identity provider* (IdP) is the system that authenticates principals and issues access tokens (Def. V.3) attesting to their identity. It is the only party whose attestation of identity a service accepts.

**Def. V.3 - Access Token.** An *access token* is a signed, time-limited credential issued by the identity provider that carries the principal's identity, tenant (Def. I.23), scopes (Def. V.4) and any roles (Def. V.6). The canonical form is a JWT signed with an asymmetric key published by the identity provider.

**Def. V.4 - Scope.** A *scope* is a coarse, contract-level grant carried in an access token that names an area of an API the token may reach, for example `orders:read`. Scopes are evaluated at the edge. A scope says *where* a principal may go, not *what* it may do to a particular resource.

**Def. V.5 - Permission.** A *permission* is a fine-grained grant to perform a named action on a resource or class of resources within a bounded context (Def. I.5), for example "cancel this order". Permissions are evaluated inside the service that owns the resource.

**Def. V.6 - Role.** A *role* is a named bundle of permissions (Def. V.5) assigned to a principal within a tenant. Roles are a convenience for administration; authorisation decisions are made on permissions, never on role names.

**Def. V.7 - Tenant Isolation.** *Tenant isolation* is the property that no principal acting for one tenant (Def. I.23) can read, modify or infer the existence of data belonging to another tenant, regardless of the request it constructs.

**Def. V.8 - Secret.** A *secret* is any value whose disclosure would allow a party to impersonate a principal, decrypt data, or reach a system it is not entitled to reach: passwords, API keys, signing keys, connection strings containing credentials, client secrets, private certificates.

**Def. V.9 - Data Classification.** *Data classification* assigns every field in a contract to exactly one of four levels, in ascending sensitivity:

| Level | Meaning |
|---|---|
| `Public` | May be disclosed to anyone without harm. |
| `Internal` | For company use; disclosure is undesirable but not harmful to any person. |
| `Confidential` | Disclosure harms the company or a customer organisation; includes commercial terms and most personal data. |
| `Restricted` | Disclosure causes serious harm or breaches a legal duty; includes authentication material, payment instruments, special-category personal data. |

A field with no classification is treated as `Confidential`.

**Def. V.10 - Attack Surface.** The *attack surface* of a service is the set of points through which an untrusted party can send input to it: network listeners, message queues it consumes, files it reads, and the dependencies it executes.

**Def. V.11 - Audit Event.** An *audit event* is an event (Def. I.9) recording that a security-relevant action was attempted or performed: who (the principal), what, on which resource, for which tenant, when, from where, and with what outcome. Audit events are immutable, retained for a defined period, and are not the same as logs.
