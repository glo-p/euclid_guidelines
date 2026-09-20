# Book IX - Definitions

These definitions fix the vocabulary of contract lifecycle used in Book IX and cited by Books II, III, IV and X. They are not rules. Where a definition refines one in Book I it says so.

---

**Def. IX.1 - Version.** A *version* is a named, immutable revision of a contract (Def. I.2). Once published, the content of a version never changes; a correction is a new version. A version is identified by its contract name, its major version (Def. IX.2) and, within the major, a monotonically increasing minor revision.

**Def. IX.2 - Major Version.** The *major version* is the positive integer that a consumer (Def. I.4) selects when it binds to a contract: the path segment `/v2` of an API (Prop. II.7), the suffix `.v2` of an event type (Prop. III.2), the path segment of a shared schema `$id` (Prop. IV.1). Two versions with the same major are substitutable for a conforming consumer (CN 1). The major changes if and only if a breaking change (Def. I.14) is introduced.

**Def. IX.3 - Minor Change.** A *minor change* is a compatible change (Def. I.15) published within an existing major version. A minor change produces a new version (Def. IX.1) but not a new major version. Consumers do not select minor revisions; they receive them.

**Def. IX.4 - Lifecycle Stage.** The *lifecycle stage* of a version is exactly one of the following, in this order, and a version moves only forwards:

* **Proposed.** The version exists in the repository and catalogue but is not owed. It may change or be withdrawn without notice. No consumer may bind to it in `prod`.
* **Active.** The version is published and owed (CN 2). It accepts minor changes (Def. IX.3). New consumers may bind to it.
* **Deprecated.** The version is still owed and still served unchanged, but a deprecation notice (Def. IX.5) and a sunset date (Def. IX.6) have been published. New consumers MUST NOT bind to it. No minor change is made to it except a security fix.
* **Sunset.** The sunset date has passed and traffic evidence (Def. IX.9) shows no consumer. The version is no longer owed. It may still be deployed but may be removed at any moment without further notice.
* **Retired.** The version has been removed from every environment and its artefacts are marked retired in the catalogue. The identifier is never reused (Cor. I.4.1).

**Def. IX.5 - Deprecation Notice.** A *deprecation notice* is the machine-readable declaration, carried in or alongside the contract artefact itself, that a version has entered the Deprecated stage. It carries: the date of deprecation, the sunset date (Def. IX.6), the identifier of the successor major version, and a link to the migration guide (Def. IX.7). Its medium is fixed by Prop. IX.3.

**Def. IX.6 - Sunset Date.** The *sunset date* is the calendar date, stated in the deprecation notice, on and after which the producer (Def. I.3) may cease to honour a Deprecated version. It is at least the minimum deprecation period (Post. IX.2) after the date of deprecation. It may be moved later; it is never moved earlier.

**Def. IX.7 - Migration Guide.** A *migration guide* is the document, kept beside the contract in its repository, that enumerates every breaking change (Def. I.14) between major N and major N+1 of one contract and states, for each, what a consumer must do. A migration guide that omits a breaking change is defective.

**Def. IX.8 - Consumer Registry.** The *consumer registry* is the enumerated, queryable set of consumers (Def. I.4) of each Active or Deprecated version, each identified by a stable consumer identity and its owning team (Def. I.17) or external organisation. It is populated by machine from observed bindings (Prop. IX.4) and is the sole source of the recipients of a deprecation notice.

**Def. IX.9 - Traffic Evidence.** *Traffic evidence* is the RED metric series (Prop. VI.3) for one contract version, labelled by consumer identity, over a stated window. A version has *zero traffic* over a window if its request count and message delivery count are both zero for every consumer for the whole window.
