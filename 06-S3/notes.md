## S3 Security
- Important: **S3 is private by default!**
- Only the account root user of the account that created the bucket have any permissions for the account.

### S3 Bucket Policy
- A form of **resource policy**.
- Resource policies can allow or deny access from different accounts.
- Resource policies can allow or deny anonymous principals
- Each bucket can only have one policy, but it can have multiple statements.

#### ACL's (Legacy)
- A way to apply a subresource to objects and buckets.
- These are legacy and AWS does not recomment their use.
- They are inflexible and allow simple permissions.

#### Exam Power Up

Identity or Bucket Access:

- Identity
  - Controlling different resources. Not every service supports resource policies.
  - IAM is the only place you can control permissions for everything.
  - Same account

- Bucket
  - Managing permissions on a specific product.
  - If you need anonymous or cross account access.

- ACL's
  - NEVER - unless you must
  - AWS recommends against their use

## S3 Static Hosting
- Normal access is via AWS APIs.
- This allows access via HTTP.
- You can use custom domains - BUCKET NAME MATTERS
- Used for Offloading of Out-of-band pages

### S3 Pricing
- Cost to store data, per GB / month fee
- Data transfer fee
  - Data in is always free
  - Data out is a per GB charge
- Each operation (GET/PUT/POST) has a cost per 1000 operations.

## Object Versioning and MFA Delete
- Versioning is off by default.
- Once it is turned on, it cannot go back.
- It can be suspended but not DISABLED
- **ID of Object** - when versioning is disabled, this is set to **null**
- The latest or current version is always returned when an object version is not requested.
- When an object is deleted, AWS puts a **delete marker** on the object and hides all previous versions. You could delete this marker to enable the item.
- To delete an object, you must delete all the versions of that object using their version marker.
- Space is consumed by ALL versions

### MFA Delete
- Enabled within version configuration in a bucket.
- This means MFA is required to change bucket versioning state.
- MFA is required to delete versions.
- You pass the serial number and the code passed with API calls.

## S3 Performance Optimization

### Single PUT Upload
By default once an object is uploaded to S3, it is sent as a single stream
of data. If a stream fails, the whole upload fails. This requires a full
restart of the data transfer.
- Single PUT upload up to 5GB

### Multipart Upload
- Data is broken up into smaller parts.
- The minimum daa size is 100 MB for multipart.
- **Any file greated than 100MB** benefits frrom multipart
- Upload can be split into maximum of 10,000 parts.
- The last part can be smaller than 5MB as needed.
- Parts can fail in isolation and be restarted in isolation.
- The risk of uploading large amounts of data is reduced.
- Transfer rate = speed of all parts.

### S3 Accelerated Transfer 
- Uses the network of AWS edge locations.
- S3 bucket needs to be enabled for transfer acceleration.
- **Bucketname cannot contain periods and must be DNS compatable in the naming**


## Key Management Service (KMS)
- Regional and a Public Service. Each region is isolated when using KMS.
- It can be connected to by anything with permission in the public zone.
- Create, store, and manage keys. Can handle both symmetric and asymmetric keys.
- KMS can perform cryptographic operations itself.
- Keys never leave KMS.
- Keys use **FIPS 140-2 (L2)** security standard. Some of KMS's features are compliant with Level 3, but all are Level 2.

### KMS Keys
- It is logical and contains
  - ID
  - Date
  - Policy
  - Description
  - State
- These are all backed by **physical** key material.
- You can generate or import the key material.
- KMS Keys can be used for up to **4KB of data**

### Data Encryption Key (DEKs)
- Use GenerateDataKey to create DEK that can be used on data > 4KB
- 
- KMS does not store the DEKs. Once it hands the keys over, it does not care.
- When the DEK is generated, KMS provides two version.
  - Plaintext Version - This can be used immedietly
  - Ciphertext Version - Encrypted version of the DEK. This is encrypted by the KMS Key that generated it.
- The data is encrypted with the plaintext version to generate Cyphertext. After this, the plaintext version is discarded.
- The encrypted data and the encrypted DEK are stored next to each other.
- KMS doesnt do the cryptograhic operations using DEK

### Important Concepts.
- KMS Keys are isolated to a region and never leave
- Two types of KMS Keys: AWS Owned or Customer Owned
- Customer owned has AWS Managed or Customer Managed
- Customer managed keys are more configurable.
- KMS Keys support key rotation.
- KMS Key itself contains the current backing key, physical material used to encrypt and decrypt, as well as previous backing keys.
- You can create an alias. This is also per region.

### Key Policy (resource policy)
- Every KMS Key (CMK) has one.
- Unlike other policies, the key must be told to trust the account.
- In order for IAM to work, IAM is trusted by the account, and the account must be trusted by the key.


## Object Encryption
- Buckets aren't encrypted, **objects are**
- S3 object encryption is focused on encryption at rest. Encryption in transit comes for free with S3

### Server Side Encryption
- SSE is mandatory in S3

Three general types:
#### SSE-C (Server-side encryption with customer provided keys)
- Customer is responsible for the keys themselves.
- Offloads CPU requirements for encryption.
- When placing an object in S3, it requires a key from Client.
- S3 will see the original object throughout the process.
- Once the key and object arrive, it is encrypted. A hash of the key is taken and attached to the object.
- The hash can identify if the specific key was used to encrypt the object.
- The key is then discarded after the hash is taken.

#### SSE-S3 AES256 (Server-side encryption w/ Amazon S3 managed keys)
- When putting data into S3, only need to provide plaintext.
- S3 generates master key. This is fully managed and rotated without your control.
- When an object is added, it generates a key specifically for that one object. It uses that key to encrypt that plaintext object.
- The master key is used to encrypt that one key.
- The original key is discarded.
- The encrypted key is stored next to the encrypted object.
- Very little control how the keys are used.
- Most of situations, this is the default type of encryption
  - Strong algorythm
  - Data encrypted at rest
  - Little admin overhead.
- THREE PROBLEMS:
  - Regulatory enviromment where the keys and access needs to be controlled.
  - No way to control key material rotation.
  - No role seperation.
    - A full S3 admin can rotate keys as well as encrypt or decrypt data.

#### SSE-KMS (Server-side encryption w/ customer master keys stored in AWS KMS)
- Similar as above, except for the master key created in KMS
- Everytime an object is uploaded, S3 uses a dedicated key to encrypt that specific object.
- The key is a data encryption key that KMS generates using the KMS Key.
- S3 is provided with a plaintext version of the data encryption key as well as a plaintext one.
- The plaintext one is used to encrypt the object then it is discarded.
- The encrypted data encrypted key is stored along with the encrypted object.
- you can also add logging and see any calls against this key from cloudtrails.
- The best benefit is the role seperation. To decrypt any object, you need access to the CMK that was used to generate the unique key that was used to generate them.
- The CMK is used to decrypt the data encryption key for that object. That decrypted data encryption key is used to decrypt the object itself.
- If you don't have access to KMS, you don't have access to the object.

### S3 Bucket Key
- Instead of Generating DEK for every object from KMS
- Generate a bucket key that is used for all objects for a time-period
- NOT retroactive
- Cloudtrail KMS events show bucket and NOT object


## S3 Object Storage Classes
- Picking a storage class can be done while uploading a specific object.
The default is S3 standard. Once an object is uploaded to a specific class,
it can be changed as long as the conditions are met

### S3 Standard
- The default AWS storage class that's used in S3. This is pretty good for most cases and should be user default as well.
- S3 is a **region resillent** service which means it can tolerate the failure of an availability zone. This is done by replicating objects to at least 3+ AZs when they are uploaded.
- With S3 standard, it is never less than 3 availability zones
- This has 11 9's for Object Durability and 4 9's for availability.
- Offers low latency and high throughput.
- No **minimums** or **delays** or **penalties**
- First byte latency of milliseconds
- **Data that is frequently accessed, important and non-replaceable**
- All of the other storage classes trade some compromises for another.

### S3 Standard-IA
- Designed for less frequent, but rapid access when it is needed.
- Cheaper rate to store data you will rarely need, but if you do need it, you need it quickly.
- **Durability and Availability is the same as Standard**
- This is approximately 54% cheaper for the base rate.
- No matter what the size of the object, there is a minimum of 128KB min charge.
- The cost benefits might be negated for smaller objects.
- 30 days minimum duration charge per object.
- Also charged a retrival fee for every GB of data retrieved from this class.
- **Long-lived data, important but access in infrequent**

### One Zone-IA
- Designed for data that is accessed less frequently but needed quickly.
- 80% of the base cost of Standard-IA.
- Same minimum size and duration fee and retreival fee
- Data is only stored in a single AZ, no 3+ AZ replication.
- 11 9s of durability unless AZ fails
- **Long-lived Data, non-critical, replaceable and access in infrequent**


### S3 Glacier Instant Retrieval
- Minimum duration charge of 90 days
- Retrievel fees are more expensive
- **long-lived data access once per qtr with millisecond access**

### S3 Glacier Flexible 
- Same infra as before
- 11 9s of durability
- Data in chilled state
- First byte latency of minutes or hours
- 40kb minimum size
- 90 day min duration
- Yearly access of archival data
- Needs to perform a retrieval process
 - Stored in S3 IA for a temp period
#### Expedited
Retrieval between 1 - 5 minutes, most expensive
Provisioned capacity can be bought in advance to make sure it's available when needed
#### Standard
Retrievals take 3 - 5 hours to restore.
This is good for backup data or original media if there was a mistake.
#### Bulk retrievals
Retrievals take 5 - 12 hours.
Lowest cost and is used for large amounts of data.


### S3 Glacier Deep Archive
- Data in frozen state
- 40kb min size
- 180 day min duration
- Designed for long term backups and as a **tape-drive** replacement.
- Standard Retrieval within 12 hours, bulk storage in 48 hours.
- Cannot use to make data public or download normally.

### S3 Intelligent-Tiering
- Has 5 tiers
  - Frequent Access
  - Infrequent Access
  - Archive Instance Access
  - Archive Access
  - Deep Archive
- Monitores usage of objects
- Moved to Infrequent if not access for 30 days
- 90 day min of Archive Instant access
- Last two tiers are optional
- Moved back to grequesnt access if objects are accessed more regularly
- **Long-lived data with changing or unknown patterns**

## Object Lifecycle Management
- Automatically transition or expire objects

### Transition Actions
Change the storage class over time

An example is:

- Move an object from S3 to IA after 90 days
- After 180 days move to Glacier
- After one year move to Deep Archive

Objects must flow downwards, they can't flow in the reverse direction.

One exception: One Zone-IA CAN NOT transition into Glacier - Instant Retrieval

### Expiration Actions

Once an object has been uploaded and changed, you can purge older versions after 90 days to keep costs down.

## S3 Replication

Architecture for both is similar, only difference is if both buckets are
in the same account or different accounts.

An IAM Role is configured during the replciation configuration.

Two types of replication

#### Cross-Region Replication (CRR)

#### Same-Region Replication (SRR)

The role is configured to allow the S3 service to assume it based on
its trust policy. The permission policy allows it to read objects on the
source bucket and copy them to the destination bucket.

When different accounts are used, the role is not by default trusted
by the destination account. If configuring between accounts, you must
add a bucket policy on the destination account to allow the IAM role to
access the bucket.

### S3 Replication Options

- All objects or a subset of those objects
  - Filter can be defined to make the subset.
- Select which storage class the destination bucket will use.
  - The default is the same type of storage, but this can be changed.
- Define the ownership of the objects.
  - The default is the ownership will be the same as the source account.
  - This is a problem if the buckets are in different accounts.
  - The objects in the destination bucket are not owned by that account.
- Replication Time Control (RTC)
  - Adds a guaranteed level of SLA within 15 minutes for extra cost.
  - This is useful for buckets that must be in sync the whole time.

### Important Replication Tips

Replication is not retroactive. If you enable replication on a bucket
that already has objects, the old objects will not be replicated.

Both buckets must have versioning enabled.

It is a one way replication process only.

Replication by default can handle objects that are unencrypted or SSE-S3
With extra configuratiion it can handle SSE-KMS, but KMS requires more
configuration to work.

It cannot use objects with SSE-C because AWS does not have the keys
necessary.

Source bucket owner needs permissions to objects. If you grant cross-account
access to a bucket. It is possible the source bucket account will not own
some of those objects.

Will not replicate system events, glacier, or glacier deep archive.

No deletes are replicated.

#### Why use replication

SRR - Log Aggregation
SRR - Sync production and test accounts
SRR - Resilience with strict sovereignty requirements
CRR - Global resilience improvements
CRR - Latency reduction

## S3 Presigned URL

A way to give another person or application access to a bucket
using your credentials in a safe way.

A S3 bucket without any public access configured.

In order to access the bucket, the IAM user must authenticate and send a
request to access those objects.

iamadmin can make a reqeust to S3 to **generate presigned URL**

The user must provide:

- security credentials
- bucket name
- object key
- expiry date and time
- indicate how the object or bucket will be accessed

S3 will respond with a custom URL with all the details encoded including
the expiration of the URL.

#### Exam Powerup

- You can create a URL for an object you have no access to
  - The object will not allow access because your user does not have it.
- When using the URL the permission match the identity that generated it
at the moment the item is being accessed.
- If you get an access deny it means the ID never had access, or lost it.
- Don't generate with a role. The role will likely expire before the url does.

## S3 Select and Glacier Select
This provides a ways to retrieve parts of objects and not the entire object.

If you retrieve a 5TB object, it takes time and consumes 5TB of data.
Filtering at the client side doesn't reduce this cost.

S3 and Glacier select lets you use SQL-like statement.

The filtering happens at the S3 bucket source 

## S3 Events
- Notification generated when events occur in bucket
- Object created
- Object deleta
- Object Restore (Glacier)
- Replication

## S3 Access Logs 
- Enable access loggin
- Best efforts process
- Uses S3 Log Delivery Group

## S3 Object Lock 
- Enabled on "new" buckets (Support req for existing)
- Write-Once-Read-Many (WORM)
- Requires Versioning

### Retention period
- Specify Days & Years - A retention period
- Compliance - Can't be adjusted, deleted, overwritten EVEN BY ACCOUNT ROOT USER
- Governance - special permissions can be granted allowing lock settings to be adjusted
- s3:BypassGovernanceRetention

### Legal Hold
- No retention period
- ON or OFF
- No Deletes or changes until removed
- s3:PutObjectLegalHold Permission


## S3 Access Points
- Create many access points
- each with different policies
- each with different netowrk access controls
- Console ui: **aws s3control create-access-point** IMPORTANT
