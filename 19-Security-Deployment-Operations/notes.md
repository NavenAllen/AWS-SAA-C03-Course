## AWS Secrets Manager
- Designed specifically for secrets (passwords, API keys)
- Usable via Console, CLI, API, or SDK
- Supports the automatic rotation using Lambda.
- Direct integration with AWS products (great for rotating secrets and especially with RDS)

## WAF (Web Application Firewall)
- Level 7 firewall
-  WEBACL Default Action - Non matching
- Resource Type - CloudFront or Regional Service
- Rule Groups or Rules - processed in order
- Web ACL Capacity Units (WCU) - default max 1500 (can be increased via support ticket)
- Associatin with resources can take time, but adjusting WEBACL takes less time

### Rule Groups
- contains Rules
- Dont have default actions, defined when groups or rules are added
- Managed (AWS or Marketplace), Yours, Service Owned
- Have a WCU Capacity

### Rules
- Type, Statement, Action
- Type: Regular or Rate-based
- Statement: WHAT to match or COUNT ALL or WHAT & COUNT
- Different parameters can be checked but for body only first 8192 bytes ONLY
- Single, AND, OR, NOT
- Action: Allow, Block, Count, Captcha.. Custom Response, Label
- For rate-based ALLOW is not valid
- Labels are internal to WAF - multi-stage rules
- Labels can only be used with Count/Captcha since other actions stop processing

## AWS Shield
- DDoS Protection
- Standard (Free), Advanced (Paid)

### Standard
= Free for AWS customers
- Protection at perimeter
- Common Network (L3) or Transport (L4) attacks
- Best using R53, CloudFront, Global Accelerator
- No config, works in background

### Advanced
- $3000 per month (per ORG), 1 year lock-in, data (OUT)/m
- Protects anything associated with EIPs
- Not automatic, has to be configured
- Cost Protection for unmitigated attacks
- Proactive Engagement & AWS Shield Response team
- Integrated with WAF
- Real Time visibility for DDoS events
- Health-based detection (required for Proactive engagement team)
- Protection groups of resources

## CloudHSM
- KMS is shared with applications
- True "Single Tenant" HSM
- AWS provisioned.. but customer managed
- Fully FIPS 140-2 Level 3 (KMS is L2 Overall, some L3)
- Industry Standard APIs - PKCS#11, Java Cryptography Extensions (JCE), Microsoft CryptoNG
- KMS can use CloudHSM as a custom key store
- Deployed on separate CLoudHSM VPC that we have no visibility of
- Not HA by default
- AWS has no access to secure area

### Use Cases
- No Native AWS integration
- Offload the SSL/TLS processing for web servers
- Enable Transparent Data Encryption for Oracle Databases
- Protect Private Keys for Issuing Certificate Authority

## AWS Config
- Record config changes over time on resources
- Auditing changes, compliance with standards
- Doesn't prevent changes happening
- Regional Service.. support cross-region and account aggregation
- Can generate SNS notifications or events using EventBrige & Lambda

- Standard functionality is recording changes (Configuration Item)
- Extra feature is resources are evaluated against Config Rules to generate notifs

## Amazon Macie
- Data Security and Data Privacy Service
- Discover, Monitor and Protect Data stored in S3 Buckets
- Automated Discovery of data i.e PII, PHI, Finance
- Managed Data Identifiers - Built-in - ML/Patterns
- Custom Data Identifiers - Propreitary - Regex
- Integrates - with security hub & 'finding events' to event bridge
- Centrally manage.. either with AWS ORG or inviting

### Custom Identifiers
- Regex - defintes a patter
- Keywords - optional sequences that need to be in proximity to regex match
- Maximum Match Distance - how close keywords are
- Ignore words - if regex match contains these words, they are ignored

### Findings
- Policy Findings - check for restricted bucket policy changes
- Sensitive Data findings

## Amazon Inspector
- Scans EC2 instances & the instance OS
- Also scans containers
- VUlnerabilites and deviations against best practice
- Provides a report of findings ordered by priority
- Network Assessment (Agentless)
  - Network reachability test
- Host Assessment (Needs Agent)
  - CVE findings
  - Center for Internet Security (CIS) Benchmarks
  - Security Best Practices

## Amazon Guardduty
- Continous security monitorning service
- Analyses supported Data sources using AI/ML, threat intelligence feeds
- Identifies unexpected and unauthorised activity
- notify or even-driven protetction
- Supports multiple accounts (MASTER and Member)








