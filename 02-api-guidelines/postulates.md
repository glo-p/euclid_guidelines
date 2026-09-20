# Book II - Postulates

---

**Post. II.1 - HTTP Is Authoritative.** The semantics of methods, status codes, headers and caching are those of RFC 9110 and RFC 9111. We do not reinterpret them, and we do not tunnel our own semantics through them (for example, `200` with an error body).

**Post. II.2 - OpenAPI Is the Contract.** The OpenAPI 3.1 document is the contract (Def. I.2) of an API. Code is generated from it or verified against it; where the two disagree the document is right and the code is defective.

**Post. II.3 - JSON Is the Representation.** Request and response bodies are JSON (RFC 8259) in UTF-8. Binary content is transferred by pre-signed S3 URL, never inline.

**Post. II.4 - One Ingress.** Every API is reachable only through the edge (Def. II.12). The only endpoints reachable by another path are the health endpoints of Prop. II.16, which are reachable by the orchestrator.

**Post. II.5 - Consumers We Do Not Deploy.** Consumers of an API include software we do not deploy and cannot update: front-ends cached in browsers, mobile apps in app stores, partner integrations. Post. I.5 therefore applies with indefinite lag.

**Post. II.6 - The Linter Is the Check.** The company Spectral ruleset, applied to the OpenAPI document in CI, is the primary machine check (Post. I.6) for Book II. A proposition whose rule can be expressed as a Spectral rule is checked that way.
