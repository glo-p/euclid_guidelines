# Book I - Common Notions

Common notions are self-evident principles of reasoning. They are not about our cloud or our language; they would be true in any company. Every book may invoke them by number.

---

**CN 1 - Substitutability.** Two implementations that conform to the same contract are interchangeable for every conforming consumer. *(The contract is the whole of the interface; therefore nothing else can distinguish them.)*

**CN 2 - A Published Contract Is Owed.** Once a contract is published, breaking it without the deprecation process of Book IX is a defect in the producer, never a change request to the consumers.

**CN 3 - Explicit Is Greater Than Implicit.** Anything not stated in the contract does not exist for the consumer. Behaviour, fields, ordering or error conditions that a producer exhibits but does not declare may not be relied upon and may not be complained about when they change.

**CN 4 - Compatibility Is Compositional.** A change to a contract is compatible if and only if every part of it is a compatible change (Def. I.15). One breaking part makes the whole change breaking.

**CN 5 - One Concept, One Shape.** Two schemas that describe the same concept with different shapes are a cost with no benefit. Where a shared schema exists for a concept, inventing another is a defect.

**CN 6 - The Producer Pays for Stability; the Consumer Pays for Tolerance.** A producer must not break conforming consumers. A consumer must ignore what it does not understand (unknown fields, unknown event types, unknown enum values where declared open) rather than fail. Both halves are required; neither substitutes for the other.

**CN 7 - What Cannot Be Observed Cannot Be Operated.** A behaviour that emits no log, metric, trace or event is, for operational purposes, absent.

**CN 8 - The Whole Is Not Greater Than Its Contracts.** The architecture of the company is exactly the set of published contracts and the topology that connects them. If it is not in a contract or a recorded decision, it is not architecture; it is habit.
