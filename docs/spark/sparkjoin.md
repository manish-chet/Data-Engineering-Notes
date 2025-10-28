Spark joins are considered an expensive operation primarily because they involve data shuffling. Shuffling refers to the movement of data across the network.

### **Join types**
- Inner Join -
    An inner join returns only the records that have matching keys in both the left and right tables. 
    If a key exists in one table but not the other, those records are excluded from the result

- Left Join (Left Outer Join) - 
    A left join returns all records from the left table and the matching records from the right table. 
    If there is no match for a record in the left table, the columns from the right table will have null values

- Right Join (Right Outer Join) - 
    A right join is symmetrical to a left join. It returns all records from the right table and the matchingrecords     from the left table. 
    If there is no match for a record in the right table, the columns from the left table will have null values

- Full Outer Join - 
    A full outer join returns all records when there is a match in either the left or the right table. This means       it includes: Records matching in both tables (like an inner join). Records unique to the left table (like a         left join, with nulls from the right).Records unique to the right table (like a right join, with nulls from the     left).
    
- Left Semi Join - 
    A left semi join returns only the records from the left table that have a match in the right table. It does not     include any columns from the right table.It acts like an inner join but only projects columns from the left         DataFrame.The presenter calls this a "fictional join" because the same result can be achieved by                   performing an inner join and then using dot select to pick only the required columns from the left table


- Left Anti Join - 
     left anti join returns only the records from the left table that do not have a match in the right table. 
     It is the inverse of a left semi join. Like left semi join, it only projects columns from the left DataFrame.

- Cross Join - 
    A cross join computes the Cartesian product of two tables. Every record from the left table is combined with       every record from the right table.This is an extremely expensive join and should generally be avoided unless       absolutely necessary.If Table A has M records and Table B has N records, a cross join will produce M * N           records.For example, if both tables have 1 million (10^6) records, the cross join will result in 1 trillion         (10^12) records, consuming massive computational resources.PySpark typically requires an explicit                   crossJoin() method or "cross" join type in join() to prevent accidental execution due to its high cost

### Why Spark Joins are Expensive: Data Shuffling (Wide Dependency Transformation)

   **Necessity of Joins**: Joins typically involve at least two DataFrames (e.g., `df1` and `df2`). The goal is to combine data based on a common key (e.g., `ID`).

   **Initial Data Distribution**: Data in Spark is broken down into partitions. For example, two 500MB DataFrames, `df1` and `df2`, would each be divided into four partitions if the default HDFS block size is 128MB (500MB / 128MB ≈ 4 partitions). These partitions are distributed across executors on worker nodes in a Spark cluster.
   
   **The Problem**: When performing a join, the matching keys from different DataFrames might reside on different executors or even different worker nodes. For instance, `ID=1` from `df1` (say, on Partition P1 of Executor 1) and `ID=1` from `df2` (say, on Partition P3 of Executor 2) cannot be joined directly because they are on separate machines.
   
   **The Solution**: Shuffling - To perform the join, all data with the same join key must be brought to the same executor (and partition). This process of moving data across the network is called shuffling.
   
   **How Shuffling Works (with Example)**:

   Default Partitions: By default, Spark creates 200 partitions when a join (or any wide dependency transformation like `groupBy`) is performed.
     
   Key Hashing: Spark uses the join key (e.g., `ID`) and the total number of partitions (200) to determine which new partition the data should go to. It uses a hash function or modulo operation: `Key % Number_of_Partitions`.
      
   **Example**:
    If `ID = 1`, then `1 % 200 = 1`. Both `df1` and `df2` records with `ID=1` will be sent to Partition 1.
    If `ID = 7`, then `7 % 200 = 7`. Records with `ID=7` will go to Partition 7.
    If `ID = 201`, then `201 % 200 = 1`. Records with `ID=201` will also go to Partition 1.
    This ensures that all records with the same join key land on the same partition on the same executor, making the join possible.
   
   **Impact of Shuffling**:  Data movement across the network is slow and resource-intensive. Excessive shuffling can choke the cluster and degrade performance significantly, especially with large datasets. This is why optimizing joins is crucial.

### Spark Join Strategies

Spark provides five main join strategies:

1.  Shuffle Sort Merge Join
2.  Shuffle Hash Join
3.  Broadcast Hash Join
4.  Cartesian Join
5.  Broadcast Nested Loop Join

#### **Shuffle Sort Merge Join (Default Strategy)**

   **Process**: 
   This is Spark's default join strategy.
   
   1. Shuffle: As explained above, data from both DataFrames is first shuffled so that records with the same join key land on the same partition of the same executor.
    
   2. Sort: After shuffling, the data within each partition is sorted by the join key.
    
   3. Merge: Once sorted, the two sorted streams of data (from `df1` and `df2` within the same partition) are merged by iterating through them and matching keys, similar to a merge sort algorithm.
   
   **Time Complexity**: The sorting step has a time complexity of O(N log N), where N is the number of records in the partition.
   
   **Resource Usage**: This strategy primarily utilizes CPU for sorting.

#### **Shuffle Hash Join**

   **Process**:

   1.  Shuffle: Similar to Shuffle Sort Merge Join, data is first shuffled to bring matching keys to the same partition.
   
   2.  Hash Table Creation: On each executor, Spark identifies the smaller of the two DataFrames for that partition. It then builds an in-memory hash table from the smaller DataFrame using the join key. The hash table stores the unique keys and their corresponding data.
   
   3.  Probe: The larger DataFrame's records are then probed against this in-memory hash table. For each record in the larger DataFrame, its join key is hashed, and a lookup is performed in the hash table to find matching records.
   
   **Time Complexity**: Once the hash table is built, lookup operations (probing) have an average time complexity of O(1).
   
   **Resource Usage & Trade-offs**:
   Building an in-memory hash table requires significant memory (RAM) on the executor. If the "smaller" table has too many unique keys or is too large, it can lead to an Out-Of-Memory (OOM) error.
    
   Compared to Shuffle Sort Merge Join (which uses CPU for sorting), Shuffle Hash Join uses less CPU for the actual join part once the hash  table is built.
   
   **Comparison to Shuffle Sort Merge Join**:
    
   Hash Joins can be faster (O(1) lookup vs. O(N log N) sort) if the hash table fits in memory.
    
   Sort Merge Join is generally more robust because it doesn't have the same risk of OOM errors as Hash Join. This is why Spark defaults to Shuffle Sort Merge Join.

#### **Broadcast Hash Join**
Broadcast Hash Join is a join strategy in Spark that combines three terms: Broadcast, Hash, and Join.
• Broadcast: Similar to how TV broadcasts spread information from one source to many receivers, in Spark, it means sending a small table from one location (the Driver) to all other locations (Executors).
• Hash: This refers to hashing on the smaller table, as discussed in previous videos about hash joins.
• Join: As learned previously, this means combining columns from two tables or DataFrames.
Why it's needed: We need Broadcast Hash Join to avoid data shuffling (shuffle). In traditional join strategies like Shuffle Hash Join or Shuffle Sort Merge Join, data needs to be moved across different executors to bring matching keys together. This shuffling can lead to significant network overhead and slow down the cluster, making it "choke". Broadcast Hash Join optimizes this process by eliminating the need for shuffling for one of the tables

How Broadcast Hash Join Works
Broadcast Hash Join is applied when you have one large table and one small table.
• The Driver is responsible for sending the small table to all Executors.
• Each Executor then has a complete copy of the small table.
• This allows each Executor to perform the join operation locally with its partition of the large table, without needing to exchange data with other Executors (i.e., no shuffling). This makes each executor "self-sufficient" for joining.
Small vs. Large Tables:
• By default, Spark considers a table smaller than 10 MB as a "small table" that can be broadcast.
• Tables greater than 10 MB are generally considered "large tables".
• For example, if you have a 1 GB table and a 5 MB table, the 5 MB table is considered small, and the 1 GB table is large.
Process Example:
1. Assume a 5 MB small table and a 1 GB large table, with the large table partitioned across multiple executors.
2. The Driver takes the 5 MB small table.
3. The Driver then sends a copy of this 5 MB table to all Executors.
4. Each Executor now has its portion of the 1 GB large table and a complete copy of the 5 MB small table.
5. Executors perform the join internally, searching for matching keys within their own partitions of the large table and the broadcasted small table.
6. Result: This avoids data shuffling, as data does not need to move between Executors to find matching keys. This optimizes performance.
3. When Broadcast Hash Join Is Not Good (Failure Scenarios)
While beneficial, Broadcast Hash Join has limitations:
• Driver Memory: The Driver needs sufficient memory to store the small table before broadcasting it. If the broadcasted file is too large (e.g., trying to broadcast a 1 GB file with a 2 GB Driver), the Driver might run "out of memory".
• Network Throughput: Broadcasting a very large file, even if the Driver has enough memory, will still involve sending the entire file over the network to all Executors. This can slow down the network.
• Executor Memory: If the broadcasted table is too large, it can fill up the Executor's memory, leaving insufficient space for join operations, potentially leading to Executor "out of memory" errors when data is multiplied during the join.
• Recommendation: It is advised to define the size of your small table based on your cluster's configuration (e.g., if an Executor has 16 GB of RAM, broadcasting a 100 MB file might be fine, but a 2 GB file could cause issues).
4. Changing the Broadcast Threshold Size
Spark provides a configuration to change the default 10 MB broadcast threshold.
• Default Threshold: The default automatic broadcast threshold is 10 MB.
• Getting the current threshold: You can retrieve the current threshold using spark.conf.get("spark.sql.autoBroadcastJoinThreshold").
• Setting a new threshold: You can set a new threshold value using spark.conf.set("spark.sql.autoBroadcastJoinThreshold", value_in_bytes).
    ◦ To disable auto-broadcasting, set the threshold to -1.
    ◦ To increase the threshold, convert the desired MB value to bytes

### Broadcast Variables
In Spark, broadcast variables are read-only shared variables that are cached on each worker node rather than sent over the network with tasks. They're used to give every node a copy of a large input dataset in an efficient manner. Spark's actions are executed through a set of stages, separated by distributed "shuffle" operations. Spark automatically broadcasts the common data needed by tasks within each stage.

    from pyspark.sql import SparkSession
    # initialize SparkSession 
    spark = SparkSession.builder.getOrCreate()
    # broadcast a large read-only lookup table
    large_lookup_table = {"apple": "fruit", "broccoli": "vegetable", "chicken": "meat"}
    broadcasted_table = spark.sparkContext.broadcast(large_lookup_table)
    def classify(word):
        # access the value of the broadcast variable  
        lookup_table = broadcasted_table.value     
        return lookup_table.get(word, "unknown")
    data = ["apple", "banana", "chicken", "potato", "broccoli"] 
    rdd = spark.sparkContext.parallelize(data)
    classification = rdd.map(classify) 
    print(classification.collect())


### Accumulators
Accumulators are variables that are only "added" to through associative and commutative operations and are used to implement counters and sums efficiently. They can be used to implement counters (as in MapReduce) or sums. Spark natively supports accumulators of numeric types, and programmers can add support for new types.

A simple use of accumulators is:
    
    from pyspark.sql import SparkSession
    # initialize SparkSession 
    spark = SparkSession.builder.getOrCreate()
    # create an Accumulator[Int] initialized to 0
    accum = spark.sparkContext.accumulator(0)
    rdd = spark.sparkContext.parallelize([1, 2, 3, 4]) 
    def add_to_accum(x):
        global accum
        accum += x rdd.foreach(add_to_accum)
    # get the current value of the accumulator 
    print(accum.value)
        
