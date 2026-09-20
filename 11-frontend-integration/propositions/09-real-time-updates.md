# Prop. XI.9 - Real-time updates flow from Book III events with polling as fallback

| | |
|---|---|
| **Level** | SHOULD |
| **Status** | Draft |
| **Since** | 2026-09-20 |
| **Owner** | Principal engineers |
| **Supersedes** | - |

## Statement

A front-end needing updates without user action SHOULD receive them over a real-time channel (Def. XI.11) that is fed from Book III events and exposed at the edge as either an Amazon API Gateway WebSocket API or a server-sent events stream over HTTP. A real-time channel MUST carry notifications only; the front-end MUST fetch the current representation through the API (Prop. XI.1) on receipt. A front-end MUST fall back to polling the API when the channel is unavailable.

## Given

* Def. I.9, Def. I.11, Def. XI.11
* Post. I.4, Post. I.9, Post. XI.2, Post. XI.3
* CN 3, CN 6
* Prop. II.14, Prop. XI.1, Prop. XI.3

## Demonstration

Facts that change without user action are, by Def. I.9, events, and the only path events take out of a bounded context is the Book III bus (Post. I.9). A real-time channel is therefore a projection of those events to a front-end, and because a front-end may consume only contracts (Post. XI.3) it cannot subscribe to the bus itself; a service at the edge must relay. The channel is unreliable like any network path (Post. I.4): connections drop and messages are lost or duplicated, so the channel can signal that something changed but the API remains the only source of the representation (CN 3), and polling must remain possible. A notification the front-end does not understand is ignored (CN 6). ∎ Q.E.D.

## Corollaries

* **Cor. XI.9.1** - Real-time messages carry an event type, the resource identifier and the trace-id of the causing event, and nothing a front-end would otherwise have to fetch; they are hints, not representations.
* **Cor. XI.9.2** - Polling uses conditional requests (Prop. II.14, `ETag` and `If-None-Match`) so that the fallback is cheap for the producer.

## Construction

* Event source: EventBridge rules on the domain events of interest, targeting an SQS queue owned by the relaying service (Book III topology).
* Relay: the front-end's BFF (Prop. XI.3) or a dedicated notification service; it authorises the connection with the same JWT (Prop. V.1) and filters events to the principal's tenant (Def. I.23).
* Option A, WebSocket: Amazon API Gateway WebSocket API with a Lambda or .NET integration; connection ids in DynamoDB; `PostToConnection` to push. *(Requires a recorded exception to Post. I.9, which lists WebSockets as an exception transport.)*
* Option B, server-sent events: an HTTP endpoint on the BFF streaming `text/event-stream`, with `Last-Event-ID` for resumption; fits Post. I.9 without exception; needs API Gateway HTTP API or ALB with response streaming.
* Fallback: exponential backoff polling of the resource through the generated client with `If-None-Match`; poll interval from configuration.
* Client: the browser's native `EventSource` or `WebSocket`; no framework dependency.

## Conformance

Architecture test on the relaying service: its only inbound event source is an SQS queue subscribed to EventBridge; its pushed message schema is registered in the catalogue and contains no fields beyond the notification envelope. Front-end contract test: the view updates correctly with the channel disabled.

## Scholium

*Open question:* pick one transport company-wide. Server-sent events fit Post. I.9 and existing HTTP tooling (auth, CORS, tracing) with no exception; WebSocket adds bidirectional messaging the platform does not currently need and requires an exception ADR. The recommendation of the authors is server-sent events unless a bidirectional case is found. This proposition is a SHOULD because most front-ends are well served by polling and gain nothing from a channel.
