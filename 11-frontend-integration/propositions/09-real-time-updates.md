# Prop. XI.9 - Real-time updates flow from Book III events with polling as fallback

| | |
|---|---|
| **Level** | SHOULD |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A front-end needing updates without user action SHOULD receive them over a real-time channel ([Def. XI.11](../definitions.md#Def.%20XI.11%20-%20Real-time%20Channel)) that is fed from [Book III](../../03-events/README.md) events and exposed at the edge as either an Amazon API Gateway WebSocket API or a server-sent events stream over HTTP. A real-time channel MUST carry notifications only; the front-end MUST fetch the current representation through the API ([Prop. XI.1](01-generated-clients.md)) on receipt. A front-end MUST fall back to polling the API when the channel is unavailable.

## Given

* [Def. I.9](../../01-foundations/definitions.md#Def.%20I.9%20-%20Event), [Def. I.11](../../01-foundations/definitions.md#Def.%20I.11%20-%20Message), [Def. XI.11](../definitions.md#Def.%20XI.11%20-%20Real-time%20Channel)
* [Post. I.4](../../01-foundations/postulates.md#Post.%20I.4%20-%20Unreliable%20Network), [Post. I.9](../../01-foundations/postulates.md#Post.%20I.9%20-%20Transport), [Post. XI.2](../postulates.md#Post.%20XI.2%20-%20Untrusted%20Clients), [Post. XI.3](../postulates.md#Post.%20XI.3%20-%20Contracts%20Only)
* [CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit), [CN 6](../../01-foundations/common-notions.md#CN%206%20-%20The%20Producer%20Pays%20for%20Stability%3B%20the%20Consumer%20Pays%20for%20Tolerance)
* [Prop. II.14](../../02-api-guidelines/propositions/14-caching.md), [Prop. XI.1](01-generated-clients.md), [Prop. XI.3](03-backend-for-frontend.md)

## Demonstration

Facts that change without user action are, by [Def. I.9](../../01-foundations/definitions.md#Def.%20I.9%20-%20Event), events, and the only path events take out of a bounded context is the [Book III](../../03-events/README.md) bus ([Post. I.9](../../01-foundations/postulates.md#Post.%20I.9%20-%20Transport)). A real-time channel is therefore a projection of those events to a front-end, and because a front-end may consume only contracts ([Post. XI.3](../postulates.md#Post.%20XI.3%20-%20Contracts%20Only)) it cannot subscribe to the bus itself; a service at the edge must relay. The channel is unreliable like any network path ([Post. I.4](../../01-foundations/postulates.md#Post.%20I.4%20-%20Unreliable%20Network)): connections drop and messages are lost or duplicated, so the channel can signal that something changed but the API remains the only source of the representation ([CN 3](../../01-foundations/common-notions.md#CN%203%20-%20Explicit%20Is%20Greater%20Than%20Implicit)), and polling must remain possible. A notification the front-end does not understand is ignored ([CN 6](../../01-foundations/common-notions.md#CN%206%20-%20The%20Producer%20Pays%20for%20Stability%3B%20the%20Consumer%20Pays%20for%20Tolerance)). ∎ Q.E.D.

## Corollaries

* **Cor. XI.9.1** - Real-time messages carry an event type, the resource identifier and the trace-id of the causing event, and nothing a front-end would otherwise have to fetch; they are hints, not representations.
* **Cor. XI.9.2** - Polling uses conditional requests ([Prop. II.14](../../02-api-guidelines/propositions/14-caching.md), `ETag` and `If-None-Match`) so that the fallback is cheap for the producer.

## Construction

* Event source: EventBridge rules on the domain events of interest, targeting an SQS queue owned by the relaying service ([Book III](../../03-events/README.md) topology).
* Relay: the front-end's BFF ([Prop. XI.3](03-backend-for-frontend.md)) or a dedicated notification service; it authorises the connection with the same JWT ([Prop. V.1](../../05-security/propositions/01-every-api-behind-the-edge.md)) and filters events to the principal's tenant ([Def. I.23](../../01-foundations/definitions.md#Def.%20I.23%20-%20Tenant)).
* Option A, WebSocket: Amazon API Gateway WebSocket API with a Lambda or .NET integration; connection ids in DynamoDB; `PostToConnection` to push. *(Requires a recorded exception to [Post. I.9](../../01-foundations/postulates.md#Post.%20I.9%20-%20Transport), which lists WebSockets as an exception transport.)*
* Option B, server-sent events: an HTTP endpoint on the BFF streaming `text/event-stream`, with `Last-Event-ID` for resumption; fits [Post. I.9](../../01-foundations/postulates.md#Post.%20I.9%20-%20Transport) without exception; needs API Gateway HTTP API or ALB with response streaming.
* Fallback: exponential backoff polling of the resource through the generated client with `If-None-Match`; poll interval from configuration.
* Client: the browser's native `EventSource` or `WebSocket`; no framework dependency.

## Conformance

Architecture test on the relaying service: its only inbound event source is an SQS queue subscribed to EventBridge; its pushed message schema is registered in the catalogue and contains no fields beyond the notification envelope. Front-end contract test: the view updates correctly with the channel disabled.

## Scholium

*Open question:* pick one transport company-wide. Server-sent events fit [Post. I.9](../../01-foundations/postulates.md#Post.%20I.9%20-%20Transport) and existing HTTP tooling (auth, CORS, tracing) with no exception; WebSocket adds bidirectional messaging the platform does not currently need and requires an exception ADR. The recommendation of the authors is server-sent events unless a bidirectional case is found. This proposition is a SHOULD because most front-ends are well served by polling and gain nothing from a channel.
