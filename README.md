## Document AI

The Coalesce Document AI UDN is a node that allows you to develop and deploy a stream on top of a table, create task and Resume task. Also, provides option to create a target table to insert data from source with a task on top of it.

Document AI is an advanced AI model to extract data from documents. It can read both text and images, like logos or signatures, and is perfect for processing documents like invoices or financial statements automatically.
More information about Document AI can be found in the official [Snowflake's Introduction to Document AI](https://docs.snowflake.com/en/user-guide/snowflake-cortex/document-ai/overview).

### Document AI Node Configuration

The Document AI node has the following configuration groups:

* [Node Properties](#stream-and-insert-or-merge-node-properties)
* [General Options](#Document-AI-general-options)
* [Stream Options](#Document-AI-stream-options)
* [Source Data](#Document-AI-Source-Data)
* [Extraction Query](#Document-AI-Extraction-Query)
* [Scheduling Options](#Document-AI-scheduling-options)


#### Document AI Node Properties

| **Property** | **Description** |
|-------------|-----------------|
| **Storage Location** | Storage Location where the stream,table,task will be created |
| **Node Type** | Name of template used to create node objects |
| **Deploy Enabled** | If TRUE the node will be deployed / redeployed when changes are detected<br/>If FALSE the node will not be deployed or will be dropped during redeployment |

#### Document AI General Options

| **Option** | **Description** |
|------------|----------------|
| **Development Mode** | True / False toggle that determines whether a task will be created or if SQL executes as DML<br/>**True** - Table created and SQL executes as Run action<br/>**False** - SQL wrapped in task with specified Scheduling Options |
| **CREATE AS** | Choose target object type:<br/>- Table - Permanent table with data retention and fail-safe<br/>- Transient Table - Temporary table without data retention |
| **Truncate Before** | True / False toggle determines whether a table will be overwritten each time a task executes<br/>**True** - Uses INSERT OVERWRITE<br/>**False** - Uses INSERT to append data |


#### Document AI Stream Options

| **Option** | **Description** |
|------------|----------------|
| **Source Object** | **Directory Table**:<br/>- A directory table is an object that sits on top of a stage, similar to an external table, and stores metadata about the files in the stage. It doesn’t have its own privileges and is used to reference file-level data. Both external (cloud storage) and internal (Snowflake) stages support directory tables. You can add a directory table to a stage when creating it with CREATE STAGE or modify it later using ALTER STAGE. |
| **Redeployment Behavior** | options for Redeployment : <br/>- Create or Replace<br/>- Create if Not Exists<br/>- Create at Existing Stream |

#### Document AI Source Data

| **Option** | **Description** |
|------------|----------------|
| **Storage Location** | The Storage location Name in Coalesce |
| **Stage Name** | The Stage name we created in Snowflake |

#### Document AI Extraction Query

| **Option** | **Description** |
|------------|----------------|
| **Coalesce Storage Location of Extraction Query** | The Storage location Name in Coalesce |
| **Extraction Query Name** | The identifier or name of the query that specifies of data extraction |
| **Extraction Query Version** | The specific iteration of the query, helping to track updates or changes |
| **Pre-SQL** | SQL to execute before data insert operation |
| **Post-SQL** | SQL to execute after data insert operation |

#### Document AI Scheduling Options

| **Option** | **Description** |
|------------|----------------|
| **Scheduling Mode** | Choose compute type:<br/>- **Warehouse Task**: User managed warehouse executes tasks<br/>- **Serverless Task**: Uses serverless compute |
| **When Source Stream has Data Flag** | True/False toggle to check for stream data<br/>**True** - Only run task if source stream has capture change data<br/>**False** -  Run task on schedule regardless of whether the source stream has data. If the source is not a stream should set this to false. |
| **Select Warehouse on which to run task** | Visible if Scheduling Mode is set to Warehouse Task. Enter the name of the warehouse you want the task to run on without quotes.|
| **Select initial serverless Warehouse size** | Visible when Scheduling Mode is set to Serverless Task.<br/> Select the initial compute size on which to run the task. Snowflake will adjust size from there based on target schedule and task run times. |
| **Task Schedule** | Choose schedule type:<br/>- **Minutes** - Specify interval in minutes.<br/>- **Cron** - . Specifies a cron expression and time zone for periodically running the task. Supports a subset of standard cron utility syntax.|
| **Enter task schedule using minutes**| Visible when Task Schedule is set to Minutes.<br/> Enter a whole number from 1 to 11520 which represents the number of minutes between task runs |
| **Enter task schedule using CRON**| For more reference visit [Cron expressions](https://docs.coalesce.io/docs/reference/cron-reference/) |

### Stream and Insert or Merge System Columns

| **Column** | **Description** |
|------------|----------------|
| **METADATA$ACTION** | Indicates the DML operation (INSERT, DELETE) recorded |
| **METADATA$ISUPDATE** | Indicates whether operation was UPDATE (shown as DELETE/INSERT pair with TRUE value) |
| **METADATA$ROW_ID** | Unique and immutable row ID for change tracking |

### Stream and Insert or Merge Deployment

#### Stream and Insert or Merge Deployment Parameters

The Dimension with Task includes an environment parameter that allows you to specify a different warehouse used to run a task in different environments.

The parameter name is `targetTaskWarehouse` with default value `DEV ENVIRONMENT`.

```json
{
    "targetTaskWarehouse": "DEV ENVIRONMENT"
}
```

When set to any value other than `DEV ENVIRONMENT` the node will attempt to create the task using a Snowflake warehouse with the specified value.

For example, with the below setting for the parameter in a QA environment, the task will execute using a warehouse named `compute_wh`.

```json
{
    "targetTaskWarehouse": "compute_wh"
}
```

#### Stream and Insert or Merge Initial Deployment

For tasks without predecessors:

| **Stage** | **Description** |
|-----------|----------------|
| **Create Stream** | Creates Stream in target environment |
| **Create Work Table/Transient Table** | Creates table loaded by task |
| **Target Table Initial Load** | Loads initial data |
| **Create Task** | Creates scheduled task |
| **Resume Task** | Enables task execution |

For tasks with predecessors:

| **Stage** | **Description** |
|-----------|----------------|
| **Create Stream** | Creates Stream in target environment |
| **Create Work Table/Transient Table** | Creates target table |
| **Target Table Initial Load** | Loads initial data |
| **Suspend Root Task** | Suspends root task |
| **Create Task** | Creates scheduled task |

If a task is part of a DAG of tasks, the DAG needs to include a node type called `Task DAG Resume Root`. This node will resume the root node once all the dependent tasks have been created as part of a deployment.

The task node has no ALTER capabilities. All task-enabled nodes are CREATE OR REPLACE only, though this is subject to change

#### Stream and Insert or Merge Redeployment

Stream redeployment behavior:

| **Redeployment Behavior** | **Stage Executed** |
|--------------------------|-------------------|
| **Create Stream if not exists** | Re-Create Stream at existing offset |
| **Create or Replace** | Create Stream |
| **Create at existing stream** | Re-Create Stream at existing offset |

Table changes execute:

| **Stage** | **Description** |
|-----------|----------------|
| **Rename Table/Alter Column/Delete Column/Add Column/Edit description** | Alters table as needed |
| **Target Initial Load** | If the initial load toggle is enabled and the redeployment behavior of the stream is "Create or Replace," it loads the table with "INSERT OVERWRITE INTO." For all other scenarios, it uses "INSERT INTO." |

If the materialization type is changed from one type to another(transient table/table) the following stages execute:

| **Stage** | **Description** |
|-----------|----------------|
| **Drop Table/Transient Table** | Drop the target table|
| **Create Work/Transient table**| Create the target table|
| **Target Initial Load** | If the initial load toggle is enabled and the redeployment behavior of the stream is "Create or Replace," it loads the table with "INSERT OVERWRITE INTO." For all other scenarios, it uses "INSERT INTO." |

Task changes:

| **Stage** | **Description** |
|-----------|----------------|
| **Create Task** | Creates scheduled task |
| **Resume Task**| Resumes the task|

### Stream and Insert or Merge Undeployment

When node is deleted, the following stages execute:

| **Stage** | **Description** |
|-----------|----------------|
| **Drop Stream** | Removes the stream |
| **Drop Table** | Drop the table |
| **Drop Current Task** | Drop the task |

---
