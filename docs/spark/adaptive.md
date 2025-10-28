### **Understanding Adaptive Query Execution (AQE) in Spark**

Adaptive Query Execution (AQE) is a powerful feature introduced in Spark 3.0 and later versions that provides flexibility to dynamically change query execution plans at runtime. This means that Spark can adapt and optimize your query execution based on actual data characteristics and runtime statistics, rather than relying solely on initial, compile-time plans.

The primary purpose and benefit of AQE is to significantly improve query performance. For instance, if an initial plan decides on a Sort-Merge Join for two large tables, but after intermediate transformations (like filters) one table becomes small enough to be broadcasted, AQE can dynamically switch to a Broadcast Join, which is much faster.

### **Why AQE is Needed**

AQE addresses several common performance issues in Spark queries, especially those arising from data skew or suboptimal initial query plans:

- Dynamic Optimization: Initial query plans might be suboptimal because they are formed based on estimated data sizes. Data sizes can change significantly after transformations like filters are applied. AQE continuously monitors runtime statistics to make better decisions.
- Improved Performance: By dynamically adapting strategies (like join types) and optimizing partition management, AQE can drastically reduce query execution time.
- Resource Efficiency: It helps in better utilization of cluster resources by reducing wasted CPU cores and managing tasks more effectively.
- Simplified Partition Management: It removes the burden from users to manually define the optimal number of partitions, as AQE can dynamically coalesce them.

### **Key Capabilities of AQE**

AQE provides three main capabilities to dynamically optimize Spark queries at runtime:

#### **Dynamically Coalescing Shuffle Partitions**
This feature helps manage the number and size of partitions after a shuffle operation, particularly when dealing with data skew.

The Problem Before AQE (or without this feature enabled):

- When a wide dependency transformation like groupBy (which involves a shuffle) is performed, Spark typically creates 200 partitions by default.
- If your data is skewed (e.g., 80% of sales come from one product like "sugar," while other products contribute very little), after shuffling, you'll end up with a few very large partitions (for the skewed key, like sugar) and many very small or empty partitions.
- For example, if you have 5 distinct product keys and 200 default shuffle partitions, you might have 5 non-empty partitions and 195 empty partitions.
- The Spark scheduler still has to schedule and monitor tasks for all 200 partitions, even the empty ones, leading to resource wastage (CPU cores) and increased overhead.
- Furthermore, the single large partition for the skewed key (e.g., "sugar") will take a significantly longer time to process than the smaller ones, leading to a situation where "199 out of 200 tasks complete, but one task takes a very long time".

How AQE Solves It (Dynamically Coalescing):

- AQE's entry point is exactly where data begins to shuffle. It has an "AQE shuffle reader" that reads the shuffled data.
- AQE dynamically merges (coalesces) these small and empty shuffle partitions into a fewer, more manageable number of partitions.
- Conceptual Example:
    - Imagine you have two initial data partitions (P1, P2) where "sugar" data is prevalent, and other products (yellow, blue, white, red) are less frequent.
    - After a groupBy and shuffle, "sugar" data might consolidate into one large partition (P1), while other products form smaller partitions (P2, P3, P4, P5), but many (195 out of 200) default partitions remain empty.
    - AQE observes these small, manageable partitions (P2, P3, P4, P5) and merges them together into a single, larger, more balanced partition.
    - This reduces the total number of active tasks. If there were 5 active tasks initially (1 for sugar, 4 for other products), after coalescing the 4 small ones, there might be only 2 active tasks: one for sugar, and one for the merged smaller products.
- Benefits:
    - Reduced number of tasks: Fewer tasks mean less scheduling and monitoring overhead for Spark.
    - Saved CPU Cores: If two tasks are merged into one, you save one CPU core.
    - Balanced Workload (Partial): While the skewed "sugar" partition might still be large, other smaller partitions are balanced, leading to faster completion for those.
    - Dynamic Partition Sizing: Instead of pre-defining a fixed number of partitions, AQE dynamically determines the optimal number based on actual data characteristics.

Handling Highly Skewed Data (Splitting):

- Even with coalescing, if one partition (like the "sugar" partition) remains extremely large (e.g., 80% of data), it can still cause "1 out of 2 tasks" to be very slow.
- In such cases, AQE can split the highly skewed large partition into multiple smaller ones.
- Conditions for Splitting Skewed Data: A partition will be split if both of these conditions are met:
    1. The size of the skewed data partition is greater than 256 MB (e.g., 80% of data is very large).
    2. The size of the skewed data is more than 5 times the median size of all other partitions (e.g., if the median partition size is 5 MB, the skewed partition must be > 25 MB and also > 256 MB for splitting to occur).
- Conceptual Example: The "sugar" partition (80% of data) could be split into four smaller partitions, each approximately 20% of the data, to balance the workload. This results in tasks that complete in similar times.

#### **Dynamically Switching Join Strategy**

AQE can change the type of join executed at runtime based on the actual size of the tables after transformations.

The Problem Before AQE:
- When Spark initially creates the query plan (DAG), it might decide on a Sort-Merge Join if both tables involved in the join are large (e.g., Table 1 is 10 GB, Table 2 is 20 GB). Sort-Merge Join involves shuffling and sorting, which can be computationally expensive.
- However, after applying multiple transformations, such as filters, to these tables, their sizes can significantly reduce. For example, Table 1 might become 8 GB, and Table 2 might become just 5 MB (less than 10 MB).
- Without AQE, the initial plan would still execute a Sort-Merge Join, even though a much faster join strategy is now possible.
How AQE Solves It (Dynamically Switching):
- AQE continuously monitors the runtime statistics of tables during execution.
- If, after transformations, one of the tables becomes small enough to fit into memory (typically less than a configured broadcast threshold, often around 10MB or more depending on configuration), AQE will dynamically switch the join strategy to a Broadcast Join.
- Conceptual Example:
    - Initial plan: Join Table 1 (10 GB) and Table 2 (20 GB) on product_name key. Spark defaults to Sort-Merge Join.
    - Runtime transformation: Filters are applied, reducing Table 2 to 5 MB.
    - AQE action: Since Table 2 is now small enough, AQE detects this at runtime and changes the join strategy from Sort-Merge Join to Broadcast Join.
- Benefits:
    - Significantly Faster Joins: Broadcast Join is generally the fastest join strategy as it avoids shuffling and sorting the larger table, instead broadcasting the smaller table to all executor nodes.
    - Runtime Adaptability: AQE ensures that the most optimal join strategy is used based on the actual data sizes encountered during execution, not just initial estimates.
- Note: While it optimizes join strategy, shuffling might still occur for the other (larger) table depending on the overall query plan, but the expensive Sort-Merge operation for the join itself is avoided.

#### **Dynamically Optimizing Skewed Joins**

This feature specifically tackles performance bottlenecks caused by data skew during join operations.
The Problem Before AQE:
- When joining two tables, even if they initially appear large, the data might be skewed on the join key. For example, if "sugar" product data is 80% of the sales in both tables, and these tables are being joined on the "product name" key.
- After shuffling to bring matching keys together, a single partition corresponding to the skewed key (e.g., "sugar") can become extremely large.
- This large partition can cause issues:
    - "Out Of Memory Exception": Executors trying to process this single massive partition might run out of memory.
    - Long-running Tasks: Similar to shuffle coalescing, this skewed partition will take a disproportionately long time to process, leading to the "199 out of 200 tasks complete, but one task is stuck" scenario.
    - Query Failure: The query might eventually fail due to memory issues or timeouts.
How AQE Solves It (Splitting Skewed Partitions and Duplicating Data):
- AQE utilizes its "AQE shuffle reader" (also used for coalescing) to identify and analyze skewed partitions during the join process.
- If a partition is identified as skewed (based on the same conditions as coalescing, i.e., > 256MB and > 5x median size), AQE will split this large, skewed partition into multiple smaller partitions.
- Conceptual Example:
    - You have two tables, Table 1 (left side) and Table 2 (right side), both with skewed data where "sugar" is the dominant key.
    - After the initial shuffle and grouping by the join key, the "sugar" partition becomes excessively large on both sides of the join.
    - AQE detects this skew in the "sugar" partition.
    - AQE splits the large "sugar" partition on the left side into multiple smaller partitions (e.g., two parts: P1 and P2).
    - To enable these new smaller partitions to join correctly, AQE duplicates the corresponding data for the skewed key from the right-hand side table. This means if the left-side sugar partition is split into two, the right-side sugar data will be duplicated so that each of the two new left-side partitions has a copy of the right-side sugar data to join with.
- Benefits:
    - Avoids Out Of Memory (OOM) Errors: By breaking down large partitions, the memory footprint per task is reduced, preventing OOM exceptions.
    - Balanced Task Execution: All tasks, including those processing previously skewed data, complete in a more balanced and timely manner.
    - Self-Sufficient Partitions: Each new, smaller partition becomes self-sufficient for performing the join, improving parallelism and overall performance.
