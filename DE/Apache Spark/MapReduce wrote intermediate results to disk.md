That line means that **after each stage of computation, Hadoop MapReduce saved the output to storage (hard disk/HDFS) before moving to the next stage**.

Let's break it down.

Suppose you have a multi-step data processing pipeline:

1. Read logs
2. Filter errors
3. Group by user
4. Count errors per user
5. Sort results

### In Hadoop MapReduce

Each MapReduce job is separate.

```
Input Data
    ↓
Job 1 (Filter)
    ↓
Write output to disk
    ↓
Job 2 (Group)
    ↓
Write output to disk
    ↓
Job 3 (Count)
    ↓
Write output to disk
    ↓
Job 4 (Sort)
    ↓
Write output to disk
```

Even though the next step needs the data immediately, Hadoop:

- Writes the intermediate result to disk.
- Reads it back from disk for the next job.

Disk I/O is much slower than RAM, so a lot of time is spent writing and reading.

---

### Example

Imagine you start with 100 GB of data.

After filtering, you get 20 GB.

Hadoop would:

1. Process 100 GB.
2. Write 20 GB to disk.
3. Read that 20 GB back.
4. Process it.
5. Write another intermediate result.
6. Read it back again.

The CPU may finish calculations quickly, but the system keeps waiting for disk operations.

---

### Spark's improvement

Spark introduced **in-memory computation**.

Instead of:

```
Step 1 → Disk → Step 2 → Disk → Step 3
```

Spark can do:

```
Step 1 → RAM → Step 2 → RAM → Step 3
```

The data stays in memory across operations, avoiding repeated disk reads/writes.

```
Input Data
    ↓
Transform 1
    ↓
Transform 2
    ↓
Transform 3
    ↓
Final Result
```

Only the final output may be written to disk.

That's why Spark became dramatically faster—especially for machine learning, graph algorithms, and interactive analytics where the same data is processed multiple times.