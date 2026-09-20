# Book II - Definitions

These refine the [Book I](../01-foundations/README.md) definitions for synchronous HTTP APIs.

---

## Def. II.1 - API

An *API* is the set of operations ([Def. II.2](definitions.md#Def.%20II.2%20-%20Operation)) exposed by one service ([Def. I.1](../01-foundations/definitions.md#Def.%20I.1%20-%20Service)) under one base path and described by exactly one OpenAPI 3.1 document. A service may expose more than one API only when the APIs have different audiences (for example a public API and an administrative API).

## Def. II.2 - Operation

An *operation* is the pair (HTTP method, path template) together with its declared parameters, request body, responses and security requirements. An operation is the unit of contract in an API.

## Def. II.3 - Collection

A *collection* is a resource ([Def. I.7](../01-foundations/definitions.md#Def.%20I.7%20-%20Resource)) whose representation is an ordered set of resources of one type. A collection has a stable, total ordering declared in the contract.

## Def. II.4 - Item

An *item* is a single resource addressed within a collection by its identifier ([Def. I.18](../01-foundations/definitions.md#Def.%20I.18%20-%20Identifier)).

## Def. II.5 - Media Type

The *media type* of a representation is `application/json` for success bodies and `application/problem+json` for error bodies. No other body media types are used for structured data.

## Def. II.6 - Page

A *page* is a bounded, contiguous subset of a collection in the collection's declared ordering, together with the information needed to reach the adjacent pages.

## Def. II.7 - Cursor

A *cursor* is an opaque, URL-safe string, minted by the producer, that names a position in a collection's ordering. A consumer may store and present a cursor; it may not construct, decode or compare one.

## Def. II.8 - Problem

A *problem* is an error representation conforming to RFC 9457 and to the shared schema `ProblemDetails` ([Prop. IV.5](../04-shared-schemas/propositions/05-problem-details.md)). Its `type` member is a stable URI that identifies the *kind* of error independently of the wording of `title` and `detail`.

## Def. II.9 - Major Version

The *major version* of an API is the integer `n` in the first path segment `/v{n}` of its base path. It changes only on a breaking change ([Def. I.14](../01-foundations/definitions.md#Def.%20I.14%20-%20Breaking%20Change)).

## Def. II.10 - Idempotency Key

An *idempotency key* is a client-generated identifier ([Def. I.18](../01-foundations/definitions.md#Def.%20I.18%20-%20Identifier)) sent in the `Idempotency-Key` header that names one *intent* to perform a non-idempotent operation, so that repeated deliveries of the same intent are recognised.

## Def. II.11 - Entity Tag

An *entity tag* (`ETag`) is an opaque validator, minted by the producer, that changes whenever the representation of a resource changes.

## Def. II.12 - Edge

The *edge* is Amazon API Gateway configured per environment as the sole ingress for every API. The edge terminates TLS, validates tokens, applies rate limits and forwards to services by their stable names ([Post. I.8](../01-foundations/postulates.md#Post.%20I.8%20-%20Stable%20Names)).

## Def. II.13 - Client

A *client* is the software making an HTTP request. It is distinct from the *principal* ([Book V](../05-security/README.md)), which is the identity on whose behalf the request is made.

## Def. II.14 - Envelope

A *collection envelope* is the object `{ items, pageInfo }` defined by the shared schema `Page` ([Prop. IV.6](../04-shared-schemas/propositions/06-page.md)). Single resources have **no** envelope: the body *is* the representation.

## Def. II.15 - Field

A *field* is a named member of a JSON object in a representation. Fields are camelCase and their names are part of the contract.

## Def. II.16 - Open Enumeration

An *open enumeration* is a string-valued field whose contract lists known values and declares that new values may appear on output without a major version change ([Prop. IV.9](../04-shared-schemas/propositions/09-open-enumerations.md)). A *closed enumeration* declares the list complete.

## Def. II.17 - Long-Running Operation

A *long-running operation* is any request whose processing may exceed the edge's request timeout or that is queued for later execution. Its progress is itself a resource.

## Def. II.18 - Sub-resource

A *sub-resource* is a resource whose identity is scoped to a parent resource (for example `/orders/{orderId}/lines/{lineId}`). A sub-resource cannot exist without its parent and is deleted with it.
