### A file is split into blocks

Suppose you have a 1 GB Parquet file.

It is not stored as one giant file on a single machine. Instead, HDFS splits it into blocks:

```text
Parquet File (1 GB)

Block 1  → Node 1
Block 2  → Node 3
Block 3  → Node 2
Block 4  → Node 5
...
```

(Usually each block is also replicated to other nodes for fault tolerance.)

---

### Spark creates partitions from those blocks

When Spark reads the Parquet file:

```python
df = spark.read.parquet("/data/sales.parquet")
```

it asks the storage system:

> "Where are the blocks of this file located?"

The storage system replies with metadata like:

```text
Partition 0 -> Node 1
Partition 1 -> Node 3
Partition 2 -> Node 2
Partition 3 -> Node 5
```

So Spark knows which machines physically hold the data.

---

### Why does partition 7 "live" on Node 3?

Because the underlying block(s) containing that partition were stored on Node 3's disk when the file was written.

For example:

```text
Node 1
 └─ Block A

Node 2
 └─ Block B

Node 3
 └─ Block C  ← Partition 7 reads from this block
```

When Spark creates partition 7, it maps that partition to Block C.

Therefore Spark knows:

> "Partition 7's data is already on Node 3."

---

### Why not run the task elsewhere?

Imagine Executor A is on Node 3 and Executor B is on Node 8.

If Spark runs partition 7 on Node 8:

```text
Node 3 disk
    ↓
Network transfer
    ↓
Node 8 Executor
```

Data must travel over the network.

Networks are much slower than reading from local RAM or local SSD.

---

### Data locality

Instead Spark prefers:

```text
Node 3 disk
    ↓
Executor on Node 3
```

No network transfer.

This is called **data locality**.

The idea is:

> Move computation to the data, not data to the computation.

Sending a tiny task (a few KB) to Node 3 is much cheaper than sending hundreds of MBs or GBs of data across the network.

---
