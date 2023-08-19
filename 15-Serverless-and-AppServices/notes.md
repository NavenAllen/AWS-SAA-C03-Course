## Architecture Evolution
### Monolithic

- It fails together. One error will bring the whole system down.
- Scales together. Everything expects to be running on the same compute hardware
- Bill together. All components are always running and always incurring charges.

This is the least cost effective way to architect systems.

### Tiered
- The different components can be on the same server or different servers
- The components are coupled together because the endpoints connect together
- You can increase the size of the server that is running each application tier
- We can utilize load balancers in between tiers to add capacity
- The tiers are still coupled, the upload tier **expects** and requires processing to respond
  - If the Processing fails completely, the upload tier will fail because it does not recieve a response
- If there is a backload in one tier, it will impact the other tiers and the customer experience
- Even if there is no job to be processed, the middle tier will need to be running because otherwise it would fail.

### Evolving with Queues
- This is a system that accepts messages off a queue. Often times they are **FIFO** (first in, first out)
- Instead of passing data into the processing tier
  - It will store this in an S3 bucket as well as detailing the information into the queue. This now moves towards the next slot in the queue
- The upload tier doesn't expect an immediate answer from the processing tier
- It sends an asycn message. While this is happening, the upload can add more messages to the queue
- The queue will have an autoscaling group to increase capacity at the end and process appropriately
- The queue has the location of the S3 bucket as well as the location of the information
- The autoscaling group will only bring up servers as their needed


### Event Driven Architecture
- Event producers - interact with customers or systems monitoring components They produce events in reaction to something.
- Event consumers - pieces of software waiting for events to occur
- Services can be producers and consumers at once
- In either case, there are no resources waiting around to be used
- Event router is needed for event driven architecture that also manages an event bus.

#### Highlights
- No constant running or waiting for things
- Producer generate events when something happens
- Clicks, events, errors, actions
- Events are delivered to consumers of events
- Actions are taken and the system returns to waiting
- Mature event driven architecture only consumes resources while handling events.

## AWS Lambda
- Function-as-a-service (FaaS)
- Event driven invocation (execution)
- **Lambda function** piece of code in one language
- Lambda functions use a **runtime** (e.g. Python 3.6)
- Runs in a **runtime environment**
- You are billed only for the duration a function runs. There is no charge for having lambda functions waiting and ready to go
- Custom runtimes such as Rust are possible using layers
- Directly control the memory allocated for functions whereas vCPU is allocated indirectly
  - 128 -> 10240MB in 1MB steps
- 512MB storage available as /tmp
- Maximum of 15 minutes - 900s -IMPORTANT

### Common uses of Lambda
- Serverless application
- File processing
- Database triggers
- Serverless CRON
- Realtime stream data processing

### Public Lambda
- Lamda is public by default and can access public AWS services (SQS, DynamoDB)
- Offers best performance
- By default, public lambdas dont have access to VPCs unless they have public IPv4

### Private Lambda
- Obey all VPC networking rule
- Needs ec2 network permissions
- ENIs are now created for each unique combination of subnets and SGs 
  - If all subnets use same SG, only one ENI

### Security
- Lambda needs execution role to decide which AWS services it can access
- Also has resource policy that decide who and what can access it
  - Kind of like bucket policy
 
### Logging
- Lambda uses CW, CW Logs and x-ray
- Logs from Lambda executions - Cloudwatch Logs
- Metrics - CW
- Lambda can be integrated with X-Ray for distributed tracing
- CW logs requires permissions via execution roles


### Invocation
#### Synchronous
- CLI/API invokes a function and WAITS for a response
- Errors or Retries have to be handled within the client

#### Asynchronous
- Typically used when AWS services invoke functions
- Lambda is responsible for retries
  - function needs to be idempotent (reprocessing should have same end state)
- Events sent to Dead Letter Queues if failed after repeated processing
- Sent to destinations if successful or failed too

#### Event Source Mapping
- Typically used on streams or Queues which which dont support event generation
- Lambda polls the resources for any jobs
- Event Source Mapping needs permission on the resource - IMPORTANT

### Versions
- Lambda supports versions
- A version is code + config
- Each version is immutable - has own ARN
- $Latest points at the latest version - immutable
- ALIASES point at a version - can be changed

### Launch
- Cold Start - full creation
- Warm Start - same execution context is reused if another event happens soon (infrequent contexts are removed)
- Provisioned concurrency can be used
  - AWS will create and keep X contexts warm


## CloudWatch Events and EventBridge
- Delivers near real time stream of system events that describe changes in AWS products and services. EventsBridge will replace CW Events
- EventsBridge can also handle events from third parties. Both share the same underlying architecture. AWS is now encouraging a migration to EB.

### Key Concepts
- They can observe if X happens at Y time(s), do Z. This is basically CW Events V2
- Both systems have a default Event bus for a particular AWS account.
- In CW Events, there is only one bus (implicit), this is not exposed
- EventBridge can have additional event buses. These can be interacted with in the same way as the default bus
- Rules match incoming events or schedules. The rule matches an event and routes that event to one or more targets as you define on that rule
- Architecturally at the heart of event bridge is the default event bus
- The default event bus is moving packets of JSON data.


## Serverless
- This is not one single thing, you manage few if any servers
- Applications are a collection of small and specialized functions
- These functions are stateless and run in ephemeral environments
  - Each time they run, they obtain the data they need each time
- Generally, everything is event driven.
- FaaS is used whenever possible for compute
- Should use managed services when possible.



## Simple Notification Service (SNS)
- HA, Durable, and Secure service
- This is a public service which needs access to the public endpoint
- Coordinates the sending and delivery of messages
- Messages are under 256KB in size.
- SNS topics are the base entity of SNS
- A publisher sends messages to a topic
- Topics have subscribers which recieve messages
- You can create topics inside the SNS
- By default all topics will recieve the message
  - **you can put filters on those lines to make sure they don't trigger additional lambdas.**
- You can use fanout to process different flows from SQS

### Functionality Offered:
- Delivery Status including HTTP, Lambda, SQS
- Delivery retries - Reliable Delivery
- HA and Scalable (Regional)
- SSE (server side encryption)
- Topics can be used cross-account via Topic Policy

## AWS Step Functions
- There are many problems with lambdas limitations that can be solved with a state machine.
- This is a serverless workflow
  - Start
  - States
  - End
- States are **things** which occur
- Maximum duration is 1 year
- Standard workflow and express
  - At a high level, standard is the default and has a 1 year workflow
  - Express is for IOT and highly transactional such as IoT - 5 minutes
- Started via API Gateway, IOT Rules, EventBridge, Lambda.
- Amazon States Languate (ASL) - JSON template
- These use IAM Roles for permissions.

### States
- Succeed & Fail : Will wait until either is achieved
- Wait : will wait until specific date and time or period of time
- Choice : different path is determined based on an input
- Parallel : will create parallel branches based on a choice
- Map : accepts a list of things
- Task : Single unit of work (lambda, batch, dynamoDB)

## API Gateway
- Create and manages APIs
- Endpoint/entry-point for applications
- HA, scalable, handles authorisation, throttling, caching, CORS, transformations, OpenAPI spce, direct integration
- Can connect to services/endpoints in AWS or on-premises
- HTTP APIs, REST APIs and WebSocket APIs
- Integrates with CW to storate and mange request and response logs and metrics
- Has a cache that can be used to reduce the number of calls

### Authentication
- Cognito
- Lambda based Authorization

### Endpoint Types
- Edge-Optimized
  - Routed to the nearest CloudFront Point of Presenve
- Regional
  - Clients in the same region
- Private
  - Acessible within VPC using Interface endpoint
 
### Stages
- APIs are deployed to stages
- Each stage has one deployment
- Stages can be enabled for canary deployments
  - COnfigured so a certain percentage of traffic is sent to canary
 
### Errors
- Common HTTP status code errors
- 429 - API Gateway throttling
- 504 - INtegration failure/timeout - 29s limit

### Caching
- Can be 500MB to 237GB
- TTL default is 300s
  - Configurable min 0 and max 360s
- **CAN BE ENCRYPTED**
- Defined per stage within API Gateway

## Simple Queue Service (SQS)
- Provides managed message queues, fully managed, highly available.
- Public Service
- Replication happens within a region by default.
- Messages up to 256KB in size - link to larger sets of data.
- When a client polls and recieves messages, they are hidden due to **visibility timeout**.
  - If a client recieves messages on the queue and finishes on that workload it can delete the message
  - If the client doesn't delete the message, then it will reappear on the queue
  - The queue will put the message back in and make sure a different client can retry that workload.
- **Dead-letter queue** if a message is recieved multiple times but is unable to be finished
  - this puts it into a different workload to try and fix the corruption
- ASG can scale and lambdas can be invoked based on queue length

### SNS and SQS Fanout
- Message added to SNS topic
- Multiple SQS queues subscribe to topic

### Highlights
- Standard - multi-lane HW guarantee the order and at least once delivery.
- FIFO - single lane road with no way to overtake guarantee the order and at exactly once delivery
  - 3,000 messages per second with batching or up to 300 messages second without
  - Messages need to have **.fifo** to be valid (very important)
- Billed on **requests** not messages. A request is a single request to SQS
  - One request can send 1 - 10 messages up to 64KB total.
- Requests can return 0 messages. The more frequently you poll a SQS Queue, the less effective it is.
- Two ways to poll
  - short (immediate) : uses 1 request and can return 0 or more messages. If the queue is empty, it will return 0 and try again. This hurts queues that stay short  
  - long (waitTimeSeconds) : it will wait for up to 20 seconds for message to arrive on the queue. It will sit and wait if none currently exist.
- They offer KMS encryption at rest
- Messages can live on SQS Queue for up to 15 days
- Access is based on identity policies or a queue policy

### SQS Delay Queues
- DelaySeconds - messages start invisible for this time
- Default is 0, max is 15 mins
- Message timers allow a per-message invisibility
  - NOT supported on FIFO queues
 
### SQS Dead-letter Queues
- Enqueue timestamp of message remains unchanged
- Retention period of this queue should be longer than source queues

## Kinesis
- This is a scalable streaming service
- **Real-time service**
- It is designed to inject data from lots of devices or lots of applications
- Producers send data into a Kinesis Stream
- The stream can scale from low to near infinite data rates
- Highly available public service by design
- Streams store a 24-hour moving window of data. Can be increased to 365 days.
  - Data that is 24 hours and a second more is replaced by new data entering the stream.
- Kinesis includes the store within it for the amount of data that can be ingested during a 24 hour period
- Multiple consumers can access data from that moving window
- Uses shard architecture
 - Each shard can have 1MB/s for ingestion and 2MB/s consumption.
- **Kinesis data records (1MB)** are stored accross shards and are the blocks of data for a stream.

### SQS vs Kinesis
- Is this about the ingestion of data or is it about the worker pools
- Large throughput or large numbers of devices, it is likely Kinesis
- SQS has 1 thing sending messages to the queue. One consumption group from that tier.
  - Allow for async communications where the sender and reciever don't care about what the other is doing
  -  Once the message is processed, it is deleted.
- Kinesis is desiged for huge scale ingestion with multiple consumers
  - Rolling window for multiple consumers
  - Designed for data ingestion, analytics, monitoring, app clicks.

## Kinesis Firehose
- connects to a Kinesis stream
- It can move the data from a stream onto S3 or another service.
- Automatic Scaling
- Near Real Time
- Supports transformation
- Valid Destinations:
    - HTTP
    - Splunk
    - Redshift
    - ElasticSearch
    - S3
- Can directly accept data without streams
- Near Real Time: waits for 1mb of data for 60 seconds on buffer
- Delivery to S3 alone using S3 as an intermediate

## Kinesis Data Analytics
- Real Time processing using SQL
- Ingests from Kinesis Data Streams or Firehose
- Destinations
    - Firehose destinations (Only near real time)
    - AWS Lambda
    - Kinesis Data Streams
- A reference table is used to enrich the streaming input
- Application code processes Input and produces enriched output using SQL

### When and where
- Streaming data needing real-time SQL processing
- Time-series analytics - elections / e-sports
- Real-time dashboards - leaderboards for games
- Real-time metrics - Security and response teams

## Kinesis Video Streams
- Ingest Live video from producers
- Can access data frame-by-frame or as needed
- Can persist and encrypt (in-transit and at rest)
- **can't access directly via storage.. only via APIs**
- Integrates with other AWS services

 ## Cognito
 ### User Pools
 - Swaps external identity for JWT only
 - cannot be used to access AWS resources

### Identity Pools
- Swaps identity for AWS creds
- Can use User Pool tool to return AWS creds

## AWS Glue
- Serverless ETL (Extract, Transform & Load)
- Uses EMR servers
- Moves and transforms data between source and destinations
- Crawls data sources and generated Data catalog
- Glue is serverless and ad-hoc compared to Data Pipeline

### Data Catalog
- Persistent metadata about data sources in region
- One catalog per region per account
- Avoids data silos
- Crawlers connect to data stores to create metadata and store it in data catalog
- Glue jobs extract data and load it in targets
- Only charged for resources used by Glue Jobs 

## Amazon MQ
- Open Source Message broker
- Based on Apache ActiveMQ
- JMS API - protocols such as AMQP, MQTT, OpenWire and STOMP
- VPC Bases
- No AWS native integration

### When to use Amazong MQ
- SNS or SQS for new implementations
- SNS or SQS if AWS integration is required
- Amazon MQ if migration without much application change
- Amazon MQ if APIs just as JMS or protocols such as AMQP, OpenWire, STOMP or MQTT
- NEED PRIVATE NETWORKING FOR AMAZON MQ

## Amazon AppFlow
- Integration service
- Exchange data between applications using flows
- Sync or aggregate from different sources
- Public Endpoints, but works with PrivateLink
