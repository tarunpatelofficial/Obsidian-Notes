## 🔥 Apache Spark — From 0 to 100

Let's start with the **big picture**: what Spark is and why it exists.

### Why Spark?

Before Spark, Hadoop MapReduce was the standard for big data. But [[MapReduce wrote intermediate results to disk]] after every step — painfully slow for iterative algorithms (like ML training). Spark solved this by keeping data **in memory** across operations, making it 10–100× faster.

---

### [[1. The Core Architecture]]

**Key roles:**
- **Driver** — your program's brain. It builds the execution plan and sends tasks to workers.
- **Cluster Manager** — allocates resources (YARN, Kubernetes, etc.). Spark doesn't care which one.
- **Executors** — JVM processes on worker nodes. They run tasks and cache data in memory.
- **Tasks** — the smallest unit of work. One task = one data partition.

![[Pasted image 20260616130533.png]]

---

### 2. RDDs, DataFrames & Datasets — The Data Abstractions


![[Pasted image 20260616130549.png]]
**In practice as a data engineer:** You'll use DataFrames 95% of the time. RDDs are for edge cases. Datasets are Scala-only.

---

### 3. How Spark Actually Executes — The DAG

This is the heart of Spark's magic. When you write transformations, Spark doesn't run them immediately — it **builds a plan first**.

![[Pasted image 20260616130732.png]]

**The golden rule:** Minimize shuffles. Every `groupBy`, `join`, and `orderBy` moves data across the network — that's your biggest performance enemy.

---

### 4. Spark's Component Ecosystem

![[Pasted image 20260616130751.png]]

---
### 5. The Catalyst Optimizer & Tungsten Engine — Why Spark is Fast

![[Pasted image 20260616130934.png]]

---

### 6. Partitions, Memory & Caching

![[Pasted image 20260616130947.png]]

---

### 7. Key Performance Patterns — What Every DE Must Know

![[Pasted image 20260616131017.png]]

---

### 8. Structured Streaming — Real-time with Spark

![[Pasted image 20260616131054.png]]

---

### 9. Complete Mental Map — Everything in One View

![[Pasted image 20260616131121.png]]

---

1. **Core architecture** — Driver, Cluster Manager, Executors, Tasks
2. **Data abstractions** — RDD → DataFrame → Dataset (use DataFrames!)
3. **DAG execution** — lazy evaluation, narrow vs wide transforms, shuffles
4. **Ecosystem** — SQL, Streaming, MLlib, GraphX
5. **Catalyst + Tungsten** — how Spark auto-optimizes your queries
6. **Partitions & memory** — the engine's physical model
7. **Performance patterns** — joins, skew, caching, file formats
8. **Structured Streaming** — triggers, output modes, watermarks
9. **Complete reference map** — everything at a glance
