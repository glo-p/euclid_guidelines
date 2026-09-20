# Book III - Postulates

---

## Post. III.1 - The Bus

Amazon EventBridge custom buses are the only event transport between services. There is one bus per environment, in a central account ([Prop. VIII.2](../08-infrastructure-aws/propositions/02-account-topology-and-central-bus.md)).

## Post. III.2 - The Buffer

Amazon SQS standard queues are the only consumer-side buffer. A subscriber never processes directly from an EventBridge target other than its own queue.

## Post. III.3 - At-Least-Once

The bus and the queue deliver at least once and may deliver more than once. Exactly-once delivery does not exist and is not to be approximated at the transport layer.

## Post. III.4 - No Transport Order

EventBridge and SQS standard queues do not preserve order across events, even for one publisher. Any ordering a consumer needs must be reconstructed from information in the envelope.

## Post. III.5 - Bounded Payloads

An EventBridge event is at most 256 KB including envelope. Anything larger travels by claim check ([Def. III.20](definitions.md#Def.%20III.20%20-%20Claim%20Check)).

## Post. III.6 - Facts Are Permanent

An event describes something that has already happened and has been committed. It cannot be rejected, and a consumer cannot "veto" it. If the consumer disagrees, it publishes its own event.

## Post. III.7 - Schema Validation Is the Check

JSON Schema validation of `data` against the registered schema, in the publisher's tests and at the consumer's edge, is the primary machine check ([Post. I.6](../01-foundations/postulates.md#Post.%20I.6%20-%20Machine%20Verification)) for this Book.
