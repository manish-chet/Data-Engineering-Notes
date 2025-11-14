Data engineering is evolving beyond just tools and technology.  
Undercurrents are foundational practices that support the entire data engineering lifecycle.
![Steps](uc.png)

**-------------------------------------------------------------------------------------------------------------**

### **Security: A Critical Undercurrent**

Only grant users/systems the minimum access necessary to perform their function. Avoid giving admin or superuser access unless strictly required. Prevents accidental damage and maintains a security-first mindset.
Giving all users full admin access. Running commands with root privileges unnecessarily. Querying data with superuser roles when not needed.

People and organizational behavior are the biggest vulnerabilities.  
Security breaches often stem from: Neglecting security protocols, Falling for phishing, Irresponsible behavior.

Timely and Contextual Data Access: Data should be accessible: Only to the right people/systems, Only for the necessary time period.
Data must be protected in transit and at rest using: Encryption, Tokenization, Data masking, Obfuscation, Access controls.

Security is Continuous: Security is embedded across all phases of the data engineering lifecycle. Must be revisited and reinforced at every stage of data handling.

**-------------------------------------------------------------------------------------------------------------**

### **Data Management: Importance of Metadata in Data Engineering**

Metadata is not just technical—it's social.  
Airbnb emphasized human-centric metadata via tools like Dataportal to capture data ownership and usability context.

![Steps](management.svg)

Types of Metadata

- Business Metadata: Describes how data is used in business (definitions, rules, ownership).
- Technical Metadata: Covers schema, lineage, pipeline workflows, etc.
- Operational Metadata: Includes logs, stats, job run data—useful for debugging and monitoring.
- Reference Metadata: Lookup values like codes and classifications (e.g., ISO country codes).

![Steps](quality.svg)

Data Accountability: Assign a responsible person (not necessarily a data engineer) for specific data artifacts (tables, fields, logs). Supports data quality by enabling someone to coordinate governance.

Data Quality Characteristics:

- Accuracy: Data should be correct, deduplicated, and valid.
- Completeness: All required fields are filled with valid data. 
- Timeliness: Data should be available at the right time for its intended use.

Real-World Complexity: Accuracy and completeness are challenged by factors like bots or offline data submissions (e.g., ad views in video apps). Data engineers must define and enforce acceptable latency standards.

Master Data Management (MDM): Centralized "golden records" for business entities (customers, products, etc.). Combines business policy with tech (like APIs) to enforce consistency across systems and partners.

**-------------------------------------------------------------------------------------------------------------**

### **DataOps**



DataOps aims to improve the delivery, quality, and reliability of data products, just as DevOps does for software products.  
Data products, unlike software products, revolve around business logic, metrics, and decision-making processes.  
DataOps applies Agile principles, DevOps practices, and statistical process control (SPC) to the data engineering lifecycle to reduce time to value, improve data quality, and facilitate collaboration.

![Steps](ops.svg)

Key Aspects of DataOps:

- Cultural Practices: Data engineers need to foster a culture of communication, collaboration with the business, breaking silos, and continuous learning from both successes and failures.
- Automation: Automates processes like environment, code, and data version control, CI/CD, and configuration management. This ensures data workflows are reliable and consistent.
- Monitoring & Observability: Ensures that data systems and transformations are constantly monitored, with alerting and logging in place to avoid errors or delays in reporting.
- Incident Response: Focuses on identifying and resolving issues rapidly, leveraging automation, monitoring, and observability tools, with a proactive, blameless communication culture.

DataOps Lifecycle in an Organization:

- Low Maturity: A company may rely on cron jobs to schedule data transformation, which can lead to failures when jobs fail or take too long. Engineers often aren't aware of these failures until stakeholders report issues.
- Medium Maturity: Adoption of orchestration frameworks like Airflow helps automate dependencies and scheduling. However, challenges like broken DAGs may still occur, which necessitate automated DAG deployment and pre-deployment testing to prevent issues.
- High Maturity: Engineers continuously enhance automation, possibly introducing next-gen orchestration frameworks or frameworks for automatic DAG generation based on data lineage.

Core Pillars of DataOps:

![Steps](pillar1.svg)

- Automation: Ensures consistency and reliability in data product delivery by automating various aspects of the data lifecycle. This includes CI/CD and automating data quality checks, metadata integrity, and model drift.
- Observability and Monitoring: Critical to catch problems early, prevent data disasters, and keep stakeholders informed of system performance and data quality.
- Incident Response: Ensures fast and effective responses to failures or issues, using both proactive identification and retrospective resolution, supported by clear communication channels.

**-------------------------------------------------------------------------------------------------------------**

### **Data Architecture**

![Steps](architecture.svg)

Data engineers need to start by understanding the business requirements and use cases. These needs will inform the design and decisions about how to capture, store, transform, and serve data. The design of data systems must strike a balance between simplicity, cost, and operational efficiency. 

Data engineers must understand the trade-offs involved in choosing tools and technologies, whether for data ingestion, storage, transformation, or serving data. While data engineers and data architects often have distinct roles, collaboration is key. 

Data engineers should be able to implement the designs created by data architects and provide valuable feedback on those designs. With the rapid evolution of tools, technologies, and practices in the data space, data engineers must remain agile and continuously update their knowledge to maintain a relevant and effective data architecture.

**-------------------------------------------------------------------------------------------------------------**

### **Orchestration**

Orchestration is the process of managing and coordinating the execution of multiple jobs in a way that optimizes efficiency and speed. This is typically achieved through an orchestration engine like Apache Airflow, which monitors job dependencies, schedules tasks, and ensures tasks run in the correct order.

Not Just a Scheduler: Unlike simple schedulers (like cron), which only manage time-based scheduling, orchestration engines like Airflow manage complex task dependencies using Directed Acyclic Graphs (DAGs). DAGs define the order in which tasks should execute and can be scheduled for regular intervals.

Key Features of Orchestration Systems:

- High Availability: Orchestration systems should stay online continuously, ensuring they can monitor and trigger jobs without manual intervention.
- Job Monitoring & Alerts: They monitor job execution and send alerts when tasks don’t complete as expected (e.g., jobs not finishing by the expected time).
- Job History & Visualization: Orchestration systems often include visualization tools to track job progress, along with maintaining historical data to help with debugging and performance monitoring.
- Backfilling: If new tasks or DAGs are added, orchestration systems can backfill these tasks, ensuring that missing or delayed data can be processed.
- Complex Dependencies: Orchestration engines allow setting dependencies over time (e.g., ensuring a monthly reporting job doesn’t run until all necessary ETL tasks are completed for that month).

Evolution of Orchestration Tools:

- Early Tools: Traditional tools like Apache Oozie were primarily used in large enterprise environments, particularly with Hadoop. However, they were costly and not easily adaptable to different infrastructures.
- Airflow's Rise: Apache Airflow, introduced by Airbnb in 2014, revolutionized orchestration by offering an open-source, Python-based solution that is highly extensible and cloud-friendly. Airflow became widely adopted due to its flexibility and ability to manage complex workflows in modern, multi-cloud environments.
- Newer Tools: New orchestration tools like Prefect, Dagster, Argo, and Metaflow are emerging, focusing on improving portability, testability, and performance. Prefect and Dagster, in particular, aim to address issues related to portability and transitioning workflows from local development to production.

Batch vs. Streaming Orchestration:

- Batch Orchestration: Most orchestration engines focus on batch processing, where tasks are executed based on scheduled time intervals or job dependencies.
- Streaming Orchestration: For real-time or continuous data processing, orchestration becomes more complex. While streaming DAGs are difficult to implement and maintain, next-generation streaming platforms like Pulsar are attempting to simplify the process.

Orchestration plays a pivotal role in data engineering by enabling the coordination of data workflows, ensuring that tasks are executed in the correct order and monitoring their progress. Tools like Apache Airflow have made orchestration accessible to a wider range of organizations, and newer solutions continue to improve the scalability and portability of orchestration tasks across various environments.

**-------------------------------------------------------------------------------------------------------------**

### **Software Engineering**


Software engineering plays a central role in data engineering, and its importance has only grown as the field has evolved.  
While modern frameworks like Spark, SQL-based cloud data warehouses, and dataframes have abstracted much of the complexity, core software engineering skills are still crucial.

![Steps](software.svg)

Key Areas of Software Engineering for Data Engineers:

Core Data Processing Code:

- Data engineers still need to write and optimize core data processing code, which is a significant part of the data lifecycle. Whether using tools like Spark, SQL, or Beam, proficiency in these frameworks is necessary.
- Writing efficient data transformations and processing pipelines requires a solid understanding of software engineering principles, such as modularity, maintainability, and performance optimization.


Code Testing:
- Data engineers need to apply proper testing methodologies to ensure that their code is correct and reliable. This includes unit testing, regression testing, integration testing, end-to-end testing, and smoke testing.
- Testing is essential in all stages of the data pipeline to guarantee data quality and system reliability.

Development of Open Source Frameworks:

- Many data engineers are involved in creating and contributing to open-source frameworks. These tools help address specific challenges within the data engineering lifecycle, whether related to data processing, orchestration, or monitoring.
- While there is a proliferation of open-source tools like Airflow, Prefect, Dagster, and Metaflow, data engineers must evaluate the best fit for their organization's needs and consider factors like cost, scalability, and ease of use.

Streaming Data Processing:

- Streaming data processing is more complex than batch processing. Engineers face challenges with handling real-time joins, applying windowing techniques, and ensuring high throughput and low latency.
- Frameworks like Spark, Beam, Flink, and Pulsar are used for processing streaming data, and data engineers need to be familiar with them to effectively design real-time analytics and reporting systems.

Infrastructure as Code (IaC):

- As data systems move to the cloud, Infrastructure as Code (IaC) becomes increasingly important. IaC allows engineers to automate the deployment and management of infrastructure using code, improving repeatability and version control.
- Cloud services like AWS, GCP, and Azure provide IaC frameworks, such as Terraform, CloudFormation, and Kubernetes, which help automate data system management.

Pipelines as Code:

- Modern orchestration systems, such as Apache Airflow, allow engineers to define data pipelines as code. This approach provides flexibility, version control, and scalability in managing complex workflows across various stages of the data lifecycle.
- Data engineers need to be proficient in writing Python code to define tasks and dependencies, which the orchestration engine interprets and runs.

General-Purpose Problem Solving:

- Despite using high-level tools, data engineers often encounter situations where custom code is needed. This could involve writing connectors for unsupported data sources or integrating new tools into existing pipelines.
- Proficiency in software engineering allows data engineers to solve these problems by understanding APIs, handling exceptions, and ensuring smooth integration of new components into the system.

In essence, while the tools and abstractions for data engineering have advanced significantly, software engineering remains foundational. Data engineers must not only be skilled in using these tools but also in developing solutions to the unique challenges they encounter in the data engineering lifecycle.
