**Storage & File Formats**

- Parquet internals — row groups, column chunks, predicate pushdown, bloom filters
- ORC vs Parquet vs Avro — when each wins and why
- Delta Lake / Iceberg / Hudi — ACID on object storage, time travel, merge-on-read vs copy-on-write

**Query Execution**

- Vectorized execution — why columnar processing destroys row-by-row
- Query planning — logical vs physical plan, cost-based optimizer
- Predicate pushdown, projection pruning — how engines avoid reading data
- Shuffle in distributed joins — why it kills performance, broadcast join as escape hatch
- Partitioning strategies — range vs hash vs list, partition pruning

**Distributed Systems fundamentals**

- CAP theorem — not just definition, actual tradeoffs in real systems
- Consistent hashing — used in distributed caches, Kafka partition assignment
- Exactly-once semantics — how Kafka + Flink actually achieve it (idempotent producers, two-phase commit)
- Backpressure — what it is, how Flink handles it, why it matters

**Stream Processing**

- Watermarks — how out-of-order events are handled
- Event time vs processing time vs ingestion time — real difference with examples
- Windowing — tumbling, sliding, session windows internals
- State backends in Flink — RocksDB vs heap, checkpointing
- Kafka internals — log segments, ISR, leader election, consumer group rebalancing

**Data Modeling**

- Slowly changing dimensions (SCD types 1-6) — deep, not just definitions
- Data vault vs Kimball vs OBT — tradeoffs at scale
- Normalization vs denormalization in analytical workloads

**Compute & Memory**

- Spill to disk — when and why engines spill, how to avoid it
- Memory management in Spark — unified memory model, off-heap
- Skew handling — salting, AQE in Spark, skew joins

**Reliability & Correctness**

- Idempotency — designing pipelines that can safely re-run
- Schema evolution — backward/forward compatibility, how Avro/Protobuf handle it
- Data quality frameworks — expectations, contract testing, anomaly detection

**Orchestration**

- DAG scheduling internals — Airflow scheduler loop, why it doesn't scale linearly
- Dynamic DAGs vs static DAGs — tradeoffs
- Backfill strategies — partitioned backfills, dependency management

**Cost & Performance**

- Compaction — why small files kill performance, strategies to fix it
- Z-ordering / data clustering — what it does to query performance
- Caching layers — result cache vs metadata cache vs data cache in engines like Trino/BigQuery

**Learning order:** Storage formats → Query execution → Kafka internals → Stream processing → Distributed systems fundamentals → Data modeling → Reliability → Orchestration → Cost optimization