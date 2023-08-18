## Virtualization 101

### Emulated Virtualization - Software Virtualization
- A host OS operated on the hardware and it included a hypervisor
- The software ran in priviledged mode and had full access to the hardware on the server.
- Guest operating systems were wrapped in a virtual machine and had devices mapped into their OS to emulate real hardware.
- Drivers such as graphics cards were all software emulated to allow the process to run properly.
- The areas were not real and only allocated space to them for the moment.
- The hypervisor performs **binary translation**. Any calls attempted to make are intercepted and translated in software on the way.
- The guest OS needs no modification, but it slows the guest OS down a lot.

### Para-Virtualization
- The OS is modified to change the **system calls** to **user calls**.
- Instead of calling on the hardware, they call on the hypervisor called hypercalls.
- The OS becomes virtualization aware and got faster, but this was still a software trick.

### Hardware Assisted Virtualization
- The physical hardware itself is virtualization aware.
- When guest OS try to run priviledged instructions, they are trapped by the CPU and do not halt the process.
- They are redirected to the hypervisor from the hardware.
- The problem is multiple OS try to access the same piece of hardware but they get caught up on sharing.

### SR-IOV (Singe Route IO virtualization)
- Allows a network or any addon card to present itself as many mini cards.
- As far as the hardware is concerned, they are real dedicated cards for their use.
- No translation needs to be done by the hypervisor. The physical card handles their own process internally.
- In EC2 this feature is called **enchanced networking**. This means less CPU usage for the guest.

## EC2 Architecture and Resilience

- EC2 instances are virtual machines (OS+Resources)
- Run on EC2 hosts
- Shared hosts or dedicated hosts
  - Every customer is isolated even on the same shared hardware
  - Dedicated hosts pay for entire host, don't pay for instances
- AZ resilient service. They run within only one AZ system.
- A primary elastic network interface is provisioned in a subnet which maps to the physical hardware on the EC2 host.
- Instances can have multiple network interfaces, even within different subnets so long as they're within the same AZ.
- EC2 can make use of remote storage, Elastic Block Store (EBS).
- An instance runs on a specific host. If you restart the instance it will stay on that host.
- Instances stay on the host until either
  - The host fails or is taken down by AWS
  - The instance is stopped and then started, different than restarted.
  - The instance will be relocated to another host in the same AZ.
- A migration is taking a copy of an instance and moving it to a different AZ.
- In general instances of the same type and generation will occupy the same host
- The only difference will generally be their size.

### EC2 Strengths
- Traditional OS+Application compute need. A specific OS with a specific hardware setup.
- Long running compute needs. Many other AWS services have run time limits.
- Server style applications
  - things waiting for network response
  - burst or stead-load
  - monolithic application stack
    - middle ware or specific run time components
  - migrating application workloads or disaster recovery
    - existing applications running on a server and a backup system to intervene

## EC2 Instance Types
- Raw CPU, memory, local storage capacity and type
- Resource ratios: Some might have more CPU while others have more storage. Storage and Data network Bandwidth
- Influences the architecture and vendor: AMD CPU vs Intel CPU
- Influcences features and capabilities with that instance

### EC2 Categories
- General Purpose - default steady state workloads with even resources
- Compute Optimized - Media processing, scientific modeling and gaming
- Memory Optimized - Processing large in-memory datasets
- Accelerated Computing - Hardware GPU, FPGAs
- Storage Optimized - Large amounts of super fast local storage. Massive amounts of IO per second. Elastic search and analytic workloads.

### Naming Scheme
R5dn.8xlarge - whole thing is the instance type. When in doubt give the
full instance type
- 1st char: Instance family. No expectation to remember all the details
- 2nd char: Instance generation. Generally always select the newest generation.
- char after period: Instance size. Memory and CPU considerations.
  - often easier to scale system with a larger number of smaller instance sizes
- 3rd char - before period: additional capabilities
  - a: amd cpu
  - d: nvme storage
  - n: network optimized
  - e: extra capacity for ram or storage

## Storage Refresher
**Direct** local attached storage - these are physical disks on EC2

**Network** attached storage - volumes delivered over the network (EBS)

**Ephemeral** storage - temporary storage. Instant store attached to EC2 host

**Persistent** storage - permanent storage that lives on past the lifetime
of the instance. EBS is persistent storage.

### Three types of storage
##### Block Storage 
- volume presented to the OS as a collection of blocks. No structure beyond that.
- These are mountable and bootable.
-  The OS will create a file system on top of this, NTFS or EXT3 and then it mounts it as a drive or a root volume on Linux.
-  Spinning hard disks or SSD. This could also be delivered by a physical volume
-  You can mount an EBS volume or boot off an EBS volume.

#### File Storage 
- Presented as a file share with a structure. You access the files by traversing the storage.
- You cannot boot from storage, but you can mount it.

#### Object Storage 
- It is a flat collection of objects
- To retrieve the object, you need to provide the key and then the value will be returned.
- This is not mountable or bootable.
- It scales very well and can have simultanious access.

### Storage Performance
- IO Block Size - size of the wheels. This determines how to split up the data.
- IOPS - speed of an engine rev (RPM). How many reads or writes a storage system can accomidate in a second.
- Throughput - end speed of the race car. This can be influenced by the network speed for network storage. Expressed in MB/s (megabyte per second).

Block Size * IOPS = Throughput


## Elastic Block Store (EBS)
- Allocate block storage **volumes** to instances.
- Volumes are isolated to one AZ.
- The data is highly available and resilient for that AZ.
- Different physical storage types available (SSD/HDD)
- Not attached to instance lifecycle - persistent
- Varying level of performance (IOPS, T-put)
- Can backup to S3 and be regionally resillient
- Billed as GB/month.
  - If you provision a 1TB for an entire month, you're billed as such.
  - If you have half of the data, you are billed for half of the month.
 - **No cross-AZ attachment possible**

### General Purpose SSD 
#### GP2
- Min 1GB, Max 16TB
- IO Credit is 16kb
- Each volume has a IO credit bucket
- Capacity 5.4 million IO credits
- Bucket fills with min 100 credits per second
- Beyond 100, 3 IO credits per second, per GB of volume size
- Can burst up to 3000 IOPS
- Volumes greather than 1000GB, baseline is burst and no credit
- Maximum of 16000 IOPS credits per second
- Used for **Boot volumes, low latency interactive apps, dev & test

#### GP3
- No credit size
- Always achieves 3000 IOPS & 125 MiB/s - Standard
- Extra cost for up to 16,000 IOPS or 1000 MiB/s
- 4x Faster Max throughput vs GP2 (1000 MiB/s vs 250 MiB/s) 


### Provisioned IOPS SSD 
- IOPS can be adjusted independently of size
- Consistent Low Latency and Jitter
- Per Volume Maximums
  - Up to 64k IOPS per volume
  - Up to 1k MB/s throughput
  - Upto 256k IOPS per volume (Block Express)
  - Upto 4k MB/s throughput (Block Express)
- Size:
  - 4BG-16TB
  - 4GB-64TB (block express)
- Per GB Maximums
  - io1 50 IOPS/GB
  - io2 500 IOPS/GB
  - BlockExpress 1000 IOOPS/GB
- Per EC2 instance maximums
  - io1 260k IOPS & 7.5k MB/s
  - io2 160k IOPS & 4.75k MB/s
  - io2 Block Express  260k IOPS & 7.5k MB/s
- High Performance, latency sensitive workloads, NOSQL and RDS


### HDD Volume Types
st1 - throughput
sc1 - cold

Uses credit bucket architecture like GP2

#### ST1
- 125 GiB - 16 TiB
- Max 500 IOPS (1MB)
- Max 500MB/s
- 40MB/s/TB Base
- 250MB/s/TB Burst
- Frequent Access
- Throught-Intensive
- Sequential

#### SC1
- Max 250 IOPS
- Max 250 MB/s
- 12MB/s/TB Base
- 80MB/s/TB Burst
- Less frequently accessed workloads
- Colder data requiring fewer scans per day



## EC2 Instance Store
- Local physical storage that instances can utilize attached to an instance.
- **block storage** devices. They're just like EBS but they're local.
- The volumes are physically connected to one EC2 host. They are isolated to that one specific host.
- Highest storage performance in AWS.
- They are included in instance price, use it or lose it.
- They can be attached ONLY at launch. Cannot be attached later.
- Instances can move between hosts for many reasons: 
  - If an instance is stopped and started, that migrates hosts.
  - If a host undergoes AWS maintenance, it will be wiped.
  - If you change the type of an instance, these will be lost.
  - If a physical hardware fails, then the data is gone.

### Performance
- D3 = 4.6 GB/s throughput
- I3 = 16GB/s can achieve upto 1 mil IOP

#### Exam Powerup

- Instance store volumes are local to EC2 host.
- Can only be added at launch time. Cannot be added later.
- Any data on instance store data is lost if it gets moved, or resized.
- Highest data performance in all of AWS.
- You pay for it anyway, it's included in the price.
- TEMPORARY

## EBS vs Instance Store
- **Persistence** - EBS
- **Resilience** - EBS
- **Isolated from instance lifecycle** - EBS
- Resilience w/ App In-built replication - depends
- High Performance - depends
- Super high performance needs - Instance Store
- Cost - Instance Store

### Specific Points
- Cheap = ST1 or SC1
- Throughput.. streaming - ST1
- BOOT - NOT ST1 or SC1
- GP2/3 upto 16k IOPS
- IO1/2 up to 64k IOPS (256k IOPS for Block Express) - needs to be paired with large instances
- RAID0 + EBS utop 260k IOPS (io1/2-BE/GP2/3)
- More than 260k IOPS - Instance Store


## Snapshots, restore, and fast snapshot restore
### EBS Snapshots
- Efficent way to backup EBS volumes to S3.
- Protect data against AZ issues. becomes region resilient.
- Can be used to migrate data between hosts.
- Snapshots are incremental volume copies to S3.
  - The first is a **full copy** of `data` on the volume. This can take some time.
  - Future snaps are incremental
- Volumes can be created (restored) from snapshots.
- Snapshots can be used to move EBS volumes between AZs.
- Snapshots can be used to migrate data between volumes.

### Snapshot and volume performance
- When creating a new EBS volume without a snapshot, the performance is available immediately.
- If you restore a snapshot, it does it lazily.
- If you attempt to read data that hasn't been restored yet, it will pull it from S3
- You can force a read of all data immediately
- You could also use Fast Snapshot Restore (FSR) - Immediate restore
- You can have up to 50 snaps per region. This is set on the snap and AZ.
- FSR is not free and can get expensive with lost of different snapshots.
- You can force the same response by reading every block manually using DD or another tool in the OS.

### Snapshot Consumption and Billing
- They are billed using a GB/month metric.
- This is used data, not allocated data. If you have a 40 GB volume but only use 10 GB, you will only be charged for the allocated data.
- This is not true for EBS itself.
- The data is incrementally stored which means doing a snapshot every 5 minutes will not necessarily increase the charge as opposed to doing one every hour.

## EBS Encryption
- Not encrypted by default but can be setup to be default
- It will use the default CMK unless a different one is chosen.
- Each volume uses 1 unique DEK (data encryption key)
- Snapshots and future volume use the same DEK
- Can't change a volume to NOT be encrypted
- The OS isn't aware of the encryption, there is no performance loss

## EC2 Network Interfaces, Instance IPs and DNS
- An EC2 instance starts with at least one ENI - elastic network interface.
- An instance may have ENIs in seperate subnets, but everything must be within one AZ.
- When you launch an instance with Security Groups, they are on the network interface and not the instance.

### Elastic Network Interface
- MAC address
- Primary IPv4 private address
- 0 or more secondary private IP addresses
- 0 or 1 public IPv4 address
  - Dynamic IP
  - Changes when start/stop 
- 1 elastic IP per private IPv4 address
  - Can have 1 public elastic interface per private IP address on this interface
  - Allocated to your AWS account
  - Can associate with a private IP on the primary interface or  on the secondary interface.
  - If you are using a public IPv4 and assign an elastic IP, the original IPv4 address will be lost. There is no way to recover the original address.
- 0 or more IPv6 address on the interface
  - These are by default public addresses
- Security groups
  - applied to network interfaces
  - will impact all IP addresses on that interface
  - if you need different IP addresses impacted by different security groups, then you need to make multiple interfaces and apply different security groups to those interfaces
- Source / destination checks
  - if traffic is on the interface, it will be discarded if it is not from going to or coming from one of the IP addresses

Secondary interfaces function in all the same ways as primary interfaces except you can detach interfaces and move them to other EC2 instances.

### Exam Power Ups
- Legacy software is licensed using a mac address.
  - If you provision a secondary ENI to a specific license, you can move around the license to different EC2 instances.
- Multi homed (subnets) management and data.
- Different security groups are attached to different interfaces.
- OS - **doesnt see public IPv4** 
-  IPv4 Public IPs are Dynamic, starting and stopping will kill it
- Public DNS for a given instance will resolve to the primary private IP address in a VPC.
  - If you have instance to instance communication within the VPC, it will never leave the VPC

## Amazon Machine Image (AMI)
- Images of EC2.
- AMI's can be used to launch EC2 instance.
- When you launch an EC2 instance, you are using an Amazon provided AMI.
- Can be Amazon or community provided
- Marketplace (can include commercial software)
  - Will charge you for the instance cost and an extra cost for the AMI
- Regional, unique ID
  - ami-`random set of chars`
- Controls permissions
  - Default only your account can use it
  - Can be set to be public
  - Can have specific AWS accounts on the AMI
- Can create an AMI from an existing EC2 instance to capture the current config

### AMI Lifecycle

#### Launch

EBS volumes are attached to EC2 devices using block IDs
- BOOT /dev/xvda
- DATA /dev/xvdf

#### Configure

- Can customize the instance from applications or volume sizes

#### Create Image
- Once it has been customized, an AMI can be created from that
- AMI contains:
  - Permissions: who can use it
  - EBS snapshots are created from attached volumes
    - Block device mapping links the snapshot IDs and a device ID for
    each snapshot.

#### Launch
When launching an instance, the snapshots are used to create new EBS
volumes in the availability zone of the EC2 instance and contain the same
block device mapping.

### Exam Powerups
- AMI can only be used in one region
- AMI Baking: creating an AMI from a configuration instance.
- An AMI cannot be edited. If you need to update an AMI, launch an instance, make changes, then make new AMI
- Can be copied between regions
- Remember permissions by default are your account only
- Billing is for the storage capacity for the EBS snapshots the AMI references

## EC2 Pricing Models
### On-Demand Instances
- Hourly rate based on OS, size, options, etc
- Billed in seconds (60s min) or hourly
  - Depends on the OS
- Default pricing model
- No long-term commitments or upfront payments
- New or uncertain application requirements
- Short-term, spiky, or unpredictable workloads which can't tolerate any disruption.

### Spot Instances
- Up to 90% off on-demand
- Depends on the spare capacity
- You can set a maximum hourly rate in a certain AZ in a certain region.
- If the max size you set is above the spot price, you pay for the instance.
- As the spot price increases, you'll keep paying until the price increases.
- Once this price increases too much, it will terminate the instance.
- Great for data analytics when the process can occur later.

### Reserved Instance
- Up to 75% off on-demand
- The trade off is commitment.
- You're buying capacity in advance for 1 or 3 years.
- Unused Reservation still billed
- Can partially cover larger instance
- Flexibility on how to pay  
  - All up front
  - Partial upfront
  - No upfront
 
#### Scheduled Reserved Instances
- Ideal for long term usage which doesn't run constantly
- Doesnt support all isntance types or regions
- 1200 hour per year min and min. 1year

### Dedicated Hosts
- Pay for the whole host
- Manage the capacity yourselves
- Used for licensined based on Sockets/Cores
- Host Affinity
- Only your instances run on the host

### Dedicated Instances
- Dedicated Hardware
- No paying for or sharing host
- Extra charges for instances
- Really strict compliance requirements

### Capacity Reservations
- Reserve capacity and not instances
- Regional reservation provides discount for launching in any AZ in that region
  - same priority as on-demand instances
- Zonal Reservations
  - Apply to one AZ
  - Billing discounts and capacity reservation in that AZ
- On-Demand Capacity Reservartions
  - Ensure always have capacity in an AZ
  - Full On-Demand Price
  - Pay regardless of when you use it
  - NO COMMITMENT NEEDED
 
### EC2 Savings Plan
- Hourly commitment of spending for a 1 or 3 year plan
- Flexibility on size and OS
- Have an on-demand rate and a savings plan rate
- Get savings plan rate upto the committed amount

## Instance Status Checks and Autorecovery

Every instance has two high level status checks

### System Status Checks
- Failure of this check could indicate software or hardware problems of the EC2 service or the host.

### Instance Status Checks
- This is specific to the file system or has a corrupted Kernel

Assuming you haven't launched an instance, this is a problem and needs to be
fixed.

### Create Status Check Alarm

This feature has four options

- Recover this instance: can be a number of steps depending on the failure
- Stop this instance
- Terminate this instance: useful in a cluster
- Reboot this instance:

## Horizontal and Vertical Scaling

### Vertical Scaling
- As customer load increases, the server may need to grow to handle more data.
- The server can increase in capacity, but this will require a reboot.
- Often times vertical scaling can only occur during planned outages.
- Larger instances also carry a $ premium compared to smaller instances.
- There is an upper cap on performance - instance size.
- No application modification is needed.
- Works for all applications, even monoliths (all code in one app)

### Horizontal Scaling
- As the customer load increases, this adds additional capacity.
- Instead of one running copy of an application, you can have multiple versions running on each server.
- This requires a load balancer.
- When customers try to access an application, the load balancer ensures the servers get equal parts of the load.
- This means the servers are **stateless**, the app stores session data elsewhere.
- No distruption while scaling up or down.
- No real limits to scaling.
- Uses smaller instances so you pay less, allows for better granulairity.

## Instance Metadata

- EC2 service provides data to instances
- Accessible inside all instances
- Memorize **<http://169.254.169.254/latest/meta-data/>

Meta-data contains:

- enviroment the instance is in
- networking is the big reason why
- authentication information
  - instances can be used to gain access on other services
- user-data
- NO AUTHENTICATION or ENCRYPTED
  - Anyone who can gain access and it can and will get exposed
  - Can be restricted by local firewall
