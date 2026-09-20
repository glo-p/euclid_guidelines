# Prop. V.3 - Service-to-service authentication

| | |
|---|---|
| **Level** | MUST |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A service calling another service's API MUST authenticate as its own principal ([Def. V.1](../definitions.md#Def.%20V.1%20-%20Principal)) using an OAuth2 client-credentials access token issued by the identity provider ([Post. V.1](../postulates.md#Post.%20V.1%20-%20One%20Identity%20Provider)). A service MAY instead use AWS IAM SigV4 request signing where both parties are in AWS and the target is a private integration; this alternative is an open question and MUST NOT be adopted without an ADR. A service MUST NOT forward a user's access token to another service as its own credential.

## Given

[Def. V.1](../definitions.md#Def.%20V.1%20-%20Principal), [Def. V.3](../definitions.md#Def.%20V.3%20-%20Access%20Token), [Post. V.1](../postulates.md#Post.%20V.1%20-%20One%20Identity%20Provider), [Post. V.2](../postulates.md#Post.%20V.2%20-%20Single%20Ingress), [Post. V.4](../postulates.md#Post.%20V.4%20-%20Least%20Privilege), [Prop. V.1](01-every-api-behind-the-edge.md), [Prop. V.9](09-audit-events.md), [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit).

## Demonstration

Every request is performed by exactly one principal ([Def. V.1](../definitions.md#Def.%20V.1%20-%20Principal)), and a service is a principal. By [Post. V.1](../postulates.md#Post.%20V.1%20-%20One%20Identity%20Provider) the identity provider issues every token, so a service obtains its own token from it, and client credentials is the OIDC grant for a non-human principal. Forwarding the user's token would attribute the downstream action to the user rather than the calling service, making the audit event ([Prop. V.9](09-audit-events.md)) untrue. SigV4 binds identity to IAM ([Post. V.4](../postulates.md#Post.%20V.4%20-%20Least%20Privilege)) and fits private integrations, but it bypasses the identity provider, so it is admitted only by recorded decision. ∎ Q.E.D.

## Corollaries

* **Cor. V.3.1** - The identity of the originating user, where relevant to the callee, travels as data in the request or event, not as the bearer credential.
* **Cor. V.3.2** - Each service has its own client registration; two services never share a client id.

## Construction

* Client registration per service in the identity provider; client secret stored in AWS Secrets Manager ([Prop. V.4](04-secrets-never-in-source.md)) and rotated by Secrets Manager rotation.
* `Duende.AccessTokenManagement` or `Microsoft.Extensions.Http` handlers acquiring and caching client-credentials tokens; typed `HttpClient` per downstream contract.
* API Gateway JWT authorizer accepting service audiences; scopes per [Prop. V.2](02-coarse-scopes-fine-authorisation.md).
* Alternative (open): API Gateway IAM authorizer with `AWSSDK.Extensions.NETCore.Setup` SigV4 signing via `AwsSignatureVersion4` handler.

## Conformance

Architecture test: every outbound `HttpClient` to a company API has the token management handler registered. Identity provider report: no client id used by more than one service.

## Scholium

Client credentials keeps one identity model for humans and services. SigV4 avoids a network hop to the identity provider and is attractive for Lambda-to-Lambda calls; the principals will decide whether it becomes a sanctioned MAY or is withdrawn.
