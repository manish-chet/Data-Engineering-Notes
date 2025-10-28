Both Spark Session and Spark Context serve as the entry point into a Spark cluster, similar to how a `main` method serves as the entry point for code execution in languages like C++ or Java. This means that to run any Spark code, you first need to establish one of these entry points.

### **Spark Session**

The Spark Session is the unified entry point introduced in Spark 2.0. It is now the primary way to interact with Spark.

- **Unified Entry Point**: Prior to Spark 2.0 (specifically up to Spark 1.4), if you wanted to work with different Spark functionalities like SQL, Hive, or Streaming, you had to create separate contexts for each (e.g., `SQLContext`, `HiveContext`, `StreamingContext`). The Spark Session encapsulates all these different contexts, providing a single object to access them. This simplifies development as you only need to create a Spark Session to gain access to all necessary functionalities.

- **Resource Allocation**: When you create a Spark Session, you can pass configurations for resources needed, such as the amount of memory or the number of executors. The Spark Session takes these values and communicates with the Resource Manager (like YARN or Mesos) to request and allocate the necessary driver memory and executors. Once these resources are secured, the Spark Session facilitates the execution of your Spark jobs within that allocated environment.

- **Default in Databricks**: If you've been using Databricks notebooks, you might have implicitly been using a Spark Session without realizing it. Databricks typically provides a default `spark` object, which is an instance of `SparkSession`, allowing you to directly write code like `spark.read.format(...)`. This is why the local setup is demonstrated in the source, as the default session is not automatically provided outside environments like Databricks.

- **Configuration**: You can configure properties like `spark.driver.memory` by using the `.config()` method when building the Spark Session.

### **Spark Context**

The Spark Context (`SparkContext`) was the original entry point for Spark applications before Spark 2.0.

- **Historical Role**: In earlier versions of Spark (up to Spark 1.4), `SparkContext` was the primary entry point for general Spark operations. However, for specific functionalities like SQL, you needed additional context objects like `SQLContext`.
   
- **Current Role (RDD Level)**: While Spark Session has become the dominant entry point, `SparkContext` is still relevant for RDD (Resilient Distributed Dataset) level operations. If you need to perform low-level transformations directly on RDDs (e.g., `flatMap`, `map`), you would typically use the Spark Context. An example provided is writing a word count program using RDDs, where `SparkContext` comes into use.

- **Access via Spark Session**: With the advent of Spark Session, you do not create a `SparkContext` directly as a separate entry point anymore. Instead, you can access the `SparkContext` object through the `SparkSession` instance. This means that the `SparkContext` is now encapsulated within the `SparkSession`.

### **Code Example**

Here’s an example demonstrating how to create a Spark Session and then obtain a Spark Context from it, based on the provided transcript:

```python
# First, import the SparkSession class from pyspark.sql
from pyspark.sql import SparkSession

# Create a SparkSession builder
# The .builder() method is used to construct a SparkSession instance
spark_builder = SparkSession.builder

# Configure the SparkSession
# .master("local"): Specifies that Spark should run in local mode.
#                  This means Spark will use your local machine's resources.
# .appName("Testing"): Sets the name of your Spark application.
#                     This can be any descriptive name for your project.
# .config("spark.driver.memory", "12g"): An optional configuration to request specific resources,
#                                       here requesting 12GB for the driver memory.
spark_session_config = spark_builder.master("local").appName("Testing").config("spark.driver.memory", "12g")

# Get or Create the SparkSession
# .getOrCreate(): This is a crucial method. If a SparkSession with the specified
#                name and configuration already exists, it will retrieve it.
#                Otherwise, it will create a new one.
spark = spark_session_config.getOrCreate()

# Print the SparkSession object to verify it's created
print(spark)

# Access the SparkContext from the created SparkSession
# The SparkContext is encapsulated within the SparkSession object.
# This is how you get the sc (SparkContext) object in modern Spark applications.
sc = spark.sparkContext

# Print the SparkContext object
print(sc)

# Example of using SparkSession to read data (common operation)
# This is what you often do in Databricks without explicitly creating a session.
# employee_df = spark.read.format("csv").load("path/to/employee_data.csv")
```