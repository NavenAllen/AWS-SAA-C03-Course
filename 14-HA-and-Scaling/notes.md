## ELB Evolution
- 3 types of load balancers
- Split between v1 (avoid/migrate) and v2 (prefer)
- Classic Load Balancer (CLB) - v1
  - Not really layer 7, 1 SSL per CLB
- Application Load Balancer (ALB) - v2 - HTTP/S/WebSocket
- Network Load Balancer (NLB) - v2 - TCP, TLS, UDP
- v2 = faster, cheaper, support target groups and rules

## Elastic Load Balancing
- ELB is a DNS A Record pointing at 1+ Nodes in AZ
   - Each AZ has one node and it's not a central server
- Nodes in each AZ can scale and is HA
- Internet-Facing LB has public IPv4 but can connect to private instances too (instances doesnt have to be public)
- Internal LB has only private IPv4 address
- *Listener* configuration controls what Load Balancer does
- ELB needs 8+ (/28) free IP addresses per Subnet, but AWS recommends /27 

### Cross zone load balancing
- Each node that is part of the load balancer is able to distribute load even if its not in the same AZ
- It is the reason we can achieve a balanced distribution of connections behind a load balancer
- It can also provide health checks on the target servers.
- If all instances are shown as healthy, it can distribute evenly.



## Application Load Balancer (ALB)
- ALB is a layer 7 - only on HTTP and/or HTTP, nothing else
- Can understand Layer 7 content
- NO TCP/UDP/TLS Listeneres
- HTTP HTTP SSL/TLS always terminated on ALB and broken, new connection to application
- ALBs MUST have SSL certs if HTTPS is used
- ALBs are slower than NLB
- Health checks evaluate application health
- ALBs have rules that decided what to do and processed in priority order
  - Different rules can have different target groups (Ex. based on enterprise customer static source IP)
  - Redirect domains 

## Network Load Balancer (NLB)
- Layer 4 device, cant understand HTTP or HTTPS
- Really fast, default to this for any non http/s
- Health checks JUST check ICMP/TCP Handshake
- NLB's can have static IPs - useful for whitelisting
- Unbroken Encryption
- Used with private link to provide service to other VPCs

## When to use ALB or NLB
- Unbroken Encryption: NLB
- Static IP for whitelisting: NLB
- Fastest Perfromance: NLB
- Not HTTP/S: NLB
- Privatelink: NLB

ANYTHING ELSE: ALB


## Launch Configuration and Templates

### LC and LT key concepts:
- They are documents which allow you to config an EC2 instance in advance.
- You can configure userdata and IAM role along with networking and security groups.
- NOT editable but LC has versions

### Launch templates
- Provide T2/T3 Unlimited, placement groups with more.  
- Newer and recommended to use over launch configurations, they include the latest features and improvements.
- Supports versioning of templates.
- Can be used to save time when provisioning EC2 instances from the console UI / CLI.



## Auto Scaling Groups
- Automatic scaling and self-healing for EC2
- They make use of Launch Templates to know what to provision
- They use one version or one configuration they're assigned with
- Minimum, Desired, and Maximum Size
- Provision or Terminate instances to keep at the desired level
- Scaling Policies automate based on metrics or a schedule
- Auto Scaling Groups will try to keep the AZs equal with the number of EC2 instances.

### Scaling Policies
- Manual Scaling - manually adjust the desired capacity
- Scheduled Scaling - time based adjustments
- Dynamic Scaling
  - Simple : If CPU is above 50%, add one to capacity
  - Stepped : Bigger +/- based on difference 
  - Target : Desired aggregate CPU = 40%, ASG will achieve this, can not use all metrics 
- Cooldown Period - How long to wait at the end of a scaling action before scaling again.


AGS can use the load balancer health checks rather than EC2 - health checks need to be proper according to application complexity

### Scaling Processes
- Launch and Terminate - SUSPEND and RESUME
- AddToLoadBalancer
- AlarmNotification
- AZRebalance
- HealthCheck
- ReplaceUnhealthy
- ScheduledActions - Scheduled on/off
- Standby - Suspend actitivty of ASG on a particular instance


### Final points
- Free
- Use cool downs to avoid rapid scaling
- more, smaller instances: granuality
- Use with ALB's for elasticity
- ASG defines when and where, Launch Template defines what.

## ASG Lifecycle Hooks
- Define a custom action during scale up or scale down
- Instance Launch of Instance terminate transitions
- Instances are paused within the flow.. they wait
  - until a timeout (then either CONTINUE or abandon)
  - or you resume the ASG process using CompleteLifecycleAction
- Eventbridge or SNS notifications 

Example EC2 Scale Out Lifecycle hook:
Pending -> Pending: Wait (Do custom actions like SNS topic) -> Pending: Proceed -> InService

## ASG Health Checks
- EC2 (Default), ELB (can be enabled) & Custom
- EC2 - Stopping, Stopped, Terminated, Shutting Down or Impaired = UNHEALTHY
- ELB - Healthy = Running & passing ELB health check
  - can be more application aware (Layer 7)
- Custom - Instances marked healthy and unhealthy by an external system
- Health check grace period (default 300s) - delay before starting checks
  - allows system lauch, bootstrapping and application start


## SSL Offload 
### Bridging - Default mode
- One or more clients makes one or more connections to a load balancer.
- The load balancer is configured so the **listener** uses HTTPS, SSL connections occur between the client and the load balancer
- The load balancer then needs an SSL certificate that matches the domain name of the application
- If you need to be careful of where your certificates are stored, you may have a problem with this system
- ELB initiates a new SSL connection to backend instances with a removed HTTPS certificate. This can take actions based on the content of the HTTP
  - It needs to decrypt any data that is being encrypted by the client
- The EC2 will need matching SSL certificates
  - Needs the compute for the cryptographic operations. Every EC2 instance must peform these cryptographic operations.

### Pass-through
- The client connects, but the load balancer passes the connection along without decrypting the data at all
- The instances still need the SSL certificates, but the load balancer does not
- The load balancer is configured for TCP, it can see the source or destinations, but it never touches the encrypted connection
- The certificate never needs to be seen by AWS
- Negative is you don't get any load balancing based on the HTTP part because that is never exposed to the load balancer.

### Offload
- Clients connect to the load balancer using HTTPS and are terminated on the load balancer
- The LB needs an SSL certificate to decrypt the data, but on the backend the data is sent via HTTP
- While there is a certificate required on the load balancer, this is not needed on the EC2 instance
- Data is in plaintext form across AWS's network. Not a problem for most.

## Connection Stickiness
- If there is no stickiness, each time the customer logs on they will have a stateless experience
- Session Stickiness is an option
- If enabled, the first time a user makes a request, the load balancer generates a cookie called AWSALB
- A valid duration is between one second and seven days
- For this time, sessions will be sent to the same backend instance. This will happen until:
  - If we have a server failure, then the user will be moved to a different server.
  - The cookie could expire, the whole process will repeat and will recieve a new cookie and the process will start again.
- This could cause backend unevenness

## Gateway Load Balancer
- Helps run and scale 3rd party applications
  - firewalls, intrustion detection and prevention systems
- Inbound and Outbound traffic
- GWLB endpoints - traffic enters/leaves via these endpoints
- GWLB balances multiple backend appliances
- Traffic and metadata is tunnelled using GENEVE protocol
  - Original packets remain unaltered encapsulated through to applications and back
  - Gateway Load Balancer initially sends packets to Horizontally scaling appliances for security



