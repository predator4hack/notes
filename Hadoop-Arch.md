# Architecture

![img](./assets/Screenshot%202025-09-30%20at%201.43.55 AM.png)

## Airflow

Apache Airflow is a workflow orchestration platform that schedules and monitors data pipelines. Think of it as the conductor of an orchestra - it doesn't process data itself, but it coordinates when and in what order tasks should run.

-   Creates DAGs (Directed Acyclic Graphs) to define task dependencies
-   Triggers Spark jobs, runs data quality checks, sends alerts
-   Provides a UI to monitor pipeline health and retry failed tasks
-   Handles complex scheduling (hourly, daily, conditional triggers)

## Spark Job

Apache Spark is a distributed data processing engine that performs the actual computation on your data. It's extremely fast because it processes data in-memory across multiple machines.

-   Reads data from HDFS, processes it (transformations, aggregations, joins)
-   Writes results back to HDFS or directly into Hive tables
-   Handles both batch processing and streaming data
-   Much faster than traditional MapReduce for iterative algorithms

## Hadoop Cluster (HDFS)

HDFS (Hadoop Distributed File System) is your distributed storage layer. It stores massive datasets by breaking them into blocks and distributing them across multiple machines.

-   Provides fault-tolerant storage (data is replicated across nodes)
-   Optimized for large files and sequential reads
-   Acts as the data lake where raw and processed data lives
-   Stores data for both Spark and Hive to access

## YARN (Yet Another Resource Negotiator)

YARN is the cluster resource manager for your Hadoop cluster. It's the brain that decides which applications get which resources.

-   Manages CPU, memory allocation across the cluster
-   ResourceManager: Coordinates resource allocation globally
-   NodeManagers: Run on each node and manage resources locally
-   In your architecture: When Airflow triggers a Spark job, YARN allocates containers (resources) on the cluster for Spark to use

## Hive Tables

Apache Hive provides a SQL-like interface over data stored in HDFS. It's essentially a data warehouse solution that allows you to query big data using familiar SQL syntax.

-   Creates a schema/table structure on top of HDFS files
-   Enables analysts to query data using HiveQL (similar to SQL)
-   Stores metadata about tables in the Hive Metastore
-   Underlying data still lives in HDFS

## Data Nodes vs. Compute - Data Locality

HDFS has DataNodes that store data, and YARN allocates compute resources using NodeManagers that typically run on the same physical machines as the DataNodes. This is a key design principle called data locality.
How it works:

-   Your cluster has nodes (servers), each running both:
    -   DataNode (HDFS storage service)
    -   NodeManager (YARN compute service)
-   When Spark requests resources from YARN, YARN tries to allocate containers on nodes where the data already exists
-   Spark executors run in containers on these same nodes
