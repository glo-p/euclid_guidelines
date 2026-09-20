# Book I - Postulates

A postulate is something we accept as true about **our** world, without proof. Postulates are the things that could be otherwise in another company. If one of them stops being true, every proposition that cites it must be re-examined. That is the point of citing them.

---

## Post. I.1 - Cloud

All workloads run in AWS. Where AWS offers a managed service that meets a need, it is preferred to a self-managed equivalent.

## Post. I.2 - Language

Back-end services are written in .NET on the current LTS release. Front-ends use a mixture of technologies and this mixture will persist; no guideline may assume a particular front-end framework.

## Post. I.3 - Autonomy

Teams are autonomous and own their services end to end. It follows that **teams interact only through contracts** ([Def. I.2](definitions.md#Def.%20I.2%20-%20Contract)); there is no shared database, shared in-process library of business logic, or "just call their internal method" between teams.

## Post. I.4 - Unreliable Network

Any call across a process boundary may fail, be delayed arbitrarily, be delivered more than once, or succeed without the caller learning of it. No guideline may assume otherwise.

## Post. I.5 - Independent Evolution

Producers and consumers are deployed independently and at different cadences. At any moment several versions of a consumer exist against one version of a producer, and a consumer may lag a producer for an indefinite period.

## Post. I.6 - Machine Verification

A rule that is not verified by a machine (a linter, a contract test, an architecture test, a policy check) will be violated within a year. Human review is a supplement, never the primary enforcement.

## Post. I.7 - Contract First

It is possible to write and publish a contract before writing the implementation, and to generate clients, servers and validators from it. (This is the postulate that lets us "draw a straight line" between any two teams.)

## Post. I.8 - Stable Names

It is possible to reach any service in any environment by a stable, environment-scoped name that does not change when the service is redeployed, rescaled or moved.

## Post. I.9 - Transport

Synchronous communication uses HTTP/1.1 or HTTP/2 with JSON representations over TLS. Asynchronous communication uses Amazon EventBridge as the bus and Amazon SQS as the consumer buffer. Other transports (gRPC, GraphQL, Kafka, WebSockets) exist only as recorded exceptions ([Def. I.20](definitions.md#Def.%20I.20%20-%20Exception)).
