# Book II - Postulates

---

## Post. II.1 - HTTP Is Authoritative

The semantics of methods, status codes, headers and caching are those of RFC 9110 and RFC 9111. We do not reinterpret them, and we do not tunnel our own semantics through them (for example, `200` with an error body).

## Post. II.2 - OpenAPI Is the Contract

The OpenAPI 3.1 document is the contract ([Def. I.2](../01-foundations/definitions.md#Def.%20I.2%20-%20Contract)) of an API. Code is generated from it or verified against it; where the two disagree the document is right and the code is defective.

## Post. II.3 - JSON Is the Representation

Request and response bodies are JSON (RFC 8259) in UTF-8. Binary content is transferred by pre-signed S3 URL, never inline.

## Post. II.4 - One Ingress

Every API is reachable only through the edge ([Def. II.12](definitions.md#Def.%20II.12%20-%20Edge)). The only endpoints reachable by another path are the health endpoints of [Prop. II.16](propositions/16-health-endpoints.md), which are reachable by the orchestrator.

## Post. II.5 - Consumers We Do Not Deploy

Consumers of an API include software we do not deploy and cannot update: front-ends cached in browsers, mobile apps in app stores, partner integrations. [Post. I.5](../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution) therefore applies with indefinite lag.

## Post. II.6 - The Linter Is the Check

The company Spectral ruleset, applied to the OpenAPI document in CI, is the primary machine check ([Post. I.6](../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification)) for [Book II](README.md). A proposition whose rule can be expressed as a Spectral rule is checked that way.
