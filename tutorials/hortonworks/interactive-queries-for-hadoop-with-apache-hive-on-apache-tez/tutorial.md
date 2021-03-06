---
layout: tutorial
title: Interactive Query for Hadoop with Apache Hive on Apache Tez
tutorial-id: 290
tutorial-series: Basic Development
tutorial-version: hdp-2.4.0
intro-page: true
components: [ hive, tez, ambari ]
---

# Interactive Query for Hadoop with Apache Hive on Apache Tez

### Introduction

In this tutorial, we’ll focus on taking advantage of the improvements to [Apache Hive](http://hortonworks.com/hadoop/hive) and [Apache Tez](http://hortonworks.com/hadoop/tez) through the work completed by the community as part of the [Stinger initiative](http://hortonworks.com/labs/stinger). We will explore the new features that Hive on Tez brings to HDP 2.4:

*   Performance improvements of Hive on Tez
*   Performance improvements of Vectorized Query
*   Cost-based Optimization Plans
*   Multi-tenancy with HiveServer2
*   SQL Compliance Improvements

## Pre-Requisites
*  Downloaded and Installed latest [Hortonworks Sandbox](http://hortonworks.com/products/hortonworks-sandbox/#install)
*  [Learning the Ropes of the Hortonworks Sandbox](http://hortonworks.com/hadoop-tutorial/learning-the-ropes-of-the-hortonworks-sandbox/)
*  Allow yourself around one hour to complete this tutorial

## Outline
- [Step 1: Download Data](#download-data)
- [Step 2: Upload Data Using HDFS](#upload_data_using_hdfs)
- [Step 3: Create Hive Queries](#create-hive-queries)
- [Speed Improvements](#speed-improvements)
- [Step 4: Configure MapReduce as Execution Engine in Hive view Settings Tab](#configure-mapreduce-engine-in-hive-settings)
- [Step 5: Test Query on MapReduce Engine](#test-query-mapreduce-engine)
- [Step 6: Configure Tez as Execution Engine in Hive view Settings Tab](#configure-tez-engine-in-hive-settings)
- [Step 7: Test Query on Tez Engine](#test-query-on-tez-engine)
- [Step 8: Execute Query as MapReduce Then Tez Engine](#execute-query-mapreduce-tez-engine)
- [Step 9: Track Hive on Tez Jobs](#track-hive-on-tez-jobs)
- [Stats & Cost Based Optimization (CBO)](#stats-cost-optimization)
- [Multi-tenancy with HiveServer2](#multi-tenancy-hiveserver2)
- [SQL Compliance](#sql-compliance)
- [Further Reading](#further-reading)

### Step 1: Download Data <a id="download-data"></a>

The dataset that we will need for this tutorial is [SensorFiles.zip](http://s3.amazonaws.com/hw-sandbox/tutorial14/SensorFiles.zip). Please download and save the file in a folder on your local machine.

Once you unzip the zip file – SensorFiles.zip, you will see the following files inside. We will be using these data files for the following tutorial.

![](/assets/realtime-queries-hive-on-tez/step1_sensor_files_zip_interactive_hive_tez.png)


### Step 2: Upload Data Using HDFS <a id="upload_data_using_hdfs"></a>

Let’s use the above two csv files (HVAC.csv & building.csv) to create two new tables using the following step. Navigate to [http://sandbox.hortonworks.com:8080](http://sandbox.hortonworks.com:8080) using your browser. Click the **HDFS Files** view from the dropdown menu.

![](/assets/realtime-queries-hive-on-tez/hdfs_files_view_interactive_hive_tez.png)


Go to the `/tmp` folder and if it is not already present, create a new directory called `data` using the controls toward the top of the screen. Then right-click on the folder and click **Permissions**. Make sure to check (blue) all of the permissions boxes.

![](/assets/realtime-queries-hive-on-tez/02_hdfs_files_permissions.png)


Now, let’s upload the above data files into HDFS and create two hive tables using the following steps.

Upload the two files under `/tmp/data` using **Upload** at the top of the screen

![](/assets/realtime-queries-hive-on-tez/tmp_data_upload_2files_interactive_hive_tez.png)


### Step 3: Create Hive Queries <a id="create-hive-queries"></a>

Now head on over to the Hive view

![](/assets/realtime-queries-hive-on-tez/hive_view_interactive_hive_tez.png)


We will now use hive and create the two tables. They will be named per the csv file names : “hvac” and “building”.

Use the following two queries to create the tables a then load the data

#### 3.1 Create Table building
~~~
create table building
(BuildingID int,
 BuildingMgr string,
 BuildingAge string,
 HVACproduct string,
 Country string) 
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
TBLPROPERTIES("skip.header.line.count"="1");
~~~

#### 3.2 Create Table hvac
~~~
create table hvac (
recorddate string,
Time string,
TargetTemp int,
ActualTemp int,
System int,
SystemAge int,
BuildingID int) 
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
TBLPROPERTIES("skip.header.line.count"="1");
~~~


![](/assets/realtime-queries-hive-on-tez/05_tables_created.png)

> Tables In DataBase Explorer


We're are now going to load the data into the two tables using the `LOAD DATA INPATH` Hive command

#### 3.3 Load Data into Query Tables
~~~
LOAD DATA INPATH '/tmp/data/HVAC.csv' OVERWRITE INTO TABLE hvac;
~~~

~~~
LOAD DATA INPATH '/tmp/data/building.csv' OVERWRITE INTO TABLE building;
~~~

You should now be able to obtain results when selecting small amounts of data from either table

![](/assets/realtime-queries-hive-on-tez/06_tables_loaded.png)


## Speed Improvements <a id="speed-improvements"></a>

To take a look at the speed improvements of Hive on Tez, we can run some sample queries. For this we will use the above two tables – hvac and building.

By default, the Hive view runs with Tez as it's execution engine. That's because Tez has great speed improvements over the original MapReduce execution engine. But by how much exactly are these improvements? Well let's find out!

### Step 4: Configure MapReduce as Execution Engine in Hive view Settings Tab <a id="configure-mapreduce-engine-in-hive-settings"></a>

Click on the Hive view **Settings** tab. Then we're going to need to add a new setting.

![Image hive](/assets/realtime-queries-hive-on-tez/07_hive_settings.png)


Then we're going to need to find the property which is `hive.execution.engine`. Select this property and then for it's value select, `mr` (short for MapReduce).

![Image hive](/assets/realtime-queries-hive-on-tez/08_setting_execution_engine.png)


### Step 5: Test Query on MapReduce Engine <a id="test-query-mapreduce-engine"></a>

We are now going to test a query using MapReduce as our execution engine. Execute the following query and wait for the results.

~~~
select h.*, b.country, b.hvacproduct, b.buildingage, b.buildingmgr 
from building b join hvac h 
on b.buildingid = h.buildingid;
~~~

![enter image description here](/assets/realtime-queries-hive-on-tez/09_mr_join_query.png)


This query was run using the MapReduce framework.

### Step 6: Configure Tez as Execution Engine in Hive view Settings Tab <a id="configure-tez-engine-in-hive-settings"></a>

Now we can enable Hive on Tez execution and take advantage of Directed Acyclic Graph (DAG) execution representing the query instead of multiple stages of MapReduce program which involved a lot of synchronization, barriers and IO overheads. This is improved in Tez, by writing intermediate data set into memory instead of hard disk.

Head back to the **Settings** in the Hive view and now change the `hive.execution.engine` to `tez`.

### Step 7: Test Query on Tez Engine <a id="test-query-on-tez-engine"></a>

Run the same query as we had run earlier in Step 5, to see the speed improvements with Tez.

~~~
select h.*, b.country, b.hvacproduct, b.buildingage, b.buildingmgr 
from building b join hvac h 
on b.buildingid = h.buildingid;
~~~

![enter image description here](/assets/realtime-queries-hive-on-tez/09_mr_join_query.png)


Check the output of this job. It shows the usage of the containers.  
Here is the rest of the output log:

![enter image description here](/assets/realtime-queries-hive-on-tez/10_tez_output_log.png)


Notice that the results will have appeared much quicker while having the execution engine set to Tez. This is currently the default for all Hive queries.

Congratulations! You have successfully run your Hive on Tez Job.

### Step 8: Execute Query as MapReduce Then Tez Engine <a id="execute-query-mapreduce-tez-engine"></a>

Now let’s try a new query to work with

~~~
select a.buildingid, b.buildingmgr, max(a.targettemp-a.actualtemp)
from hvac a join building b
on a.buildingid = b.buildingid
group by a.buildingid, b.buildingmgr;
~~~

Try executing the query first on MapReduce execution engine, then on Tez. You should notice a considerable gap in execution time

Here is the result.

![](/assets/realtime-queries-hive-on-tez/11_second_query_results.png)


To experience this further, you could use your own dataset, upload to your HDP Sandbox using steps above and execute with and without Tez to compare the difference.

### Step 9: Track Hive on Tez Jobs <a id="track-hive-on-tez-jobs"></a>

You can track your Hive on Tez jobs in HDP Sandbox Web UI as well. Please go to : [http://127.0.0.1:8088/cluster](http://127.0.0.1:8088/cluster) and track your jobs while running or post to see the details.

![enter image description here](/assets/realtime-queries-hive-on-tez/11_1_UI_job_tracking.jpg)


You can click on your job and see further details.

## Stats & Cost Based Optimization (CBO) <a id="stats-cost-optimization"></a>

Cost Based Optimization(CBO) engine uses statistics within Hive tables to produce optimal query plans.

### Benefits of CBO 

1.  Reduces need of a specialists to tune queries
2.  More efficient query plans lead to better cluster utilization

### Types of Stats 

There are two types of stats which could be collected so that the optimizer could use it in the decision making process :

1.  Table Stats
2.  Column Stats

The ‘explain’ plan feature can be used to see if the correct stats are being used.

    Note : CBO requires column stats. 

### Phases in which stats could be collected

1.  While data is inserted:`hive.stats.autographer =  [true,  **false**]`
2.  On existing data : table level`ANALYZE TABLE table [partion(key)] COMPUTE STATISTICS;`
3.  On existing data : column level`ANALYZE TABLE table [partion(key)] COMPUTE STATISTICS FOR COLUMNS col1,col2,...;`

### Configuration to make CBO effective for your query

1.  `hive.compute.query.using.stats =  [true,  **false**];`
2.  `hive.stats.fetch.column.stats =  [true,  **false**];`
3.  `hive.stats.fetch.partition.stats =  [true,  **false**];`
4.  `hive.cbo.enable =  [true,  **false**];`

Currently, CBO for Hive is enabled by defaults. You can see this if you head over to the Hive configuration tab in Ambari. 

![Hive Configs PIC 12](/assets/realtime-queries-hive-on-tez/12_hive_configs.png)

As you can see the CBO flag is **on**, meaning that Hive will attempt to optimize complex queries in order to shorten the execution time.

However, the only caveat is that for each table you will need to compute statistics before CBO can be utilized.

~~~
# Usage:
       # ANALYZE TABLE table [partion(key)] COMPUTE STATISTICS;
       # ANALYZE TABLE table [partion(key)] COMPUTE STATISTICS FOR COLUMNS col1,col2,...
# Example:         
         ANALYZE TABLE building COMPUTE STATISTICS FOR COLUMNS buildingage, buildingid;
~~~

Once these two commands are both executed, Hive will utilize CBO on more complex queries.

## Multi-tenancy with HiveServer2 <a id="multi-tenancy-hiveserver2"></a>

There could be contentions when multiple users run large queries simultaneously. Processing queries with many containers could lead to lower latency. For this, 3 controls could be put in place:

*   Container re-use timeout
*   Tez split wave tuning
*   Round Robin Queuing setup

### Diagnose: Job Viewer

Hive Job Viewer available in Ambari is a simple exploration and troubleshooting Graphical tool for Hive jobs.

The purposes of this Job Viewer are as follows:

*   Visualize execution DAG
*   Drill Down into individual stages for:
    *   Execution status
    *   Duration
    *   Number of bytes read and written, No of containers, etc.  
        DAG Viewer is releasing soon, which will be available in Ambari.

Run a simple query such as:

~~~
ANALYZE TABLE building COMPUTE STATISTICS FOR COLUMNS buildingage, buildingid;
~~~

To see the job executions visually, you can open the **Tez View** in Ambari Views. Tez View will take you to DAG Details, which includes all your DAG projects.

![PICTURE 13](/assets/realtime-queries-hive-on-tez/tez_view_dag_details_interactive_hive_tez.png)

> Notice the query utilized in the photo is "ANALYZE TABLE building COMPUTE STATISTICS FOR COLUMNS." After executing your query, make sure to save it, then visit "Tez View."


Try clicking on the different parts above, such as **Graphical View** and explore some of the other execution information from Tez.

![PIC 14](/assets/realtime-queries-hive-on-tez/building_tez_graphical_view.png)


## SQL Compliance <a id="sql-compliance"></a>

There are several SQL query enhancements in this version of Hive.

### Query Enhancements Support extensions:

*   Expanded Join Semantics – Supports from table1, table2 where table1.col1=table2.col2
*   IN, NOT IN subqueries in WHERE Clause
*   EXISTS and NOT EXISTS
*   Correlated Subqueries with equality operation only
*   Common Table Expressions (CTE)
*   The CHAR datatype – trailing White Space

### Authorization System enhancements:

*   SQL Authorizations : Actions
    *   Grant/Revoke
        *   Create
        *   Insert
        *   Select
        *   Drop
        *   Delete
        *   All
            *   Create Roles & Grant with admin option
            *   Using views to restrict data visibility

We will go into these in much more details in a later tutorial. You learned to perform basic hive queries, compared Hive on MapReduce and Tez Engine

## Further Reading <a id="further-reading"></a>
- [Apache Hive](http://hortonworks.com/hadoop/hive/)
- [Hive Language Manual](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL)
