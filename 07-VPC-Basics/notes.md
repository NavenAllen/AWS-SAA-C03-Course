## VPC Sizing and Structure
Things to keep in mind
- What size should the VPC be. This will limit the use
- Are there any networks we can't use?
- Be mindful of ranges other VPC's use or are used in other cloud environments
- Try to predict the future uses
- VPC structure with tiers and resilience (availability) zones

VPC min /28 network (16 IP)
VPC max /16 (65456 IP)

Personal preference for 10.x.y.z range

Avoid common range 10.0 or 10.1, include up to 10.10

Suggest starting of 10.16

Reserve 2+ network ranges per region being used per account.
Think of the highest region you will operate in and add extra as a buffer.

An example:

- 3 regions in US, Europe, Aus (5) x 2 - assume 4 AWS accounts
- Total of 40 ranges

VPC services run from within subnets, not directly from the VPC

### How to size VPC

Subnet is located in one availability zone. How many AZ's will your VPC use?
Some regions have more AZ's, but they all have at least 3. It is a good practice
to start splitting the network into at least 4 AZ's.

This would be split into tiers (web, application, db, spare)

Taking a /16 subnet and splititng it 16 ways will make a /20

## Custom VPC

- Regional Service - All AZs in that region
- Allows isolated networks inside AWS
- Nothing IN or OUT without explicit configuration
- Flexible configuration - simple or multi-tier
- Allow connection to other cloud or on-prem networking
- Default or Dedicated Tenancy
  - Default allows on a per resource decision later on
  - Dedicated locks any resourced created in that VPC to be on dedicated hardware which comes at a cost premium.
  - Should pick default most of the time

### VPC Facts

- Can use IPv4 Private CIDR block and public IPs
- Allocated 1 mandatory primary private IPv4 CIDR blocks
  - Min /28 prefix (16 IP)
  - Max /16 prefix (65,536 IP)
- Can add secondary IPv4 Blocks
  - Max of 5, can be increased with a support ticket
  - When thinking of VPC, it has a pool of private IPv4 addresses and can use public addresses when needed.
- Another option is a single assigned IPv6 /56 CIDR block
  - Still being matured, not everything works the same.
  - With increasing use of IPv6, this should be thought of as default
  - Range is either allocated by AWS, or private addresses can be used  that are already owned.
  - Don't have a different of private vs public, they are all routed as public by default.

#### DNS
- Provided by Route 53
- Available on the base IP address of the VPC + 2
- If the VPC is 10.0.0.0 then the DNS IP will be 10.0.0.2

Two important settings which can cause issues.
These are found in the actions area of a VPC
- enableDnsHostnames: **Edit DNS hostnames**
  - Indicates whether instances with public IP addresses in a VPC are given public DNS hostnames. If this is set to true, instances will get get public DNS hostnames.
- enableDnsSupport: **Edit DNS resolution**
  - indidcates if DNS resolution is enabled or disabled in the VPC
  - If True: instances in the VPC can use the dns ip address. **VPC + 2**
  - If False this is not available

## VPC Subnets
- This is what services run from inside VCPs.
- AZ Resilient subnetwork of a VPC. This runs within a particular AZ.
- Runs inside of an AZ. If the AZ fails, the subnet and services also fail.
- 1 subnet can only have 1 AZ
- 1 AZ can have zero or many subnets
- IPv4 CIDR is a subset of the VPC CIDR block.
- Cannot overlap with any other subnets in that VPC
- Subnets can communicate with other subnets in the VPC by default.

### Reserved IP addresses
There are five IP addresses within every VPC subnet that you cannot use.
Whatever size of the subnet, the IP addresses are five less than you expect.

10.16.16.0/20 (10.16.16.0 - 10.16.31.255)
- Network address: 10.16.16.0
- Network + 1: 10.16.16.1 - VPC Router
- Network + 2: 10.16.16.2 - Reserved for DNS
- Network + 3: 10.16.16.3 - Reserved for future AWS use
- Broadcast Address: 10.16.31.255 (Last IP in subnet)

Keep this in mind when making smaller VPCs and subnets.

VPC has a configuration object applied to it called DHCP Option Set.
This is how computing devices recieve IP addresses automatically. There is one option set applied to a VPC at one time and this flows through to subnets.

- This can be changed, can create new ones, but you cannot edit one.
- If you want to change the settings
  - You can create a new one
  - Change the VPC allocation to the new one
  - Delete the old one

Can also define

- Auto Assign IPv4 address
  - This will create a public IP address in addition to their private subnet
- Auto Assign IPv6 address
  - For this to work, the subnet and VPC need an allocation.

These are defined at the subnet level and flow down.

## VPC Routing and Internet Gateway

### VPC Router
- Highly available device available in every VPC which moves traffic from somewhere to somewhere else.
- Router has a network interface in every subnet in the VPC. **network + 1**
- Controlled by 'route tables' defines what the VPC router will do with traffic when data leaves that subnet.
- A VPC is created with a main route table. 
  - If you don't associate a custom route table with a subnet, it uses the main route table of the VPC.
  - If you do associate a custom route table you create with a subnet, then the main route table is disassociated.
  - A subnet can only have one route table associated at a time, but one route table can be associated by many subnets.

### Route Tables
- The higher the prefix, the more specific the route, thus higher priority
- When target says **local** that means the VPC can route to the VPC itself.
- Local routes always take priorty and can never be updated.

### Internet Gateway
- Regional resilient gateway attached to a VPC.
- DO NOT NEED one per AZ. One IGW will cover all AZ's in a region
- 1 VPC can have no IGW or it can have 1 IGW
- A IGW can be created and attached to no VPC, but it can only be attached to one VPC at a time.
- Runs from within the AWS public zone
- It is what allows gateway traffic between the VPC and the internet or AWS Public Zones (S3, SQS, SNS, etc.)
- It is a managed gateway to AWS handles the performance.

IMPORTANT: **IPv4 addresses never touch actual services in the VPC. IGW handles all of this, creates a record for each service.**
**OS knows of IPv6 addresses.**

### Bastion Host / Jumpbox
- These are used to allow incoming management connections.
- Once connected, you can then go on to access internal only VPC resources.
- Used as a management point or as an entry point at a private VPC.


## Network Access Control List (NACL)
Network Access Control Lists (NACLs) are a type of security filter (like firewalls) which can filter traffic as it enters or leaves a subnet.

- STATELESS
- impacts data crossing subnet boundary
- EXPLICIT ALLOW and DENY
- NO Logical resources
- Can ONLY be assigned to subnets
- Each subnet => One NACL
- One NACL can be associated with MANY subnets

## Security Groups
- STATEFUL
- NO EXPLICIT DENY - can't block specific bad actors
- Supports logical resources including other SGs AND itself (self-referencing)
- Attached to ENIs not instances or subnets
- Used in conjunction with NACLs for explicit denys


## Network Address Translation (NAT) gateway
- IP masquerading, hides CIDR block behind one IP. This is many private IPs attached to one public IP.
- This gives private CIDR range **outgoing** internet access.

### Key facts
- Must run from a public subnet to allow for public IP address
- Uses Elastic IPs (Static IPv4 Public)
  - Don't change
  - Allocated to your account
- **AZ resilient service (HA in that AZ)** (NOT region resilient)
  - For a fully region resillient service, you must deploy one NATGW in each AZ
- RT in each AZ with NATGW as target
- Managed service, scales up to 45 Gbps. Can deploy multiple NATGW to increase bandwidth.
- NATGW ONLY supports NACL and NOT SGs



