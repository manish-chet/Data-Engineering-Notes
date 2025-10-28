The methods persist() and cache() in Apache Spark are used to save the RDD, DataFrame, or Dataset in memory for faster access during computation. They are effectively ways to optimize the execution of your Spark jobs, especially when you have repeated transformations on the same data. However, they differ in how they handle the storage:

### **cache()**
- **What is Caching in Spark?**
Caching is an optimization technique in Spark that allows you to store intermediate results in memory. This prevents Spark from re-calculating the same data repeatedly when it is used multiple times in subsequent transformations.
- **Where is Cached Data Stored?**
When data is cached, it is stored within the Storage Memory Pool of a Spark Executor. An executor's memory is divided into three parts: User Memory, Spark Memory, and Reserved Memory. Spark Memory, in turn, contains two pools: Storage Memory Pool and Execution Memory Pool. Cached data specifically resides in the Storage Memory Pool. If the Storage Memory Pool fills up, Spark might evict (remove) data that is not frequently used (using an LRU - Least Recently Used - fashion) or spill it to disk.
- **Why Do We Need Caching?**
Spark operates with lazy evaluation, meaning transformations are not executed until an action is called. When an action is triggered, Spark builds a Directed Acyclic Graph (DAG) to determine the lineage of the data. If a DataFrame (DF) is used multiple times, Spark will re-calculate it from the beginning each time it's referenced, because DataFrames are immutable and executors' memories are short-lived

- **How Caching Helps**: By calling .cache() on df, its intermediate result is stored in the Storage Memory Pool. Now, whenever df is needed again, Spark directly retrieves it from memory instead of re-calculating it. This significantly reduces computation time and improves efficiency

- **When Not to Cache?**
You should avoid caching data when the DataFrame is very small or when its re-calculation time is negligible. Caching consumes memory, and if the benefits of caching don't outweigh the memory consumption, it's better to avoid it.

- **Limitations of Caching**:
 - Memory Fit: If the cached data's partitions are larger than the available Storage Memory Pool, the excess partitions will not be stored in memory and will either be re-calculated on the fly or spilled to disk if using a storage level that supports it. Spark does not store partial partitions; a partition is stored entirely or not at all.
 - Partition Loss: If a cached partition is lost (e.g., due to an executor crash), Spark will re-calculate it using the DAG lineage

- **How to Uncache Data()**
To remove data from the cache, you can use the .unpersist() method.

When you call df.cache(), it internally calls df.persist() with a default storage level of MEMORY_AND_DISK.

persist() offers more flexibility because it allows you to specify the desired storage level as an argument


### **persist(storageLevel)**

Storage levels define where data is stored (memory, disk, or both) and how it is stored (serialized or deserialized, and with replication). These levels provide fine-grained control over how cached data is managed, balancing performance, fault tolerance, and memory usage.
To use StorageLevel with persist(), you need to import it:
from pyspark import StorageLevel
Here are the different storage levels explained:
• MEMORY_ONLY:
    ◦ Stores data only in RAM (deserialized form).
    ◦ If memory is insufficient, partitions will be re-calculated when needed.
    ◦ Advantage: Fastest processing because data is in memory and readily accessible.
    ◦ Disadvantage: High memory utilization, potentially limiting other operations.
    ◦ Use case: For small to medium-sized datasets that fit entirely in memory and where re-calculation overhead is high.
• MEMORY_AND_DISK:
    ◦ Default for cache().
    ◦ Attempts to store data in RAM first (deserialized form).
    ◦ If RAM is full, excess partitions are spilled to disk (serialized form).
    ◦ Advantage: Provides a good balance of speed and resilience; data is less likely to be re-calculated.
    ◦ Disadvantage: Disk access is slower than memory. Data read from disk (serialized) requires CPU to deserialize it, leading to higher CPU utilization.
    ◦ Use case: For larger datasets that might not fully fit in memory but where performance is still critical.
• MEMORY_ONLY_SER:
    ◦ Stores data in RAM only, but in a serialized form.
    ◦ Serialization saves memory space, allowing more data to be stored in the same amount of RAM (e.g., 5GB uncompressed might become 8GB serialized).
    ◦ Disadvantage: Data needs to be deserialized by the CPU when accessed, leading to higher CPU utilization and slightly slower access compared to MEMORY_ONLY.
    ◦ Limitation: This serialization specifically works for Java and Scala objects, and not for Python objects (though Python has its own pickling mechanisms, the _SER storage levels in Spark are typically for JVM objects).
    ◦ Use case: When memory is a major constraint and you can tolerate increased CPU usage for deserialization.
• MEMORY_AND_DISK_SER:
    ◦ Stores data first in RAM (serialized), then spills to disk (serialized) if memory is full.
    ◦ Combines memory saving of serialization with resilience of disk storage.
    ◦ Disadvantage: High CPU usage due to deserialization for both memory and disk reads.
    ◦ Use case: For very large datasets where memory constraints are severe and some CPU overhead for deserialization is acceptable.
• DISK_ONLY:
    ◦ Stores data only on disk (serialized form).
    ◦ Slowest storage level due to reliance on disk I/O.
    ◦ Advantage: Good for extremely large datasets that don't fit in memory, or for fault tolerance where data needs to be durable across executor restarts.
    ◦ Disadvantage: Significantly slower than memory-based storage levels.
    ◦ Use case: When performance is less critical than fault tolerance or when datasets are too large for memory.
• Replicated Storage Levels (e.g., MEMORY_ONLY_2, DISK_ONLY_2):
    ◦ These levels store two copies (2x replicated) of each partition across different nodes.
    ◦ For example, MEMORY_ONLY_2 stores two copies in RAM on different executors.
    ◦ Advantage: Provides fault tolerance. If one executor or worker node goes down, the data can still be accessed from its replica, avoiding re-calculation from the DAG.
    ◦ Disadvantage: Doubles memory/disk consumption compared to non-replicated versions.
    ◦ Use case: For highly critical data that is complex to calculate and must be readily available even if a node fails. Generally, cache() (which is MEMORY_AND_DISK) is preferred unless specific fault tolerance is required

#### **Choosing the Right Storage Level**
The choice of storage level depends on your specific needs:
• Start with MEMORY_ONLY: If your data fits in memory and transformations are simple, this is the fastest.
• Move to MEMORY_AND_DISK: If data is larger and might not fit entirely in memory, or if re-calculation is expensive. This is the most commonly used for general caching.
• Consider _SER options: Only if memory is a severe bottleneck and you can tolerate increased CPU usage for serialization/deserialization. Note that in Python, direct serialization using _SER levels like in Java/Scala might not provide the same benefits.
• Use _2 options: Only for critical, complex data where high fault tolerance is a must and you can afford the doubled storage cost.
4. Code Examples and Spark UI Demonstration
The video demonstrates the effect of cache() and persist() using the Spark UI (specifically the Storage tab at localhost:4040/storage)