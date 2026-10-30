# intent-bus

**Subscribe by intent, not by topic.** A semantic layer for Apache Kafka.

Kafka assumes people name topics, partitions and schemas up front, and every
consumer knows them. Agents don't. With intent-bus a consumer declares what it
cares about — *anything that changes an order total* — and receives exactly
those events, from whichever topics they live in, in order per entity.
Producers emit typed events with provenance. When a producer changes an event's
shape, every consumer whose intent it touches gets an adapter and a migration
pull request **before the first changed message flows**.

> Status: founded, not yet built. The architecture is being designed first, as a
> board in [ProvenMap](https://provenmap.com); the code follows from it.

## How it works

- **Intents are typed, not fuzzy.** An intent is a predicate over the event
  catalog, such as `changes(Order.total)`, compiled and checked against the
  registered schemas. An LLM may help *write* one from a sentence, and a person
  reviews it; routing at runtime is deterministic and never uses embeddings.
- **Ordering and delivery are Kafka's.** Each intent is materialised as a
  derived stream keyed by entity, so per-entity order and delivery guarantees
  are exactly those of Kafka Streams.
- **Unmodified clients.** A filter in a Kafka protocol proxy
  ([Kroxylicious](https://github.com/kroxylicious/kroxylicious)) lets any Kafka
  client subscribe to `intent.<name>` as if it were a topic.
- **Contracts evolve ahead of breakage.** A schema change is checked for
  compatibility ([Apicurio Registry](https://github.com/Apicurio/apicurio-registry));
  the affected intents are computed, and their consumers get the adapter and
  the migration before the change goes live.

## Planned modules

| Module | What it does |
| --- | --- |
| `intent-model` | The intent language and event catalog; a pure library, no Kafka |
| `router` | Kafka Streams app materialising one derived stream per intent |
| `proxy-filter` | Kroxylicious filter: `intent.<name>` reads that stream |
| `contracts` | Compatibility check, affected intents, adapter and migration PRs |
| `demo` | Order-to-cash producers and consumers, run with Docker Compose |

Scala 3 and sbt; Kafka in KRaft mode.

## Milestones

1. **Typed intents** — one consumer receives `OrderPlaced`, `OrderAmended` and
   `RefundIssued` from three topics, in order per order, through one intent.
2. **Subscribe by name** — a stock `kafka-console-consumer` subscribes to an
   intent through the proxy.
3. **Contracts ahead of breakage** — a new producer shape arrives; every
   affected consumer gets an adapter and a migration PR before its first message.
4. **The event catalog, live** — the intents, channels and contracts on a
   public ProvenMap board.

## Licence

Apache License 2.0. See [LICENSE](LICENSE) and [NOTICE](NOTICE).

Apache Kafka, Kafka and the Kafka logo are trademarks of the Apache Software
Foundation. intent-bus is not affiliated with or endorsed by the ASF.
