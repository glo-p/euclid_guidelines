# Book I - Definitions

These definitions fix the meaning of words used throughout the guidelines. They are not rules. When a later book needs a more specific meaning it *refines* a definition here and says so; it never contradicts one.

---

**Def. I.1 - Service.** A *service* is an independently deployable unit of software that owns a bounded context (Def. I.5), exposes one or more contracts (Def. I.2), and is owned by exactly one team (Def. I.17).

**Def. I.2 - Contract.** A *contract* is the published, versioned, machine-readable description of an interface that a service commits to honour: an OpenAPI document for an API, a JSON Schema for an event or shared type. A contract is the *whole* of what a consumer may rely on.

**Def. I.3 - Producer.** A *producer* (or *provider*) is the service that publishes a contract and is responsible for honouring it.

**Def. I.4 - Consumer.** A *consumer* is any party, inside or outside the company, that depends on a contract. A consumer is *conforming* if it relies on nothing outside the contract.

**Def. I.5 - Bounded Context.** A *bounded context* is an explicit boundary within which a domain model and its vocabulary are consistent. The same word may mean different things in different bounded contexts; that is expected, not a defect.

**Def. I.6 - Aggregate.** An *aggregate* is a cluster of domain objects that is changed and persisted as a single unit and has one root entity through which it is addressed. It is the unit of transactional consistency.

**Def. I.7 - Resource.** A *resource* is a thing a service exposes under a stable identifier through a synchronous API. A resource has one or more *representations* (Def. I.8).

**Def. I.8 - Representation.** A *representation* is the serialised form of a resource or message at a point in time, in a named media type.

**Def. I.9 - Event.** An *event* is an immutable, timestamped record that something has happened inside a bounded context, published *after* the fact was durably committed. Events are named in the past tense. An event cannot be rejected or un-happened.

**Def. I.10 - Command.** A *command* is a request that a service perform an action. It is named in the imperative and may be rejected.

**Def. I.11 - Message.** A *message* is an envelope carrying an event or a command across a transport. The envelope carries metadata (identity, time, type, correlation) separately from the payload.

**Def. I.12 - Schema.** A *schema* is the machine-readable definition of the shape of a representation or payload, expressed in JSON Schema 2020-12 unless a book says otherwise.

**Def. I.13 - Shared Schema.** A *shared schema* is a schema owned by the platform, recorded in the catalogue of Book IV, and reused unchanged by every team for the concept it describes.

**Def. I.14 - Breaking Change.** A *breaking change* to a contract is any change that can cause a conforming consumer (Def. I.4) of the previous version to fail, misbehave, or misinterpret data. Exhaustively: removing or renaming a field, path, event type or enum value; changing a type or its constraints to be narrower on output or wider on input; changing the meaning of an existing field; adding a required input field; changing a status code or error type for an existing condition.

**Def. I.15 - Compatible Change.** A *compatible change* is any change that is not a breaking change. Exhaustively: adding an optional input field; adding an output field; adding a new path, operation, event type or enum value *on output where the contract declares the enum open*; relaxing an input constraint; adding a new error type for a condition that was previously not handled.

**Def. I.16 - Idempotent Operation.** An operation is *idempotent* if performing it several times with the same inputs has the same observable effect on the system as performing it once. The *response* may differ; the *effect* may not.

**Def. I.17 - Team.** A *team* is the group that owns a service: builds it, runs it, is paged for it, and answers for its contracts. Ownership is exclusive: every service has exactly one owning team.

**Def. I.18 - Identifier.** An *identifier* is an opaque string that uniquely and permanently names a resource, message or aggregate within its type. Identifiers carry no meaning a consumer may decode. The canonical form is the ULID (26 characters, Crockford base32, lexicographically sortable by creation time). See Book IV.

**Def. I.19 - Level.** The *level* of a proposition is one of **MUST**, **SHOULD**, or **MAY**, with the meanings of RFC 2119. **MUST NOT** and **SHOULD NOT** are their negations.

**Def. I.20 - Exception.** An *exception* is a documented, time-bound, approved deviation from a **MUST** or **SHOULD** proposition, recorded as an ADR under Book X. A deviation that is not recorded is not an exception; it is a defect.

**Def. I.21 - Environment.** An *environment* is a complete, isolated deployment of the platform (for example `dev`, `test`, `prod`). Contracts are identical across environments; only configuration differs.

**Def. I.22 - Correlation.** *Correlation* is the ability to associate every log line, span, request and message with the originating business activity through a propagated identifier. The identifier is the W3C `traceparent` trace-id.

**Def. I.23 - Tenant.** A *tenant* is the customer organisation on whose behalf data is held or an operation is performed. Where the platform is multi-tenant, every resource, message and log line is attributable to exactly one tenant or explicitly to none.

**Def. I.24 - Catalogue.** The *catalogue* is the single, versioned repository of shared schemas and their generated packages (Book IV). There is exactly one.
