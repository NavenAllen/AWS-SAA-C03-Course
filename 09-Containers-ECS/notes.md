## Intro to Containers
- Container provides isolated environments
- Runs as a process inside host OS which is isolated
- Dont need to run a whole OS for each application
- Much lighter

### Image Anatomy
- Container is a running Image
- Images are created from a base image
- Every change after that are created as file system later
- Each later contains differences between before layer

### Container Anatomy
- Has a R/W layer in addition to the images
- If multiple containers are created, same base layers are used but different R/W layers

### Container Registry
- A registry of container images
- Can be deployed into docker hosts

### Key Concepts
- Dockerfiles are used to build images
- Portable
- Lightweight - fs layers are shared
- Container only runs the application and the environment it needs
- Ports need to be **exposed**
- Application stacks can be multi-container

## Elastic Container Service (ECS) Concepts
- ECS allows you to create a cluster. Clusters are where containers run from.
- ECS supports ONLY Docket standard
- Container images will be located on a registry. AWS provides ECR (elastic container registry).
- Container definition tells ECS where the container image is.
- Task definition represents the application as a whole and stores whatever information is needed to run the application. 
  - (can have multiple customer definitions)
- Task definitions store the resources and networking used by the task.
- It also stores **task role**, an IAM role that allows the task complete its task. 

### Service Definition. 
- This defines how many copies of the task is allowed to run to load balance.
- You can use a service to provide scalability and high availability.
- Tasks or services get deployed to an ECS cluster.

### Key concepts
- **Container definition** the image and the ports that will be used
- **Task definition** the task role and security is defined. 
 - **Task role** IAM role which the task assumes.
- **Service** how many copies of a task you want to run for scaling and high availability.

## ECS Cluster Types
### EC2 mode
- ECS cluster is created within a VPC. It benefits from the multiple AZs that are within that VPC.
- You specify an initial size which will drive an **auto scaling group**
- ECS using EC2 mode is not a serverless solution, you need to worry about capacity for your cluster.
- The container instances are not delivered as a managed service, they are managed as normal EC2 instances.
- This is good because you can use spot pricing or prepaid EC2 servers.

#### Fargate mode
- Removes more of the management overhead from ECS, no need to manage EC2.
- There is a **fargate shared infrastructure** which allows all customers to access from the same pool of resources.
- Fargate deployment still uses a cluster with a VPC where AZs are specified.
- For ECS tasks, they are injected into the VPC. Each task is given an elastic network interface which has an IP address within the VPC. They then run like an VPC resource.
- You only pay for the container resources you use.

### EC2 vs ECS(EC2) vs Fargate
- If you already are using containers, use **ECS**
- **EC2 mode** is good for a large workload with price conscious. This allows for spot pricing and prepayment.
- Large workload but overhead conscious **Fargate**
- Small or burst style workloads **Fargate** makes sense
- Batch or periodic workloads **Fargate**

## Elastic Container Registry (ECR)
- Managed Container image registry service
- Each AWS account has a public and private regisrty
- Inside registry can have many repositories
- Each repository can contain many images
- Images can have several tags
- Public = public R/O - R/W needs permissions

### Benefits
- Integrated with IAM
- Image Security Scanning - Basic and enhanced (inspector)
- Real-time Metrics => Cloud Watch
- API Actions = Cloud Trail
- Events => EventBridge
- Replication

## Kubernetes
- Container Orchestration
- Cloud Agnostic

### Cluster Structure
- HA cluster of compute resources
- Control Plane manages the cluster scheduling, application, scaling and deploying
- Cluster Nodes - VM or Physical Server
   - containerd or Docker Software for handling container operations
   - kubelet - agent to interact with the control plane using Kubernetes API

### Cluster Detail
- Pods are the smallest units of computing (one-container-one-pod is v common)
  - Multiple pods are in same container only if tightly coupled
  - TEMPORARY - not HA
- kube-apiserver - front end of control plane
  - can be horizontally scalled for HA and performance
- etcd - HA key value store with cluster
- kube-scheduler - identifyes any pods withing the cluster with no assigned node and assigns a node
- cloud-controller-manager - provides cloud-specific control logic
- kube-controller-manager - cluster controller processes
  - Node Controller - monitor node outages
  - Job controller - one-off takes (jobs) -> JOBS
  - Endpoint Controller - populates endpoints (Services <=>PODS)
  - Service account and Token Controllers - Accounts/API Tokens
- kube-proxy - network proxy
  - coordinates networking with the control planes
 
### Summary 
- Cluster - deployment of Kubernetes
- Node - Resources
- Pod - smallest unit in kubernetes
- Service - Abstraction, service running on 1or more pods
- Job - ad-hoc, creates one or mode pods until completion
- Ingress - Exposes a way into a service
- Ingress Controller - used to provide ingress
- Should be stateless from pod-perspective
- Persistent Storage (PV) - Volume whose lifecycle lives beyond any 1 pod using it

## Elastic Kubernets Service
- AWS Managed Kubernetes - open source & cloud agnostic
- Control Plane scales and runs on multiple AZs
- Integrates with AWS services
- EKS Cluster - EKS Control plane & EKS Nodes
- etcd distributed across multiple AZs
- Nodes - Self Managed, Managed node groups or Fargate Pods
- Storage Providers - EBS, EFS, FSx Lusture, FSx for NetApp ONTAP

### Architecture
- Control Plane VPC is AWS manages and Multi-AZ
- Control Plane ENIs injected into Customer VPC
- Worker Nodes run on Customer VPC
- They use ENIs or public control plane endpoint to communicate with Control Plane
- Ingress config is in Customer VPC
