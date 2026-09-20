# Book V - Postulates

Postulates of [Book V](README.md) are facts about how identity, ingress and secrets work in **our** estate. They could be otherwise elsewhere. Every proposition of [Book V](README.md) rests on at least one of them; if one changes, re-examine the propositions that cite it.

---

## Post. V.1 - One Identity Provider

A single OIDC-compliant identity provider ([Def. V.2](definitions.md#Def.%20V.2%20-%20Identity%20Provider)) issues every access token ([Def. V.3](definitions.md#Def.%20V.3%20-%20Access%20Token)) accepted anywhere in the platform, for humans and for services alike. No service issues its own tokens. *(Which product plays this role is an open question recorded in the [Book V](README.md) README.)*

## Post. V.2 - Single Ingress

Amazon API Gateway is the sole ingress for synchronous traffic from outside a service's own AWS account boundary. No service exposes a listener reachable except through API Gateway.

## Post. V.3 - Managed Secret Stores

Secrets ([Def. V.8](definitions.md#Def.%20V.8%20-%20Secret)) live in AWS Secrets Manager or AWS Systems Manager Parameter Store (SecureString). No other store of secrets exists.

## Post. V.4 - Least Privilege

Every IAM principal (role, user, service-linked role) holds exactly the permissions required for its function and no more. Wildcard actions and wildcard resources are defects unless recorded as exceptions ([Def. I.20](../01-foundations/definitions.md#Def.%20I.20%20-%20Exception)).

## Post. V.5 - Untrusted Clients

Browsers and mobile applications are untrusted. Any value they send, including headers, identifiers and claimed tenant, may be forged. Only the signature of the identity provider on an access token may be trusted.
