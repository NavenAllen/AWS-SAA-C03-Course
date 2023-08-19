
## Database Refresher
### Relational (SQL)
- Structure in and between tables of data
- There is a rigid **schema**. This is defined in advance
- There is a fixed relationship between tables. This is defined before data is entered into the database
- Every row in a table must have a value for the **primary key**
- There will be multiple tables, but each one needs a primary key
- For every table, there must be a value stored for every attribute in the table
- There is a join table with a **composite key**. The key must be unique in its entirety
- Table relationships use keys
- The Table Scehmas and relationships must be defined in advance.

### Non-Relational (NoSQL)
- Not a single thing, and is a catch all for everything else
- There is generally a weak or no schema.

#### Key-Value databases
- Has a key and the Value shows the data that wants to be recorded
- So long as every key is unique, there is no real schema or structure
- These are really fast and highly scalable
- This is also used for **in memory caching**

#### Wide Column Store
- Each row has one or more keys
  - Parition key - The search key
  - Other Key - sort or range key
- Every item in a table can also have attributes, but they don't have to be the same between items
  - NO ATTRIBUTE SCHEMA
- The only requirements is the key needs to be unique
- It can be **single key** or **composite key**. Either case, it must be unique.

#### Document
- Documents are generally formated using JSON or XML
- This is an extension of a key-value store
- Good for orders or contacts
- This is good for whole documents or deep attribute interations.

#### Column
- Columns are stored together
- Bad for sales style but good for reporting or when all values for a specific size are required.

#### Graph
- Relationships between things are formally defined and stored along in the database itself with the data
- These are great for relationship driven data
- Nodes are objects inside a graph database. They can have properties
- Edges are relationships between the nodes. They have a direction
- Relationships can also have values associated with them and they are also stored inside the database
- Relationships are fast because interactions can be queried

## ACID vs BASE
- DB transaction models
- CAP Theorem - Consistency, Availability, Partition Tolerant (resilience) - Choose 2
- ACID - Consitency
- BASE - Availability

### ACID
- Atomic
  - ALL or NO components of a transaction success of ails
- Consistent
  - Transaction move databse from one valid state to another
- Isolated
  - Transaction dont interfere with each other
- Durable
  - Once commited, transaction are durable and not lost
 
Usually mentions RDS systems

### BASE
- Basically Available
  - READ and WRITE are available but without any consistency guarantes
- Soft state
  - DB doesnt enforce consistency, application needs to make sure
- Eventually consistent
  - If we wait long enough, reads are consistent

Usually means NoSQL databases



## Databases on EC2
- It is always a bad idea to do this
-  Splitting an instance over different AZs adds a small cost to moving the data between AZs if it happens.

### Reasons EC2 Database might make sense
- You might need access to the OS of the Database
  - You should question if a client requests this, it rarely is needed.
- Advanced DB Option tuning (DBROOT) AWS provides options against this
- This is typically a vendor that demands this
- DB or DB version that AWS doesn't provide
- Architecture AWS don't provide

### Reasons why you really shouldn't run a database on EC2
- **Admin overhead** is intense to manage EC2 and DBHost compatible
- Backup and Disaster Management adds complexity
- EC2 is running in a single AZ. If the zone fails, access to the database fails
- Will miss out on features from AWS DB products
- EC2 is ON or OFF, there is no way to scale easily
- **Replication** the skills and setup time to monitor this
- Performance will be slower than AWS options.

## Relational Database Service (RDS)
- Database-as-a-service (DBaaS)
  - not entirely true more of DatabaseServer-as-a-service. Managed Database Instance for one or more databases
- Multiple engines are MySQL, MariaDB, PostgreSQL, Oracle, Microsoft SQL
- Amazon Aurora. This is so different from normal RDS, it is a seperate product
- Managed service.. no access to OS or SSH Access* (RDS custom allows this)

### RDS Architecture
- Runs within an VPC
- Has a RDS Subnet Group
- Can be deployed in Private Subnet (only accessible within VPC)
- Public subnet (can be configured to access from Public)
- Each instance can have multiple DBs
- Had a dedicated Storeage per instance

### RDS - Costs
- Cost #1 - Instance Size & Type
- Cost #2 - Multi AZ or not
- Cost #3 - Storage Type and amount
- Cost $4 - Data transferred
- Cost $5 - Backups & Snapshots
- Cost #6 - Licensing (if applicable)


## RDS Multi AZ (High-Availability)
### Multi AZ - Instance
- Has a primary and standby instance
- **Synchronous Replication**
- RDS Access ONLY via database CNAME
  - The CNAME will point at the primary instance. You cannot access the standby replica for any reason via RDS.
- Backups occur from Standby to avoid performance loss on primary
- If primary goes down, RDS points to Standby instances
  - Has a bit of an outage for 60-120 seconds
  - This does not provide fault tolerance - there will be some impact during change
- ONE Standby replica ONLY
- Same region ONLY

### Multi AZ - Cluster
- Has one writer and TWO reader instances
  - TWO READERS ONLY
- Synchronous Replication
- Reader instances can be used for Reading and READING ONLY
- Data is "committed" when 1+ reader finished writing
- Has different endpoints
  - Cluster Endpoint popints at writer
  - Reader Endpoint directs ant reads to an available reader instance
  - Instance endpoints points at specific instance
 
#### Exam Powerups
- 1 writer and 2 Reader DB instances (Different AZs)
- Much faster hardware
- Fast writes to local storage -> flushed to EBS
- Allows read scaling
- Replication is via transaction logs
- Failover is faster ~35s + transaction log apply


## RDS Backup and Restores
- Have **Automated Backups** and **Snapshots**
- AWS Managed S3 Buckets - cant be seen in S3 console

### Snaphots
- First snap is FULL size of consumed data, incremental afterward
- Manual snapshots will remain in your AWS account even after the life of the snapshot. These need to be deleted manually.

### Automated Backups
- Occurs once per day, kind of like automated snapshots
- In Addition, every 5 minutes translation logs are saved to S3. A database can then be restored to a 5 min snapshot in time
- Automatic cleanups can be anywhere from 0 to 35 days. This will use both the snapshots and the translation logs
- When you delete the database, they can be retained but they will expire based on their retention period.

### RDS Cross-Region backups
- Can replicate backups to another region
- Both snapshots and transaction logs
- Charges apply for data copy and storage in another region
- NOT DEFAULT

### RDS - Restores
- When performing a restore, RDS creates a new RDS with a new endpoint address
- When restoring a manual snapshot, you are setting it to a single point in time. This influences the RPO value
- Automated backups are different, any 5 minute point in time.
 - Backups are restored and transaction logs are replayed to bring DB to desired point in time
= Restores aren't fast, think about RTO.

## RDS Read-Replicas
- Kept in sync using **asyncronous replication**
- It is written fully to the primary instance. Once its stored on disk, it is then pushed to the replica
  - This means there could be a small lag. 
- These can be created in the same region or a different region. This is a **cross region replication**
- NO automatic failover
- Read replicats are separate instances, application has to be aware of them

### Why do these matter
#### READ performance
- 5 direct read-replicas per DB instance
- Each of these provides an additional instance of read performance
- This allows you to scale out read operations for an instance
- Read-replicas can chain, but lag will become a problem
- Can provide global performance improvements

#### Availability Improvements
- Snapshots and backups improve RPO
- These don't help RTOs
- These offer near 0 RPO
- If the primary instance fails, you can promote a read-replica to take over
- Once it is promoted, it allows for read and write
- Only works for failures, these can replicate data corruption, default back to snapshots and backups
- Promotion cannot be reversed
- Global availability improvements provides global resilience

## RDS Security
- SSL/TLS is available for RDS, can be mandatory
- RDS supports EBS volume encryption - KMS
- Same as EBS encryption
- Storage, Logs, Snapshots and replicas are encrypted
- Encryption is not removable
- RDS MSSQL and RDS Oravle support Transparent Data Encryption
  - Enecryption handled within the DB engine
- RDS Oracle supports TDE with CloudHSM

### RDS IAM Authentication
- RDS can be configured to use IAM user authentication to work with Local DB authentication
- Policy attached to Users or Roles maps that IAM identity onto the local RDS user
- generate-db-auth-token: creates a token with 15 min validity which can be used in place of DB User password
- NOT used for authorisation, ONLY authentication

## RDS Custom
- Fills the gap between RDS and EC2 running a DB engine
- Currently works for MS SQL and Oracle
- Can connect using SSH, RDP, Session Manager
- You can see EC2 instances, EBS and backups in account
- Database automation
  - pause to customize
  - Resume after done


## Amazon Aurora
- Aurora architecture is VERY different from RDS
- At it's heart it uses a **cluster**
- A single primary instance and 0 or more replicas
- The replicas within Aurora can be used for reads during normal operation
  - Provides benefits of RDS multi-AZ and read-replicas
- Aurora doesn't use local storage for the compute instances
  - An Aurora cluster has a shared cluster volume. Provides faster provisioning.

### Architecture
- Aurora cluster functions across different availability zones
- There is a primary instance and a number of replicas
- The read applications from applications can use the replicas
- There is a shared storage of **max 64 TiB, 6 Replicas, AZs**
  - Synchronous Replication
- All instances have access to all of these storage nodes
- This replication happens at the storage level
- By default the primary instance is the only one who can write
- Aurora automatically detect hardware failures on the shared storage
  - If there is a failure, it immedietly repairs that area of disk. It automatically recreates that data with no corruption.
- With Aurora you can have up to 15 replicas and any of them can be a failover target
  - The failover operation will be quicker because it doesn't have to make any storage modifications. 
- Cluster shared volume is based on SSD storage by default so high IOPS and low latency.
- Aurora cluster does not specify the amount of storage needed. This is based on what is consumed.
- IMPORTANT **High water mark** - billed for the most used. Storage which is freed up can be re-used.
- Replicas can be added and removed without requiring storage provisioning.

### Endpoints
- Cluster endpoint - points at the primary instance
- Reader endpoint - will load balance over the available replicas
- Custom endpoints can also be added
- As additional replicas are used for reads, this is load balanced over replicas.

### Costs
- No free-tier option
- Aurora doesn't support micro instances
- Beyond RDS singleAZ (micro) Aurora provides best value.
- Compute - hourly charge, per second, 10 minute minimum
- Storage - GB-Month consumed, IO cost per request (High Watermark)
- 100% DB size in backups are included

### Aurora Restore, Clone and Backtrack
- Backups in Aurora work in the same way as RDS
- Restores create a new cluster.
- Backtrack allows for you to roll back to a previous point in time
  - You can roll back in place to a point before that corruption
  - nabled on a per cluster basis and can adjust the window backtrack can perform.
- Fast clones make a new database much faster than copying all the data
  - It references the original storage and only write the differences between those two
  - It only copies the difference and only store changes between the source data and the clone

## Aurora Serverless
- Provides a version of Aurora without worrying about the resources
- It uses ACU - Aurora Capacity Units
- For a cluster, you can set a min and max ACU based on the load
- Can go to 0 and be paused
- Consumption billing per-second basis
- Same resilience as Aurora (6 copies across AZs)
- ACUs are stateless and shared across many AWS customers and have no local storage
- They have access to cluster storage in the same way
- ACUs can scale in and out based on usage
- There is a shared proxy fleet.
  - When a customer interacts with the data they are actually communicating with the proxy fleet
  - The proxy fleet brokers an application with the ACU and ensures you can scale in and out without worrying about usage.

### Aurora Serverless - Use Cases
- Infrequently used applications
- New applications
- Great for variable workloads. It can scale in and out based on demand
- It is good for applications with unpredictable workloads
- It can be used for development and test databases because it can scale back when not needed
- Great for multi-tenant applications. If your incoming load is directly tied to more people, that's fine.

## Aurora Global Database
- Replication from primary cluster volume to secondary replicas for read operations
- Great for cross region disaster recovery and buisness continuity
- Global read scaling - low latency performance improvements for international customers
- The application can perform read operations against the read database
- There is ~1s or less replication between regions. It is one way replication
- No additional CPU usage is needed, it happens on the storage layer
- Secondary regions can have 16 replicas
- All can be promoted to Read or Write with diasters
- There is currently max of 5 secondary regions.

## Aurora Multi-Master Writes
- Allows an aurora cluster to have multiple instances with multiple writers
- The default aurora mode is single-master
  - This is one R/W and zero or more read only replicas.
  - Cluster endpoint is normally used to write, read endpoint is used for load balancing
- In Multi-Master cluster, all instances are R/W capable
- Aurora Multi-master has no endpoint or load balancing
- When one of the R/W nodes, it proposes all data be commited to all storage of the clusters They each confirm or deny if this change is allowed
- The writing instance is looking for a bunch of nodes to agree
  - If the group rejects it, it cancels the write in error. If it commits, it will replicate on all storage nodes.
- In a Multi-master cluster, it will then copy into other masters.
  - This ensures storage is updated on in-memory cache's
- If a writer goes down in a multi-master cluster, the application will shift all future load over to the new writter with little to no downtime.

## RDS Proxy
- Provides a pool of DB connections that applications can use
- Instead of connecting to database directly, applications connect to proxy and it provides them with a connection to DB
- Maintains a long term connection pool
- Much quicker to establish
- Connections can be reused
- Multiplexing is used, smalled number of DB connections, larger number of application connections
- Client waits for once Proxy conenction is established if Target DB is unresponsive

### When to use?
- Too many connections errors
- DB instances using T2/T3
- AWS Lambda
- Long running connections - low latency
- Resilience to DB failure
- Reduces time for failover and makes it transparent to application

### Key Facts
- Fully Managed DB Proxy for RDS/Aurora
- Auto scaling, HA by default
- Only accessible from VPC
- Accessed via Proxy Endpoint - no app changes
- Can enforce SSL/TLS
- Can reduce failover time by over 60%
- Abstracts failure away from apps


## Database Migration Service (DMS)
- A managed database migration service
- This runs using a replication instance
- Need to define the source and destination endpoints. These point at the physical source and target databases
- One endpoint MUST be on AWS.

### Architecture
- DMS uses a replication instance
- Instance needs to have defined replication tasks
- Task moves data from source to destination
- Jobs can be of three types
  - Full load - one off migration of all data
  - Full Load + CDC - migrates all data and also captures data changes
  - CDC Only - replicate data changes only 

### Schema Converstion Tool:
- Used when migrating between different engines
  - DB -> S3
- NOT USED when migrating between DBs of same type
- Works with OLTP DB Types (MySQL, MSSQL, Oracle)
- Works with OLAP (Terradata, Oracle, vertica, Gleenplum)

### DMS and Snowball
- Larger migrations - multi-TB in size
- Step 1: Use SCT to extract data and move to snowball
- Step 2: Ship the device back to AWS. They load onto an S3 bucket
- Step 3: DMS migrates from S3 into the target store
- Step 4: CDC can capture changes
- Uses STC as Data is transferred to a Physical file storage 
