## Public Hosted Zones
- A DNS database for a specific domain
- Public = hosted on R53 provided **public DNS servers**
- This is globally resilient (Multiple DNS servers)
- This is created with domain registration via R53, can be created seperately
- There is a monthly fee for non R53 hosted zones
- Host DNS records
- Hosted Zones are what the DNS system refererences, it becomes Authoritative for a domain and visible to the public internet
- Inside VPC, DNS is resolved by R53 Resolver (VPC+2 Address)

## Private Hosted Zone
- Associated with VPCs
- Using different accounts is supported
- Split-view for PUBLIC and INTERNAL use with the same zone name
- Has to explicitly associated with VPCs

### Split-view Hosted Zone
- VPC is associated with Private Zone
- Private Zone records can only be accessed by VPC
- Common architecture is to have a private zone superset
- Public zone is a subset that can be accessed by everyone

## R53 CNAME vs ALIAS
- CNAME maps a NAME to another NAME
- CNAME is invalid for naked/apex

### ALIAS records
- ALIAS records map a NAME to an AWS resource
- Can be used both for naked/apex and normal records
- For non apex/naked - functions like CNAME
- There is no charge for ALIAS requests pointing at AWS resources
- Should be the same "Type" as what the record is point at 

## Route 53 Health Checks
- Route checks will allow for periodic health checks on the servers
- If one of the servers has a bug, this will be removed from the list
- These are seperate from, but used by records on Route 53.
- These are performed by a fleet of global health checkers. If you think they are bots and block them, this could cause alarms
- Checks occur every 30 seconds by default. This can be increased to 10 seconds for additional costs
- These checks are per health checker. Since there are many you will automatically get one every few seconds. The 10 second option will complete multiple checks per second
- There could be one of three checks 
  - TCP checks: R53 tries to establish TCP with end point within 10 seconds
  - HTTP/HTTPS: Same as TCP but within 4 seconds. The end point must respond with a 200 or 300 status code within 3 seconds of checking.
  - String matching: Same as above, the body must have a string within the first 5120 bytes. This is chosen by the user.
- There are three types of checks.
  - Endpoint checks
  - Cloudwatch alarms
  - Checks of checks

## Route 53 Routing Policies
### Simple 
- Route traffic to a single resource
- Client queries the resolver which has one record.
- It will respond with 3 values and these get forwarded back to the client
- The client then picks one of the three at random
- This is a single record only. No health checks.

### Failover 
- Create two records of the same name and the same type
- One is set to be the primary and the other is the secondary
- Route 53 knows the health of both instances
- As long as the primary is healthy, it will respond with this one
- If the health check with the primary fails, the backup will be returned instead
- This is set to impliment active - passive failover

### Multi-value
- Multiple records with same name
- 8 'healthy records' are returned. If more exists, 8 are randomly selected
- Any records which fail health checks wont be returned when queries
- NOT a replacement for load balancing

### Weighted 
- Create multiple records of the same name within the hosted zone
- For each of those records, you provide a weighted value
- If all of the parts of the same name are healthy, it will distribute the load based on the weight
- If one of them fails its health check, it will be skipped over and over again until a good one gets hit
- Don't adjust total weight even if unhealthy
- '0' weight means record is never returned
- This can be used for migration to seperate servers.

### Latency-based 
- Multiple records in a hosted zone can be created with the same name and same type
- Each record is tagged with a region
- When a client request arrives, it knows which region the request comes from
- It knows the lowest latency and will respond with record with lowest latency and is healthy

### Geolocation 
- Focused to delivering results matching the query of your customers
- For US, the record will first be matched with state
- If not, matched based on the country if possible
- If this does not happen, the record will be checked based on the continent
- Finally, if nothing matches again it will respond with the default response
  - If no default record - NO ANSWER
- Can be used for **regional restrictions, language specific content or load balancing across regional endpoints**
- If overlapping regions, the priority will always go to the most specific or smallest region
- IMPORTANT: NOT ABOUT PROXIMITY, ONLY RELEVANCY in records

### Geoproximity
- Return record which is as close as possible to the customers
  - Work on Distance
- Each record is tagged with a location
- A "+" or "-" bias can be added to rules
  - "+" increases a region size and decreases neighbouring regions
 
## Route53 Interoperability
- R53 can do both Domain registrar and Domain Hosting
  - Can do both or only one job

### Usual workflow
- Accepts your money (domain registration fee)
- ALlocates 4 NSs (domain hosting)
- creates a zone file on the NS (domain hosting)
- communicates with the registry of the TLD (Domain registrar)
- sets the NS record for the domain to point at the 4 NS
