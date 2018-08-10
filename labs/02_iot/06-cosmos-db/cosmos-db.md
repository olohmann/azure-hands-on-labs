# Hot Path Analytics with CosmosDB

## Overview

Azure Cosmos DB is Microsoft's globally distributed, multi-model database. The atom-record-sequence (ARS) based data model that Azure Cosmos DB is built on natively supports multiple data models, including but not limited to document, graph, key-value, table, and column-family data models

Elastically and independently scale throughput and storage on demand and worldwide

Azure Cosmos DB provides five consistency levels: strong, bounded-staleness, session, consistent prefix, and eventual. 

In this lab you will learn

* how to set up a CosmosDB
* streaming analytics with windowing techniques
* to Store Time Series Data in CosmosDB

## Task 1: Create Cosmos DB Account

Click on **Create a resource**

![Create Resource group_](./media/create_resource.png)

Click on **Databases**

![Create Databases_](./media/databases.png)

Click on **Azure Cosmos DB**

![Create Cosmos DB_](./media/01_Create_CosmosDB.png)

Please select the SQL API type:

![Create Cosmos DB_](./media/02_Create_CosmosDB_Submit.png)

> CosmosDB is a cloud native database which is able to support several APIs for interaction. 
1. SQL API: A schema-less JSON database engine with rich SQL querying capabilities.
1. MongoDB API: A massively scalable MongoDB-as-a-Service powered by Azure Cosmos DB platform. Compatible with existing MongoDB libraries, drivers, tools, and applications.
1. Cassandra API: A globally distributed Cassandra-as-a-Service powered by Azure Cosmos DB platform. Compatible with existing Apache Cassandra libraries, drivers, tools, and applications.
1. Graph (Gremlin) API: A fully managed, horizontally scalable graph database service that makes it easy to build and run applications that work with highly connected datasets supporting Open Graph APIs (based on the Apache TinkerPop specification, Apache Gremlin).
1. Table API: A key-value database service built to provide premium capabilities (for example, automatic indexing, guaranteed low latency, global distribution) to existing Azure Table storage applications without making any app changes

## Task 2: Stop Stream Analytics

To Add Cosmos DB as output to Stream Analytics Job you will need to Stop the job, Add Cosmos DB output and corresponding Query and Start the job

![Stop Stream Analytics Job_](./media/03_stop_stream_analytics_job.png)

Stream Data To Cosmos DB:

![Stream Data to Cosmos DB_](./media/04_click_output.png)

Add Cosmos DB as an Output to Stream Analytics Job

![Stream Data to Cosmos DB_](./media/05_add_cosmosdb.png)

Select Cosmos DB as an output. Also make sure you create a new Database and a collection if its is not already created

![Select Cosmos DB Account_](./media/06_create_output.png)

Edit existing query to Add new query to consume data from IoTHub and store data into Cosmos DB

**Please keep the existing "insert" statement as is. Just add the following query below:**

```sql
SELECT
    deviceId, avg(temperature) as avgtemp
INTO
    CosmosDB
FROM
    IotHub
GROUP BY deviceId, TumblingWindow(second,30)
```

![Edit Query_](./media/07_Edit_Query.png)

## Task 3: Start Streaming Again

Start Stream Analytics Job

![Start Stream Analytics Job_](./media/08_start_asa.png)

Make sure you stream all the data from when you last stopped the job. Stream Analytics interface provides an option

![Stream Data to Cosmos DB_](./media/09_when_last_stopped.png)

Make sure Stream Analytics job goes into running mode

![Stream Data to Cosmos DB_](./media/10_running.png)

## Task 4: Explore CosmosDB Data

Use Cosmos DB data explorer to view data being streamed from IoTHub to Cosmos DB

![Stream Data to Cosmos DB_](./media/11_cosmosdb_data_explorer.png)
