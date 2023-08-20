## Border-Gateway Protocol
- Routing protocal
- Autonomous systems - Routers controlled by one entity (Black Box)
- ASN are unique and allocated by IANA
- BGP operates over tcp/179
- Not automatic - peering is manually configured
- Path-vector protocol - exchanges best path to a destination - ASPATH
- iBGP - Internal, eBGP - between As's
- AS Path Prepending can be used to make a path look longer than it actually is

## IPSec VPN
- Setup secure tunnels across insecure networks
  - between two peets (local and remote)
- Provide authentication
- Encryption
- IPSec has two main phases

### IKE Phase 1 (Slow & Heavy)
- Authenticate - Pre-shared key
- Using Asymmetric encryption to agree on and create a shared symmetric key
- IKE Security Association (SA) Created (phase 1 tunnel)

### IKE Phase 2
- Use the keys agreed in phase 1
- Agree encryption method, and keys used for bulk data transfer
- Create IPSEC SA phase 2 tunnel

### Fundamentals
- Policy based VPNs
  - rule sets match traffic => a pair of SAs
  - Different rules/security settings
  - Each Policy has a different SA Pair and Unique IPSEC Key
- Route Based VPNs
  - target matching (prefix)
  - matches a single pair of SAs

## AWS Site-to-Site VPN
- A logical connection between a VPC and on-premise network encrypted using IPSec, running over the public internet.
- This can be full High Availability if you design it correctly
- Quick to provision, less than an hour.
- VPNs connect VPCs and on-prem
- Virtual Private Gateway (VGW) is the target on one or more route tables
  - Has two endpoints
  - HA by default
- Customer Gateway (CGW)
  - logical configuration on AWS
  - also the thing that configuration mentions
  - Might be a single point of failure
- VPN connection itself stores the config and connects the VGW and CGW
- Two make it fully HA, add another customer gateways
  - This adds two more VGW endpoints
- Can be static and Dynamic VPN
  - Static VPN works with static IP addresses
  - No Load balancing and multi connection failover
  - For Dynamic VPN, BGP is configured
  - Route Propagation when enabled propagates the routes from VGW to VPC router

### Considerations
- Speed Cap on VPN 1.25Gbps
- Latency Considerations - this is inconsistent because it uses the public internet.
- Cost - AWS hourly, GB out cost, data cap
- Setup of hours or less
- Great as a backup especially for Direct Connect (DX)




## AWS Direct Connect (DX)
- This is a 1 Gpbs or 10 or 100 Gbps Network Port into AWS
- This is at a DX Location (1000-Base-LX or 10GBASE-LR)
- This is a cross connect to your customer router (requires VLANS/BGP)
- You can connect to a partner router if extending to your location.
- The port needs to be arranged to connect somewhere else and connect to your hardware.
- This is a single fiber optic cable from the DX port to your network.
- VIFS are multiple virtual interfaces (VIFS) over one DX
  - Private VIF (VPC)
  - Public VIF (Public Zone Services)
- Has one physical cable with no high availability and no encryption.
- Can take weeks or month for physica cable to be installed
- Cannot be used to access public internet unless proxy is added
- Public VIF vs Private VIF
  - **Public VIF** is only public services, not public internet
  - **Private VIF** is attached to a  VPC
- DX Port Provisioning is likely quick, the cross-connect takes longer.
- Generally use a VPN first then bring a DX in and leave VPN as backup.
- It does not use public internet and provides consistently low latency.
- DX provides NO ENCRYPTION and needs to be managed on a per application basis.




### Public VIF + VPN over DX
- We can use a Public VIF + VPN over IPSec to provide encryption
- VPN Site2Site is also a backup
- We use Public VIF cause we connecto to Public IPs (gateways)

## AWS Transit Gateway (TGW)
- Network transit hub to connect VPCs to on premises networks
- Significantly reduces network complexity
- There is a single network object which makes it HA and scalable
- Attachment to other network types
- VPC attachments are configured with a subnet in each AZ where service is required
- You can use these for cross-region peering attachment
- Can share between accounts using AWS RAM

#### Key points
- Supports transitive routing
- Supports peering even with other transit gateways (can create global networks)
- Share accounts with AWS RAM
- Peer with different regions.. same or cross account



## Storage Gateway
- Hybrid Storage Virtual Application (On-premise)
- Virtual machine (or hardware appliance in rare cases)
- Presents storage using iSCSI, NFS or SMB
- Integrates with EBS, S3 and Glacier
- Migrations, Extensions, Storage Tiering

Scenarios

Extend storange of File and Volume Storage into AWS.
Keep volume storage backups into AWS.
Tape backups into AWS. Can act as emulation layer.

Migration of extisting infrastructure into AWS slowly.

### Tape Gateway (VTL) Mode
- uses iSCSI to connecto to Media Changer and Tape Drive
- Has a Upload Buffer and Local Cache
- Virtual Tapes are stored on S3
- Tape Library is S3 and Tape Shelf is Glacier 

### File Mode : SMB and NFS
- File Storage Backed by S3 Objects
- Mount points map directly onto an S3 bucket
- Primary data is stored in S3
- Read and Write caching ensure LAN-like performance
- Can be used to implement multi-site architecture, same object in S3 can be mapped to multiple sites
  - New files don't show up unless listed
  - Use the NotifyWhenUploaded API
- File Gateway doesn't support Object Lock
- Can do replications to different regions
- Can integrate with S3 Lifecycle management

### Volume Mode 
- Raw Volume Blocks are provided via iSCSI

#### Gateway Stored (Used for backup, migration)
  - Primary data storage is On Premises
  - Block Storage backed by S3 and EBS
  - Great for disaster recovery
  - Data is kept locally
  - Awesome for migrations
  - Has a upload buffer which is used to store into S3 asynchronously using a Sotrage Gateway Endpoint
  - Not good for storage extensions

#### Cache Mode (Used for Storage extensions)
  - Primary data is stored in S3 (AWS Managed, only available through Storage Gateway console)
  - Data as added to gateway is not stored locally, only frequently accessed data is saved in local cache
  - Backup to EBS Snapshots
  - Primarily stored on AWS
  - Great for limited local storage capacity.





## Snowball / Edge / Snowmobile
- Move large amounts of data IN and OUT of AWS
- Physical storage the size of a suitcase or truck.
- Ordered from AWS, use, then return.

### Snowball

Anything on Snowball uses KMS
50TB or 80TB Capacity
1 Gbps or 10 Gbps
This makes sense from 10 TB to 10 PB (multiple devices) and over many premises.
This only includes storage
**Multiple devices to multiple premises**

### Snowball Edge

Both storage and compute
Larger capacity vs snowball
10 Gbps or up to 100 Gbps

Storage optimized (with EC2) includes 1TB SSD
Compute optimized
Compute with GPU as above with GPU

**These are great for remote sites when data processing on ingestion is needed**

### Snowmobile

Portable data center within a shipping container on a truck.

This is a special order and is not available in high volume.
Ideal for single location where 10 PB+ is required.

Up to 100 PB per snowmobile.

**This is not economical for multi-site for sub 10 PB**




## AWS Directory Service
- This is a managed service with lots of use cases
- Stores objects, users, groups, computers, servers, file Shares with a structure.
- Multiple trees can be grouped into a forest
- Commonly used in Windows Environments
- Sign in to multiple devices with the same username/password provides central management for assets.

### AWS managed implementation
- Runs within a VPC as a private service.
- Provides HA by deploying into multiple AZs.
- Certain services in AWS need a directory, Amazon Workspaces.
- To join EC2 instances to a domain you need a directory.
- Can be isolated or integrated with existing on-prem system.
- Could act as a proxy back to on-premises.

### Picking the Modes
- Simple AD should be default
  - uses SAMBA
  - cannot connect with on-prem AD
  - Up to 500 Users (Small) or 5000 Users (Large)
- Microsoft AD is anything with Windows
  - Primary running location is AWS
  - or if it needs a trust relationship with on-prem
  - This is not an emulation
  - Actual implmentation of Microsoft AD
- AD Connector - Use AWS services without storing any directory info in the cloud, it proxies to your on-prem directory.




## AWS DataSync
- Data transfer service TO and FROM AWS
- This is used for migrations or for large amounts of data processing transfers
- Designed to work at huge scales. Each agent can handle 10 GB and each job can handle 50 million files.
- This keeps metadata
- Has built in data validation to ensure the data matches

### Key Features
- Each agent is about 100 TB per day
- Can use bandwidth limiters to avoid customer impact
- Has incrememetal and scheduled transfer options
- Compression and encryption is also supported
- It does data validation and automatic recovery from transit errors
- AWS service integration with S3, EFS, FSx for Windows servers
- Pay as you use product
- The data is encrypted in transit and all of the data transfer in parts.

### Components
- Datasync agent needs to be installed on premises
- Bidirectional transfer
- Task is a job within datasync and defines what is going from where to where
- Agent is software to read and write to on prem such as NFS or SMB
- Location is the FROM and TO
- DataSync endpoint can store into S3 or EFS or FSx or NFS or SMB
- WHEN TO USE DATASYNC - **Transfers can be scheduled, automatic retries, compression, bandwith throttle, traditional file transfer protocols**




## FSx for Windows File Server
- Fully managed native windows file servers/shares
- Designed for integration with windows environments
- Integrates with Directory Service or Self-Managed AD
- Single or Multi-AZ within a VPC
- Can perform on-demand and scheduled backups
- File systems can be access using VPC, Peering, VPN, Direct Connect. Native windows filesystem or Directory Services.

### Words to look for
- VSS - User Driven Restores
- Native file system accesible over SMB
- Supports shadow copies (file level versioning)
- Windows permissions model
- Product supports DFS (Distributed File Share), scale out file share
- Managed - no file server admin
- Integrates with DS and your own directory.




## FSx for Lustre
- Managed Lustre - HPC - Linux - POSIX
- Deployment Types:
  - Scratch: Highly Optimized for Short term
  - Persistent: Longer term, HA (in one AZ), self-healing
- Lazy-Loaded from S3 repository and can be exported (not in sync)
- Split into different targets
  - Metadata targets
  - Object Storage Targets
- Baseline and credit systems for burst 
- Runs in One AZ

### Key Points
- Scratch for pure performance: No HA, No Replication
- Persistent has replication within one AZ (Auto-heals)
- Backup to S3 for both (Manual or Automatic)
- Cannot be used with Windows, only Linux HPC



## AWS Transfer Family
- Used for Protocols other than native protocols 
- Managed service: transfer FROM or TO S3 and EFS
- Managed servers that support FTP, FTPS, SFTP, AS2
- Managed File Transfer Worflows (MFTW) - serverless file workflow engine
- Endpoints Types:
  - Public: only SFTP and Dynamic IP
  - VPC - Internet: SFTP or FTPS, allocated Elastic IP
  - VPC - Internal: SFTP or FTP or FTPS
- For AS2, VPC Internet/Internal only


