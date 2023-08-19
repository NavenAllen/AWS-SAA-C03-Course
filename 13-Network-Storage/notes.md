## EFS Architecture
- EFS moves the instances closer to a stateless
- EFS is an implementation of NFSv4
- EFS Filesystems can be mounted in Linux
- Using Amazon EFS with Microsoft Windows–based Amazon EC2 instances is not supported
- Media can be shared between many EC2 instances
- This is a private service, access is via mount targets inside a VPC
- You can access from on-premises with VPN or DX so long as access is configured.

### Elastic File System Explained
- EFS runs inside a VPC
- EFS includes POSIX permissions. These are made available via mount targets
- For a fully highly available system you need a mount target for each AZ the system runs in
- You can use hybrid networking to connect to the same mount targets.

### EFS Exam Powerup
- EFS is Linux Only
- Two performance modes, general purpose and max I/O performance mode.
  - General purpose = default for 99.9% of uses
- Bursting and provisioned throughput modes
- Two storage classes available, standard and infrequent access
  - Lifecycle policies can move data between EFS storage classes

## AWS Backup
- Fully managed data-protection service
- Consolidate management of backups in one place
- Supports a wide range of AWS backups

### Key Concepts
- Backup Plans - frequency, window, lifecycle, vault, region copy
- Resources - what resources are backed up
- Vaults - Backup Destination - assign KMS Key for encryption
  - Vault Lock - Write-once-ready-many, 72 hour cool off
- On-Demand backups
- PITR - Point in Time recovery
