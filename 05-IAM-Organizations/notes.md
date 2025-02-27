## IAM Policies
- Explicit Deny, Explicit Allow, Implicit Deny
- If there are different policies for access, they are all combined and priority order checked
- Inline Policies
  - JSON for each identity
- Managed Policies
  - Own object
  - Can be reused and assigned to identities
 
## IAM Users 
- Used for long-term AWS access
- 5000 IAM Users per account
- User can be a member of max 10 groups

## IAM Groups
- No credentials, no login
- No default all-users group
- No Nesting
- 300 groups per account (can be increased with ticket)
- NOT a true identity, cannot be referenced as a PRINCIPAL

## IAM Roles
- Assumed, identities become that role
- Trust Policy - identites that can assume the role
- Permissions Policy - resources that can be accessed by Role
- Generated temporary credentials
- Generated by Secure Token Service (STS)

### When to use roles
- When resources need permissions over other resources (Example: Lambda)
- For temporary priviledge escalation
- Single Sign-On using existing identities or Active directory
  - External accounts cant use AWS directly
- Millions of users (Web federation)
  - No AWS credentials on app
  - use existing customer logins
- Cross-account access

### Service-Linked Roles
- Linked to specific service
- provides permissions to interact with other services
- cant delete until role until not required
- Users can pass roles to resources that have access to do certain access (PassRole)

## AWS Organizations
- Organization Root is NOT the same as Account Root User
- Org Root is the top level of the hierarchichal tree of Organizational Units
- Master Account is Management Account or Payer Account

### Service Control Policies
- Account Permission Boundaries
- What each account (including Root User) can do
- They don't GRANT any permissions
- Allow list vs Deny list
- Common intersection between Identity Policies and SCPs are allowed permissions

## CloudTrail
- Management events
  - Control Plane events
- Data event
  - Adding data, accessing data, so on
- Regional service
- Global services log to us-east-1
- Or else log to the region they are present in

### Key Points
- Enabled by Default - 90 events only - no s3
- Trails are how you configure S3 and CWLogs
- Management events only by default, data events come at a price
- NOT REALTIME

## AWS Control Tower
- Quick and Easy setup of multi-account
- Orchestrates other AWS services to provide this functionality
- Landing Zone - multi-account environment - SSO/ID federation, centralised logging
- Guard Rails - Detect/Madate rules/standards
- Account Factory - Automates and Standardises new account creation
- Dashboard - single page for whole org

### Landing Zone
- Well architected multi-account environment
- Build with AWS Orgs, AWS Config and CloudFormation
- Security OU - Log Archive and Audit Accounts
- Sandbox OU - Test/less rigid security
- IAM Identity Center - SSO, ID Federation
- Monitoring and Notifications - SNS and Cloud Trail

### Guard Rails
- Rules for multi-account governance
- Mandatory, Strongly Recommended or Elective
- Preventive using AWS SCP
- allow or deny regions or disallow bucket policy changes
- Detective - compliance checks (AWS Config)
    - clear, in violation or not enabled

### Account Factory
- Automated Account Provisioning
- Guardrailes - Automatically added
