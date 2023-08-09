## Template and Pseudo Parameters
### Template Parameters
- Accept input - console/CLI/API
- when created or updated
### Pseudo Parameters
- Template parameter provided by AWS

## Intrinsic Functions
### Ref & Fn::GetAtt 
- Ref - gets main value of a resource
- GetAtt - gets secondary values

### Fn::GetAZs & Fn::Select
- GetAZs - gets all AZs in the regions (returns list of subnets of the default VPC, if improperly configured might not return everything)
- Select lets you pick the AZ

### Fn:Join & Fn::Split
- Join - joins strings with delimiter
- Split - splits strings with delimiter

### Fn::Base64 & Fn::Sub
- Base64 encodes input using Base64
- Sub - used for substituting dynamic values - cant self reference

### Fn:Cidr
- Cidr splits the Cidr range using number of subnets and size of subnet inputs and returns it

## CloudFormation Mappings
- Map keys to values
- Can have one or Top or second level key
- !FindInMap is used

## CloudFormation Outputs
- Optional
- Visible from CLI or consolue UI
- Accessible from Parent Stack

## CloudFormations Conditions
- processed BEFORE resources are created
- uses AND, EQUALS, IF, NOT, OR

## CloudFormation DependsOn
- CF usually does things in parallel
- tried to determine dependency order using references or functions
- DependsOn lets you explicitly define these
- Most common issue: Elastic IP needs Internet Gateway Attachment

## CloudFormation Signals
- hold until 'X' number of success
- wait for timeout (12 hour max)

### Creation Policy
- cfn utility bootstrap is used to send signals to CF

### WaitCondition
- Actual logical resource
- Can wait on other resources and other resources can wait on this too
- Use WaitHandle which created pre-signed url that can be used by external servers to signal too
- Can return attributes of the signal

## CloudFormation Stack Characteristics
- 500 resources per stack
- Stacks are isolated, can't ref from other stacks

## CloudFormation Nested Stacks
- Root Stack is the only stack created manually
- Root/Parent stacks creates nested stacks
- Template URL for nested stacks needs to be provided in the template
- Parameters for nested stacks needs to be within the template
- Parent stack can only access the outputs of nested stacks, not resources
- Parent stack can pass the output of one nested stack to another stack

### When to use Nested Stacks
- Reuse template, NOT the actual resources
- **when stacks form part of one solution - lifecycle linked**
- overcome the 500 resource limit


## CloudFormation Cross-Stack References
- Outputs can be exported
- Exports must have a unique name in the region
- Fn::ImportValue instead of Ref

### When to use cross-stack references
- Service-orientated
- Different application lifecycles
- Stack resources reuse

## CloudFormation StackSets
- deplot stacks across many accounts & regions
- containers in an admin account
- stack instances references stacks in 'target accounts'
- Security - self-managed or service-managed

## CloudFormation DeletionPolicy
- deletion policy defines what to do on each resource before deletion
- Delete (Default), Retain or Snapshot
- Only applies to DELETE not REPLACE

## CFN Stack roles
- Stack roles allow an IAM role to be passed into the stack via PassRole
- A stack uses this role, rather than the identity interacting with the stack to create, update and delete AWS resources
- It allows role separation and is a powerful security feature

## CFN Init
- config directives stored in template
- desired state system - what
- cfn-init helper tool on EC2
- run once as part of bootstrapping

## CFN hup
- detects changes ion resource metadata
- runs actions when a change is detected


## CFN Change Sets
- Lets you preview changes
- can be applied by executing change set

## CFN Custom Resources
- lets CFN integrate with anything it doesn't yet ot natively support
- sends an event and gets data back from something

