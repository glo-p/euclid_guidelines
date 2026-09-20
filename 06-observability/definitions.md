# Book VI - Definitions

These definitions fix the meaning of the observability vocabulary used in Book VI. They are not rules. `Def. VI.6` refines `Def. I.22` and says so; nothing here contradicts Book I.

---

**Def. VI.1 - Log.** A *log* is a timestamped record of a discrete occurrence inside a process, emitted for human or machine reading, and retained for a bounded period.

**Def. VI.2 - Structured Log.** A *structured log* is a log (Def. VI.1) emitted as a single JSON object per record whose fields have stable names and types, so that a machine can filter and aggregate it without parsing free text. The message is one field among others, not the record.

**Def. VI.3 - Metric.** A *metric* is a named numeric measurement sampled or aggregated over time, with a small set of low-cardinality attributes. Metrics answer "how much" and "how often"; they do not identify individual requests.

**Def. VI.4 - Trace.** A *trace* is the record of one request or business activity as it crosses services, composed of spans (Def. VI.5) sharing a trace identifier. A trace answers "where did the time go" and "which call failed".

**Def. VI.5 - Span.** A *span* is a named, timed unit of work within a trace, with a parent span (or none, for the root), attributes, status and events. Every outbound call and every consumed message is a span.

**Def. VI.6 - Correlation.** *(Refines Def. I.22.)* *Correlation* is the ability to associate every log record, span, metric exemplar, request and message with the originating business activity through the W3C `traceparent` trace-id, and within a service through the span-id. A record lacking the trace-id is uncorrelated and counts as absent for investigation.

**Def. VI.7 - SLI.** A *service level indicator* is a quantitative measure of one aspect of the service a contract provides, expressed as a ratio of good events to total events over a window: availability, latency under a threshold, correctness, freshness.

**Def. VI.8 - SLO.** A *service level objective* is a target value for an SLI (Def. VI.7) over a stated window, declared by the owning team for a published contract, for example "99.9 % of `GET /orders/{id}` requests succeed within 300 ms over 30 days".

**Def. VI.9 - Error Budget.** The *error budget* is the complement of an SLO (Def. VI.8) over its window: the quantity of bad events the service may incur before the objective is missed. It is the currency in which risk is spent.

**Def. VI.10 - Health.** *Health* is a service's own assessment of whether it is able to do its work, exposed as a machine-readable endpoint. Health decomposes into readiness (Def. VI.11) and liveness (Def. VI.12).

**Def. VI.11 - Readiness.** A service is *ready* when it can accept and correctly process requests now: dependencies reachable, configuration loaded, warm-up complete. An instance that is not ready receives no traffic but is not restarted.

**Def. VI.12 - Liveness.** A service is *live* when its process is running and not deadlocked or otherwise unrecoverable. An instance that is not live is restarted. Liveness never depends on external dependencies.

**Def. VI.13 - Alert.** An *alert* is a notification, raised by a machine from metrics or logs, that a human must act now. An alert that does not require action is noise.

**Def. VI.14 - Runbook.** A *runbook* is the written procedure linked from an alert (Def. VI.13) that tells the responder what the alert means, how to confirm it, and what to do first.

**Def. VI.15 - Dashboard.** A *dashboard* is a persistent, shared view of a service's metrics, SLOs and recent alerts, arranged so that an on-call engineer can answer "is it healthy, and if not, where" in under a minute.
