# Prop. II.14 - Reads are cacheable and conditional

| | |
|---|---|
| **Level** | SHOULD |
| **Status** | Accepted |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |

## Statement

Every `GET` response MUST carry an explicit `Cache-Control` header. Item `GET`s SHOULD return `ETag` ([Prop. II.9](09-optimistic-concurrency.md)) and honour `If-None-Match` with `304`. Responses containing tenant or principal data MUST be `Cache-Control: private` (or `no-store` where the data is classified Restricted, [Book V](../../05-security/README.md)). Only representations that are identical for every principal MAY be `public`.

## Given

[Def. II.11](../definitions.md#Def.%20II.11%20-%20Entity%20Tag), [Post. II.1](../postulates.md#Post.%20II.1%20-%20HTTP%20Is%20Authoritative), [Post. II.5](../postulates.md#Post.%20II.5%20-%20Consumers%20We%20Do%20Not%20Deploy), [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [Prop. II.9](09-optimistic-concurrency.md), [Prop. V.5](../../05-security/propositions/05-data-classification-in-contracts.md).

## Demonstration

HTTP caching ([Post. II.1](../postulates.md#Post.%20II.1%20-%20HTTP%20Is%20Authoritative)) is performed by browsers, CDNs and proxies we do not control ([Post. II.5](../postulates.md#Post.%20II.5%20-%20Consumers%20We%20Do%20Not%20Deploy)); their default behaviour in the absence of a header is heuristic and therefore unsafe for tenant data. An explicit header makes the cacheability part of the contract ([CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit)). Conditional GET reuses the validator already required for concurrency ([Prop. II.9](09-optimistic-concurrency.md)), so it is free to add and it removes bandwidth for polling clients. ∎ Q.E.D.

## Corollaries

* **Cor. II.14.1** - `POST` results and Problems are never cacheable; the template sets `no-store` on them.
* **Cor. II.14.2** - Reference data ([Prop. VII.4](../../07-data-ownership/propositions/04-reference-data-distribution.md)) is the primary candidate for `public, max-age`, served with a long lifetime and versioned URLs.

## Construction

* ASP.NET Core `[ResponseCache]` or output caching middleware for public reference data; an endpoint filter that sets `private, no-cache` (revalidate every time) as the default for everything else and computes `304` from the entity version.
* CloudFront in front of the edge only for `public` routes.

## Conformance

Spectral: `get-declares-cache-control` (every `GET` `200` declares the header). Template test: default `GET` carries `Cache-Control: private, no-cache`.
