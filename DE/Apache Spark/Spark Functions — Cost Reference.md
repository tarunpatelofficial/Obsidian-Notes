### Narrow (Cheap — no shuffle)

These run partition-by-partition. No data moves across the network.

|Function|What it does|
|---|---|
|`select()`|Pick columns|
|`filter()` / `where()`|Row-level filtering|
|`withColumn()`|Add or modify a column|
|`drop()`|Remove a column|
|`cast()`|Change column type|
|`map()`|Transform each row (RDD)|
|`flatMap()`|One row → many rows (RDD)|
|`union()` / `unionAll()`|Stack two DFs vertically|
|`limit()`|Take first N rows|
|`sample()`|Random sample|
|`toDF()`|Rename columns|
|`na.fill()` / `na.drop()`|Handle nulls|
|`lit()`|Add literal/constant column|
|`alias()`|Rename a column|
|`explode()`|Array/map → multiple rows|

---

### Wide (Expensive — causes shuffle)

These require data to move across the network between executors.

|Function|Why expensive|
|---|---|
|`groupBy()` + `agg()`|Groups same keys onto same node|
|`join()` (sort-merge)|Both sides shuffled and sorted|
|`distinct()`|All rows compared across cluster|
|`orderBy()` / `sort()`|Global sort = full shuffle|
|`repartition(n)`|Redistributes all data, full shuffle|
|`reduceByKey()`|Key-based aggregation (RDD)|
|`groupByKey()`|Worst offender — sends all values over network|
|`pivot()`|GroupBy + transpose, double shuffle|
|`crossJoin()`|Cartesian product — avoid almost always|
|`cogroup()`|Group two RDDs by key|
|`intersection()`|Compares both datasets fully|
|`subtract()`|Compares both datasets fully|

---

### Intermediate (Depends on context)

|Function|Why it depends|
|---|---|
|`coalesce(n)`|Reduces partitions without full shuffle — but can create skew|
|`broadcast(df).join()`|Join becomes cheap if small table fits in memory|
|`cache()` / `persist()`|First computation is normal cost, subsequent uses are free|
|`window()` functions|Shuffle to group by partition key, then cheap within partition|
|`dropDuplicates()`|Cheaper than `distinct()` if you specify columns|
|`sortWithinPartitions()`|Sorts locally only, no global shuffle|
|`mapPartitions()`|Cheap per partition but can be heavy if logic is complex|
|`zip()`|Cheap if same number of partitions, error if not|

---

### The mental rule

> **Does this function need to see data from other partitions to do its job?**
> 
> - No → narrow, cheap
> - Yes → wide, shuffle, expensive

`filter` only needs its own rows. Cheap. `groupBy` needs all rows with the same key in one place. Expensive.