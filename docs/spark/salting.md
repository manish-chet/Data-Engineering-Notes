Imagine we have two DataFrames, df1 and df2, that we want to join on a column named 'id'. Assume that the 'id' column is highly skewed.

Firstly, without any handling of skewness, the join might look something like this:

    result = df1.join(df2, on='id', how='inner')

Now, let's implement salting to handle the skewness: 

    import pyspark.sql.functions as F
    # Define the number of keys you'll use for salting 
    num_salting_keys = 100
    # Add a new column to df1 for salting 
    df1 = df1.withColumn('salted_key', (F.rand()*num_salting_keys).cast('int'))
    # Explode df2 into multiple rows by creating new rows with salted keys 
    df2_exploded = df2.crossJoin(F.spark.range(num_salting_keys).withColumnRenamed('id', 'salted_key'))
    # Now perform the join using both 'id' and 'salted_key' 
    result = df1.join(df2_exploded, on=['id', 'salted_key'], how='inner')
    # If you wish, you can drop the 'salted_key' column after the join
    result = result.drop('salted_key')

In this code, we've added a "salt" to the 'id' column in df1 and created new rows in df2 for each salt value. We then perform the join operation on both 'id' and the salted key. This helps to distribute the computation for the skewed keys more evenly across the cluster.



1. What is Data Skew Problem?
• Definition: Data skew occurs when one partition contains a disproportionately large amount of data compared to other partitions. This happens when certain keys have a significantly higher frequency (e.g., ID_1 appears much more often than ID_2 or ID_3).
    ◦ Example: A "best-selling product" might account for 90% of your data, leading to a large amount of data associated with that specific product ID.
• Consequences: When performing operations like joins, if a skewed key lands on a single executor, it can slow down that executor significantly or even cause it to fail (out of memory errors or long-running tasks), leading to performance impact for the entire Spark job.
• Challenging Scenario: The problem becomes particularly acute when you need to perform a join on skewed data, and the other table is too large to be broadcast (e.g., >10 MB, >100 MB, or >200 MB), preventing the use of broadcast joins to avoid shuffles. In such situations, salting becomes a vital solution.
2. Ways to Remove Skewness (Brief Mention)
Before discussing salting, the video briefly mentions other methods:
• Repartitioning: While useful for general data distribution, it's not effective for data skew because skewed data will still end up in a single partition after repartitioning if the key remains the same.
• Adaptive Query Execution (AQE): Offers three types of optimizations that can help with skew.
• Salting: This is the primary focus of the video and is used when other methods are insufficient.
3. Understanding Joins and Skew Impact (Example)
The video uses an example of two tables to illustrate the join process and how skew impacts it:
• Table 1 (Skewed): Contains IDs with ID_1 being dominant (e.g., 5 records), ID_2 with fewer records (e.g., 2 records), and ID_3 with even fewer (e.g., 1 record).
• Table 2 (Not Skewed): Contains IDs with a balanced distribution (e.g., 2 records for ID_1, 2 for ID_2, 1 for ID_3).
• Inner Join Calculation: If an inner join is performed on these tables (e.g., table1.id == table2.id), the expected number of records would be 15 (10 from ID_1, 4 from ID_2, 1 from ID_3). The problem arises during computation, where the ID_1 data, being so large, would concentrate on one executor, slowing down the process.
4. Salting: The Core Solution
Salting involves modifying the join key to distribute the heavily skewed data across multiple partitions during the join operation.
4.1 Initial (Incorrect) Attempt at Salting
The video first demonstrates a common, but incorrect, way to implement salting, often used to test understanding in interviews:
• Left-Hand Side (Skewed Table): Create a salted_key by concatenating the original ID with a random number (e.g., from 1 to 10). This breaks down the ID_1 records into distinct keys like 1_7, 1_5, etc., distributing them into multiple partitions.
    ◦ Example: If ID_1 appears 5 times, it might become 1_7, 1_5, 1_2, 1_9, 1_4.
• Right-Hand Side (Non-Skewed Table): Similarly, create a salted_key by concatenating the original ID with a random number (1 to 10).
    ◦ Example: ID_1 records might become 1_5, 1_8.
• Problem: When these two salted tables are joined, the random numbers on both sides will likely not match, leading to a significant loss of expected records. In the example, instead of 15 records, only 3 records might be returned. This demonstrates that simply adding random numbers to both sides is not the correct approach for salting.
4.2 Correct Salting Implementation
The correct implementation of salting addresses the problem of lost records while distributing the skewed data:
• Left-Hand Side (Skewed Table - table1):
    ◦ Add a new salted_key column by concatenating the original ID with a random number between 1 and N (e.g., 1 to 10). This ensures that the skewed ID is spread across N potential new keys/partitions.
    ◦ Code Example (Conceptual for table1):
    ◦ Note: The rand() function in Spark generates a random value between 0.0 and 1.0. rand() * 10 + 1 scales this to a range of 1.0 to 11.0, and casting to IntegerType results in integers from 1 to 10.
• Right-Hand Side (Non-Skewed Table - table2):
    ◦ This is the crucial step. For each record in the non-skewed table, you need to duplicate it N times (where N is the same range used on the left side, e.g., 10 times).
    ◦ For each of these duplicated records, create a salted_key by concatenating the original ID with a sequential number from 1 to N.
    ◦ Example: If ID_1 has two records in table2, each of those two records will be duplicated 10 times. So, the first ID_1 record will generate 1_1, 1_2, ..., 1_10. The second ID_1 record will also generate 1_1, 1_2, ..., 1_10.
    ◦ Why this works: By replicating the right-hand side keys with all possible sequential suffixes (1 to 10), any salted_key generated on the left (e.g., 1_5) will find a matching key on the right-hand side, thus preserving all original join results while distributing the load.
    ◦ Code Example (Conceptual for table2):
• Joining Salted Tables: After both tables have their salted_key columns generated correctly, perform the join on these new keys.
    ◦ Code Example (Conceptual for join):
5. Implementation and Performance Demonstration
The video provides a practical demonstration using Spark code:
• Spark Session Configuration:
    ◦ spark.conf.set("spark.sql.shuffle.partitions", "3"): This is set to 3 to match the 3 distinct IDs in the example, helping to visually demonstrate partitioning.
    ◦ spark.conf.set("spark.sql.adaptive.enabled", "false"): Adaptive Query Execution (AQE) is disabled to prevent Spark from automatically changing join strategies or optimizing, allowing for a clear demonstration of the skew problem and salting solution.
• Data Generation (Simulated for Demonstration):
    ◦ table1 (skewed): Generated with 100,000 records, heavily skewed towards ID_1, then ID_2, then ID_3.
    ◦ table2 (non-skewed, but needs replication): Generated with a smaller number of records (e.g., 5 records), intended to be replicated later.
• Performance Before Salting:
    ◦ A normal inner join is performed between the original table1 (skewed) and table2.
    ◦ Spark UI Observation: When examining the Spark UI's Stages tab for the join, it clearly shows data skew.
        ▪ Some tasks finish very quickly (e.g., 0.2 seconds), while one task for the skewed data takes significantly longer (e.g., 6 seconds), even with small data. This wide disparity in task durations indicates the performance impact of data skew.
• Performance After Salting:
    ◦ The salting logic (adding random suffixes to table1, and replicating/adding sequential suffixes to table2) is applied.
    ◦ The join is then performed on the newly created salted_key columns.
    ◦ Spark UI Observation: After salting, the Spark UI shows a much more balanced distribution of task durations.
        ▪ The average task time (e.g., 9 seconds) and maximum task time (e.g., 12 seconds) are very close, indicating that the data skew problem has been resolved, and the load is evenly distributed across executors. The difference between min and max duration is drastically reduced (from 5.8 seconds to 3 seconds in the example)