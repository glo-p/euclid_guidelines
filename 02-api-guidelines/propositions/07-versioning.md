# Prop. II.7 - Only the major version is in the URI

| | |
|---|---|
| **Level** | MUST |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

An API's base path MUST begin with `/v{n}` where `n` is its major version ([Def. II.9](../definitions.md#Def.%20II.9%20-%20Major%20Version)). Compatible changes ([Def. I.15](../../01-foundations/definitions.md#Def.%20I.15%20-%20Compatible%20Change)) MUST be made in place without changing `n`. A breaking change ([Def. I.14](../../01-foundations/definitions.md#Def.%20I.14%20-%20Breaking%20Change)) MUST be published as `/v{n+1}` alongside `/v{n}`, and `/v{n}` MUST then follow the deprecation lifecycle of [Book IX](../../09-versioning-and-deprecation/README.md), emitting `Deprecation` and `Sunset` headers on every response. Minor and patch versions MUST NOT appear in the path, in headers, or in media type parameters.

## Given

[Def. I.14](../../01-foundations/definitions.md#Def.%20I.14%20-%20Breaking%20Change), [Def. I.15](../../01-foundations/definitions.md#Def.%20I.15%20-%20Compatible%20Change), [Def. II.9](../definitions.md#Def.%20II.9%20-%20Major%20Version), [Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution), [Post. II.5](../postulates.md#Post.%20II.5%20-%20Consumers%20We%20Do%20Not%20Deploy), [CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed), [CN 4](../../01-foundations/common-notions.md#CN%204%20-%20Compatibility%20Is%20Compositional), [CN 6](../../01-foundations/common-notions.md#CN%206%20-%20The%20Producer%20Pays%20for%20Stability%3B%20the%20Consumer%20Pays%20for%20Tolerance), [Prop. IX.2](../../09-versioning-and-deprecation/propositions/02-breaking-change-is-a-new-major-side-by-side.md), [Prop. IX.3](../../09-versioning-and-deprecation/propositions/03-deprecation-is-declared-in-the-contract.md).

## Demonstration

Consumers lag producers indefinitely ([Post. I.5](../../01-foundations/postulates.md#Post.%20I.5%20-%20Independent%20Evolution), [Post. II.5](../postulates.md#Post.%20II.5%20-%20Consumers%20We%20Do%20Not%20Deploy)), and a published contract is owed ([CN 2](../../01-foundations/common-notions.md#CN%202%20-%20A%20Published%20Contract%20Is%20Owed)); therefore a breaking change can only be introduced by leaving the old contract in place and adding a new one, which is what a side-by-side major version is. Compatible changes by definition ([Def. I.15](../../01-foundations/definitions.md#Def.%20I.15%20-%20Compatible%20Change), [CN 4](../../01-foundations/common-notions.md#CN%204%20-%20Compatibility%20Is%20Compositional)) cannot break a conforming consumer, so a version marker for them carries no information a consumer can act on and merely multiplies the contracts to maintain. The major version must be visible to caches, proxies, the edge and logs, all of which see the path and not a header, hence the path. Consumers tolerate additions ([CN 6](../../01-foundations/common-notions.md#CN%206%20-%20The%20Producer%20Pays%20for%20Stability%3B%20the%20Consumer%20Pays%20for%20Tolerance)), which is what makes in-place compatible change safe. ∎ Q.E.D.

## Corollaries

* **Cor. II.7.1** - The OpenAPI `info.version` is the semantic version of the *document* (`2.3.1`) and is informational; only its major must equal `n`.
* **Cor. II.7.2** - At most two majors of one API are live at once ([Post. IX.1](../../09-versioning-and-deprecation/postulates.md#Post.%20IX.1%20-%20Two%20Majors%20at%20Most)).
* **Cor. II.7.3** - Adding a value to a *closed* enumeration on output is breaking; declare enumerations open ([Prop. IV.9](../../04-shared-schemas/propositions/09-open-enumerations.md)) when growth is anticipated.

## Construction

* Routing: `Asp.Versioning.Http` with `UrlSegmentApiVersionReader`, one `[ApiVersion(2)]` group per major; shared code between majors lives in the domain, not in the controllers.
* Deprecation: middleware adds `Deprecation: @1767225600` and `Sunset: Sat, 27 Jun 2027 00:00:00 GMT` (RFC 9745, RFC 8594) plus `Link: <https://developers.company.com/orders/v2/migration>; rel="deprecation"` when the request's major is marked deprecated in configuration.
* Edge: one API Gateway stage per major routes to the same service; retirement is removing the stage.

## Conformance

Spectral `paths-version-prefix` ([Prop. II.2](02-resource-naming.md)); `oasdiff breaking` fails CI when the diff is breaking and the major did not change; runtime test asserts the deprecation headers on a deprecated major.

## Scholium

Header versioning (`Accept: application/vnd.company.orders.v2+json`) is cleaner in theory and invisible in practice: it disappears from browser address bars, curl histories, access logs and support tickets. We chose visibility.
