RDDs are the building blocks of any Spark application. 

RDDs Stands for

- **Resilient**: Fault tolerant and is capable of rebuilding data on failure
- **Distributed**: Distributed data among the multiple nodes in a cluster
- **Dataset**: Collection of partitioned data with values


Here are some key points about RDDs and their properties:

- **Fundamental Data Structure**: RDD is the fundamental data structure of Spark, which allows it to efficiently operate on large-scale data across a distributed environment.

- **Immutability**: Once an RDD is created, it cannot be changed. Any transformation applied to an RDD creates a new RDD, leaving the original one untouched.

- **Resilience**: RDDs are fault-tolerant, meaning they can recover from node failures. This resilience is provided through a feature known as lineage, a record of all the transformations applied to the base data.

- **Lazy Evaluation**: RDDs follow a lazy evaluation approach, meaning transformations on RDDs are not executed immediately, but computed only when an action (like count, collect) is performed. This leads to optimized computation.

- **Partitioning**: RDDs are partitioned across nodes in the cluster, allowing for parallel computation on separate portions of the dataset.

- **In-Memory Computation**: RDDs can be stored in the memory of worker nodes, making them readily available for repeated access, and thereby speeding up computations.

- **Distributed Nature**: RDDs can be processed in parallel across a Spark cluster, contributing to the overall speed and scalability of Spark.

- **Persistence**: Users can manually persist an RDD in memory, allowing it to be reused across parallel operations. This is useful for iterative algorithms and fast interactive use.

- **Operations**: Two types of operations can be performed on RDDs - transformations (which create a new RDD) and actions (which return a value to the driver program or write data to an external storage system).

### **When to Use RDDs (Advantages)**
Despite the general recommendation to use DataFrames/Datasets, RDDs have specific use cases where they are advantageous:

- **Unstructured Data**: RDDs are particularly well-suited for processing unstructured data where there is no predefined schema, such as streams of text, media, or arbitrary bytes. For structured data, DataFrames and Datasets are generally better.

- **Full Control and Flexibility**: If you need fine-grained control over data processing at a very low level and want to optimize the code manually, RDDs provide that flexibility. This means the developer has more control over how data is transformed and distributed.

- **Type Safety (Compile-Time Errors)**: RDDs are type-safe. This means that if there's a type mismatch (e.g., trying to add an integer to a string), you will get an error during compile time, before the code even runs. This can help catch errors earlier in the development cycle, unlike DataFrames or SQL queries which might only show errors at runtime


### **Why You Should NOT Use RDDs (Disadvantages)**
For most modern Spark applications, especially with structured or semi-structured data, RDDs are generally discouraged due to several drawbacks:

- **No Automatic Optimization by Spark**: Spark's powerful Catalyst Optimizer does not perform optimizations automatically for RDD operations. This means the responsibility for writing optimized and efficient code falls entirely on the developer.

- **Complex and Less Readable Code**: Writing RDD code can be complex and less readable compared to DataFrames, Datasets, or SQL. The code often requires explicit handling of data transformations and aggregations, which can be verbose.

- **Potential for Inefficient Operations**: Expensive Shuffling: Without Spark's internal optimizations, RDD operations can lead to inefficient data shuffling. In contrast, DataFrames/Datasets using the "what to" approach allow Spark to rearrange operations (e.g., filter first, then shuffle) to optimize performance, saving significant computational resources.
  
!!! example
    If you perform a reduceByKey (which requires shuffling data across nodes) before a filter operation, Spark will shuffle all the data first, then filter it. If the filter significantly reduces the dataset size, shuffling the larger pre-filtered dataset becomes a very expensive operation.

- **Developer Burden**: Because Spark doesn't optimize RDDs, the developer must have a deep understanding of distributed computing and Spark's internals to write performant RDD code. This makes development harder and slower compared to using higher-level APIs

### **Difference**

| **Criteria**         | **RDD (Resilient Distributed Dataset)**                                           | **DataFrame**                                                                                    | **DataSet**                                                                                    |
| -------------------- | --------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------- |
| **Abstraction**      | Low level, provides a basic and simple abstraction.                               | High level, built on top of RDDs. Provides a structured and tabular view on data.                | High level, built on top of DataFrames. Provides a structured and strongly-typed view on data. |
| **Type Safety**      | Provides compile-time type safety, since it is based on objects.                  | Doesn't provide compile-time type safety, as it deals with semi-structured data.                 | Provides compile-time type safety, as it deals with structured data.                           |
| **Optimization**     | Optimization needs to be manually done by the developer (like using `mapreduce`). | Makes use of Catalyst Optimizer for optimization of query plans, leading to efficient execution. | Makes use of Catalyst Optimizer for optimization.                                              |
| **Processing Speed** | Slower, as operations are not optimized.                                          | Faster than RDDs due to optimization by Catalyst Optimizer.                                      | Similar to DataFrame, it's faster due to Catalyst Optimizer.                                   |
| **Ease of Use**      | Less easy to use due to the need of manual optimization.                          | Easier to use than RDDs due to high-level abstraction and SQL-like syntax.                       | Similar to DataFrame, it provides SQL-like syntax which makes it easier to use.                |
| **Interoperability** | Easy to convert to and from other types like DataFrame and DataSet.               | Easy to convert to and from other types like RDD and DataSet.                                    | Easy to convert to and from other types like DataFrame and RDD.                                |
