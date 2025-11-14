###  **Data Engineering Defined**

Despite its popularity, there is much confusion about data engineering.
It has existed in some form since companies started working with data, coming into sharp focus with the rise of data science in the 2010s.

One definition states that data engineering is a set of operations aimed at creating interfaces and mechanisms for the flow and access of information. Data engineers maintain data, ensuring it remains available and usable, and they set up and operate an organization's data infrastructure for analysis by data analysts and scientists.

Data engineering is the development, implementation, and maintenance of systems and processes that take in raw data and produce high quality consistent information that supports downstream use cases, such as analysis and machine learning. Data engineering is the intersection of security, data management, DataOps, data architecture, orchestration, and software engineering.
       
Historically, there were two types of data engineering: SQL-focused (relational databases, SQL processing) and Big Data-focused (Hadoop, Cassandra, Spark, programming languages like Java, Scala, Python).

!!! note

       A data engineer **gets data, stores it, and prepares it for consumption** by data scientists, analysts.
       > — *Fundamentals of Data Engineering (O’Reilly)*

**-------------------------------------------------------------------------------------------------------------**

### **The Data Engineering Lifecycle**
The five stages are: Generation, Storage, Ingestion, Transformation, and Serving.

![Steps](DEcycle.svg)

Storage underpins other stages as it occurs throughout.

Undercurrents are critical ideas that cut across the entire lifecycle: security, data management, DataOps, data architecture, orchestration, and software engineering.

**-------------------------------------------------------------------------------------------------------------**

### **Evolution of the Field**

![Steps](DEhistory.svg)

Big Data Era (2000s and 2010s): Coincided with data explosion and cheap commodity hardware.

Big data is defined as "extremely large data sets that may be analyzed computationally to reveal patterns, trends, and associations". It's also characterized by the three Vs: velocity, variety, and volume.

Google's papers on Google File System (2003) and MapReduce (2004) were a "big bang" for data technologies.This inspired Apache Hadoop (2006) at Yahoo, leading to the birth of the big data engineer.Amazon Web Services (AWS) emerged around the same time, offering elastic computing (EC2), scalable storage (S3), and NoSQL databases (DynamoDB), creating a pay-as-you-go resource marketplace.

The era saw the rise of tools like Hadoop, Apache Pig, Apache Hive, Apache Storm, Apache Spark, and a shift from GUI-based tools to code-first engineering.
           
The transition from batch computing to event streaming ushered in "real-time" big data.

2020s: Engineering for the Data Lifecycle: The role is rapidly evolving towards decentralized, modularized, managed, and highly abstracted tools.

Data engineers are becoming data lifecycle engineers, focusing on higher-value aspects like security, data management, DataOps, data architecture, orchestration, and general data lifecycle management, rather than low-level framework details.There's a shift towards managing and governing data, making it easier to use and discover, and improving its quality.

Data engineers are now concerned with privacy, anonymization, data garbage collection, and compliance with regulations like CCPA and GDPR.

**-------------------------------------------------------------------------------------------------------------**

### **Relationship with Data Science**
Data science often struggled with basic data problems (collection, cleansing, access, transformation, infrastructure) that data engineering aims to solve.

Data scientists aren't typically trained for production-grade data systems.
Data engineers build a solid foundation for data scientists to succeed, allowing them to focus on analytics, experimentation, and ML.

Data engineering is of equal importance and visibility to data science, playing a vital role in its production success. The authors themselves moved from data science to data engineering due to this fundamental need.

**-------------------------------------------------------------------------------------------------------------**

### **Skills and Activities**

![Steps](requirements.svg)
The skillset encompasses the undercurrents: security, data management, DataOps, data architecture, and software engineering.
It requires understanding how to evaluate data tools and their fit across the lifecycle, how data is produced, and how it's consumed.

Data engineers balance cost, agility, scalability, simplicity, reuse, and interoperability.
Historically, data engineers managed monolithic technologies; now, the focus is on high-level abstractions or writing pipelines as code within orchestration frameworks.

Business Responsibilities: Communicate with technical and non-technical people, scope and gather requirements, understand business impact, and continuously learn.

Technical Responsibilities: Build architectures optimizing performance and cost, understand the data engineering lifecycle stages (generation, storage, ingestion, transformation, serving) and undercurrents, and possess production-grade software engineering chops.

SQL is a powerful tool for complex analytics and data transformation problems, essential for high productivity. Data engineers should also develop expertise in composing SQL with frameworks like Spark and Flink, or using orchestration.

**-------------------------------------------------------------------------------------------------------------**


###  **Types of Data Engineers**
Drawing from the "type A" (analysis) and "type B" (building) data scientists analogy:

Type A Data Engineers: Focus on understanding and deriving insight from data.
 
Type B Data Engineers: Share similar backgrounds but possess strong programming skills to build systems that make data science work in production.

Internal-Facing vs. External-Facing Data Engineers: Serve various end users, with primary responsibilities being external, internal, or a blend.


**-------------------------------------------------------------------------------------------------------------**

### **Data Engineers Work With (Key Technical Stakeholders)**

Data engineers are a hub between data producers (software engineers, data architects, DevOps/SREs) and data consumers (data analysts, data scientists, ML engineers).

![Steps](DEwork.svg)

- Upstream Stakeholders:

       Data Architects: Design application data layers that serve as source systems and interact across other lifecycle stages. Data engineers should understand architecture best practices.
 
       Software Engineers: Build and maintain source systems; data engineers need to understand these systems and collaborate to make data production-ready.
 
       DevOps/Site-Reliability Engineers (SREs): Ensure systems are reliable and available; data engineers collaborate on deployment, monitoring, and incident response.

- Downstream Stakeholders:

       Data Analysts: Consume data for reports, dashboards, and ad hoc analysis. Data engineers ensure data quality and provide necessary datasets.

       Data Scientists: Build models; data engineers provide the data automation and scale for data science to be efficient and production-ready.
 
       ML Engineers: Similar to data scientists, they build and deploy ML models, relying on data engineers for robust data pipelines.

**-------------------------------------------------------------------------------------------------------------**


### **Data Maturity Models**

![Steps](thinkde.svg)

Stage 1: Starting with data

Early stages, often small teams, data engineer acts as a generalist. Focus on defining the right data architecture, identifying and auditing data, and building a solid data foundation. Avoid jumping to ML without a foundation. Get quick wins (though they create technical debt), talk to people (avoid silos), and avoid undifferentiated heavy lifting.

Stage 2: Scaling with data

Company has some data, looking to expand usage. Focus on automating data flows, building systems for ML, and continuing to avoid undifferentiated heavy lifting. The main bottleneck is often the data engineering team, so focus on simple deployment/management. Shift to pragmatic leadership.

Stage 3: Leading with data

Data is a competitive advantage. Focus on automation for seamless data introduction, building custom tools for competitive advantage, "enterprisey" aspects like data management and DataOps, and deploying tools for data dissemination (catalogs, lineage). Efficient collaboration is key.

!!! note

       _“Data engineering is not about tools—it's about **building systems** that let others derive value from data.”_