## DynamoDB Architecture
- NoSQL Public Database as a Service (DBaaS)
- Wide column Key/Value database.
- Not like RDS which is a Database Server as a Product. This is only the database.
- You can take full control and provisioned capacity or use on-demand mode and set it and forget it.
- This is highly resilient, across AZs and optionally globally resilient.
- Data is replicated across multiple storage nodes by default so doesn't need to be set up or managed.
- Really fast, single digit millisecond access to data.
- Supports backups and encryption at rest.
- It allows event-driven integration. Do things when data changes.

### Dynamo DB Tables
- A table is a grouping of items which share the same primary key.
- There is no limit to the number of items in a table.
- When configuring Dynamo, you need to pick a primary key from two types
  - Simple (Partition)
  - Composite (Partition and Sort)
- Every item in the table needs a unique primary key.
- The attributes may or may not be there. This is not necessary.
- Each item can be at most 400KB in size.
- In DynamoDB, capacity means speed. If you choose on-demand capacity model you don't have to worry about capacity. You only pay for the operations for the table.
- If you choose provisioned capacity, you must set this on a per table basis.
- Capacity is set per WCU or RCU
  - 1 WCU means you can write 1KB per second to that table
  - 1 RCU means you can read 4KB per second for that table

### Backups
#### On-demand Backups 
- Similar to manual RDS snapshots.
- Full backup of the table that is retained until you manually remove that backup
- This can be used to restore data in the same region or cross-region
- You can adjust indexes, or adjust encryption settings.

#### Point-in-time Recovery 
- Must be enabled on each table and is off by default.
- This allows continous record of changes for 35 days to allow you to replay any point in that window to a 1 second granularity.

### Considerations
- If you see NoSQL, you should jump towards DynamoDB.
- If you see relational data, this is not DynamoDB.
- If you see key value and DynamoDB is an answer, this is likely the proper choice.
- Access to Dynamo is from the console, CLI, or API. There is No SQL available.
- Billing based on:
  - RCU and WCU
  - Storage on that table
  - Additional features on that table
- Can purchase reserved capacity with a cheaper rate for a longer term commit.

## DynamoDB Operations, Consistency, and Performance
### Reading and Writing
- **On-Demand**- unknown or unpredictable load on a table
- **On-Demand** - Pay a price per million Read or Write units
- **Provisioned** - RCU and WCU set on a per table basis (check above for what RCU and WCU entails)
- Every operation consumes at least 1 RCU/WCU
- Every single table has a WCU and RCU burst pool. This is 300 seconds of RCU or WCU as set by the table.

### Query
- Query accepts a single PK value and **optionally** a SK or range.
- Capacity consumed is the size of all returned items.
- Further filtering discards data, capacity is still consumed.
- It is always more efficent to pull as much data as needed per query to save RCU.
- If you filter data and only look at one attribute, you will still be charged for pulling all the attributes against that query.

### Scan
- Least efficent when pulling data from Dynamo, but the most flexible.
- Consumes the capacity of every item in the table

## DynamoDB Consistency Model
- replicated between storage node
- one Leader storage node and every other node follows
- Writes are always directed to the **leader node** and once complete, it is **consistent**
- Once the leader node has the new data it immediatly starts the process of replication
- **Eventual consistent** read might get stale data but get discount for this risk
- A **strongly consistent** read always uses the leader node and is less scalable

## CU calculation
### WCU Calculation
- You need to store 10 items per second with 2.5K average size per item.
- Calculate WCU per item, round up, then multiply by average per second.
  - (2.5 KB / 1 KB) = 3 * 10 p/s = 30 WCU

### RCU Calculation
- Need to retrieve 10 items per second with 2.5K average size per item
- Calculate RCU per item, round up, then multiply by average per second
  - (2.5 KB / 4 KB) = 1 * 10 p/s = 10 RCU for strongly consistent
  - 5 RCU for eventually consistent.

## DynamoDB Indexes
- Alternative view on table data - Indexes

### Local secondary index
- MUST be created with a table (max 5)
- Alternative SK
- Shares RCU and WCU
- Attributes to br projected: ALL, KEYS_ONLY, INCLUDE
- Sparse key: item is present in index only if attribute is present

### Global Secondary index
- created any time (max 20)
- Alternative PK and SK
- Own RCU and WCU units
- Also sparse keys
- ALWAYS eventually consistent

### Considerations
- Careful with projection of attributes
- Queries on attributes NOT projected are expensive
- GSIs as default, LSI only when **strong consistency** is required
- Use for alternative access patterns

## DynamoDB Streams and Triggers
- Time ordered list of changes to items in a DynamoDB table using Kinesis
- 24 hour rolling window of the changes.
- Records Inserts, Updates, Deletes

There are four view types that it can be configured with:
- KEYS_ONLY : only shows the item that was modified
- NEW_IMAGE : shows the final state for that item
- OLD_IMAGE : shows the initial state before the change
- NEW_AND_OLD_IMAGES : shows both before and after the change

Pre or post change state might be empty if you use **insert** or **delete**

### Trigger Concepts
- Actions to take place in the event of a change in data
- Change generates an event which contains the data
- Action is taken using that data using Streams + Lambda


## DynamoDB Global Tables
- multi-master cross-region replication.
- Tables are created in multiple AWS regions. In one of the tables, you configure the links between all of the tables.
- DynamoDB will enable replication between all of the tables
- Between the tables, **last writer wins** in conflict resolution.
- Reads and Writes can occur to any region and are replicated generally within a second
- Strongly Consistent reads **only** in the same region as writes
- **global eventual consistency** but can have same region strongly consistent
- Provides Global HA and disaster recovery easily.

### DynamoDB Accelerator (DAX)

This is an in memory cache for Dynamo.

Traditional Cache : The application needs to access some data and checks
the cache. If there is no data, it is a cache miss and it pulls from
the database. This then updates the cache with the new data. Next times
there will be a cache hit and it will be faster

DAX : The application instance has DAX SDK added on. DAX and dynamoDB are one
in the same. If DAX has the data then the data is returned direclty. If not
it will talk to Dynamo and get the data. This is one set of API calls and
is much easier for the developers.

#### DAX Architecture

This runs from within a VPC and is designed to be deployed to multiple
AZs in that VPC. It must be deployed accross availablity zones to ensure
it is available.

There is a primary node which is a Read and Write node which flows
out to read replicas on other AZs. The DAX SDK communicates with the Read
Write node.

Item cache holds results of batch **GetItem** or batch getitem. You must
specify the items partition or sort key.

There is a query cache which holds data and the parameters used for the
original query or scan. Whole query or scan operations can be rerun
and return the same cache data.

Every DAX cluster has an endpoint which can return data in microseconds.

Any cache miss can be returned in single digit miliseconds.

When writing data to DAX, it can use write-through. Data is written to the
database, then written to DAX.

#### DAX Considerations

Primary node which writes and Replicas which read

Nodes are HA, if this fails there will be an election and secondary nodes
will be made primary.

In-memory cache - scaling. If you are performing the same operations, you
can achieve faster reads and reduced costs.

With DAX you can scale up or scale out.

DAX supports write-through. If you write data to DynamoDB, you can
use the DAX SDK.

DAX is not a public service and is deployed within a VPC. Anything
that uses that data many times will benefit from DAX.

Any questions which talk about caching with DynamoDB, assume it is DAX.

### Amazon Athena

You can take data stored in S3 and perform Ad-hoc queries on data. Pay
only for the data consumed.

This is serverless.

Start off with structured, semi structured data stored in S3.

This uses **schema-on-read**, the original data is never changed
and remains on S3 in its original form.

This modifies data in flight when its read.

Normally you need to make a table and then load the data in.

With Athena you create a schema and load data on this schema on the fly in
a relational style way without changing the data.

The output of a query can be sent to other services and can be
performed in an event driven way.

#### Athena Explained

The source data is stored on S3 and Athena can read from this data.

In Athena you are defining a way to get the original data and defining
how it should show up for what you want to see.

Tables are defined in advance in a data catalog and data is projected
through when read. It allows SQL-like queries on data without transforming
the data itself.

This can be saved in the console or fed to other visualation tools.

You can optimize the original data set.
