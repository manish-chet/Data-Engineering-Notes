### **Understanding the Core Problem in Spark**
The need for repartition and coalesce arises from issues faced when processing large datasets in Spark, particularly concerning data partitioning.



![Steps](repart.svg)

- **Uneven Partition Size (Data Skew)**: When a DataFrame is created, it's often divided into multiple partitions. Sometimes, these partitions can be of uneven sizes (e.g., 10MB, 20MB, 40MB, 100MB).

- **Impact on Processing**:
 Processing smaller partitions (e.g., 10MB) takes less time than larger ones (e.g., 100MB).
 
 This leads to idle Spark executors: while one executor is busy with a large partition, others might finish their tasks quickly and then wait for the large partition to complete. This causes time delays and underutilization of allocated resources (e.g., RAM)

- **Origin of Data Skew**: This situation often arises after operations like join transformations. For instance, if a join operation is performed on a product column, and one product is a "best-selling product" with a high number of records, all those records might get grouped into a single partition, making it very large. This phenomenon is called data skew.

- **Observed Behavior**: Users often see messages like "199 out of 200 partitions processed," where the last remaining partition takes a significantly longer time to complete due to its large size.

- **Solution**: To deal with these scenarios and optimize performance, Spark provides repartition and coalesce methods

### **repartition**

- **Shuffling Data**: repartition shuffles the entire dataset across the cluster. This means data from existing partitions can be moved to new partitions.

- **Even Distribution**: The primary goal of repartition is to evenly distribute data across the specified number of new partitions. For example, if you have 200MB of data across five uneven partitions and repartition it into five, it will aim for 40MB per partition.

- **Flexibility in Partition Count**: repartition can increase or decrease the number of partitions. If you initially have 5 partitions but need 10, repartition is the only choice.It can be used when you want to increase the number of partitions to allow for more concurrent tasks and increase parallelism when the cluster has more resources.

- **Cost**: Due to the shuffling operation, repartition is generally more expensive and involves more I/O operations compared to coalesce.

- **Pros and Cons of Repartition**: Evenly distributed data. More I/O (Input/Output) because of shuffling. More expensive.


In certain scenarios, you may want to partition based on a specific key to optimize your job. For example, if you frequently filter by a certain key, you might want all records with the same key to be on the same partition to minimize data shuffling. In such cases, you can use repartition() with a column name.

### **coalesce**

- **Merging Partitions**: coalesce merges existing partitions to reduce the total number of partitions.

- **No Shuffling (or Minimal)**: Crucially, coalesce tries to avoid full data shuffling. It achieves this by moving data from some partitions to existing ones, effectively merging them locally on the same executor if possible. This makes it less expensive than repartition.

- **Uneven Distribution**: Because it avoids full shuffling, coalesce does not guarantee an even distribution of data across the new partitions. It might result in an uneven distribution, especially if the original partitions were already skewed.

- **Limited Partition Count**: coalesce can only decrease the number of partitions. It cannot be used to increase the number of partitions. If you need more partitions, you must use repartition.

- **Pros and Cons of Coalesce**: No shuffling (or minimal shuffling). Not expensive (cost-effective). Uneven data distribution.

However, it can lead to  data skew if you have fewer partitions than before, because it combines existing partitions to reduce the total number.


### **When to Choose Which?**
The choice between repartition and coalesce is use-case dependent

- **Choose repartition when**
 
You need to evenly distribute data across partitions, which is crucial for balanced workload across executors.

You need to increase the number of partitions (e.g., if you have too few partitions or want to process data in smaller chunks in parallel).

You are okay with the overhead of a full shuffle, as the benefit of even distribution outweighs the cost.

Dealing with severe data skew is a primary concern.

- **Choose coalesce when**

You primarily need to decrease the number of partitions (e.g., after filter operations drastically reduce data, or before writing to a single file).

You want to minimize shuffling and I/O costs.

You can tolerate slightly uneven data distribution across partitions, or the data skew is minimal and won't significantly impact performance.

You want to save processing time and cost by avoiding a full shuffle

### Why doesn't `.coalesce()` explicitly show the partitioning scheme?

`.coalesce` doesn't show the partitioning scheme e.g. `RoundRobinPartitioning` because: 
- The operation only minimizes data movement by merging into fewer partitions, it doesn't do any shuffling.
- Because no shuffling is done, the partitioning scheme remains the same as the original DataFrame and Spark doesn't include it explicitly in it's plan as the partitioning scheme is unaffected by `.coalesce`