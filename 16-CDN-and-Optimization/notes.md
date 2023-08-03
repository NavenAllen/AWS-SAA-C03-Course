## CloudFront

### Architecture Basics

Cloudfront is a global object cache, also known as a content delivery network (CDN)

Content is cached in locations close to customers.
If the content is not available on the local cache when requested, ClouldFront
will pull and cache it then.

This provides lower latency and higher throughput for customers.

Can handle static and dynamic content.

**Origin**  the source location of your content
**Distribution** the configuration unit of CloudFront
Edge locations - local infrastructure which hosts a cache of your data.
There are over 200 edge locations. They can be one or more racks in a
third party server system.

Regional Edge Cache - larger version of an edge location. Provides
another layer of caching.

Distributions have **Behaviors** which defines what to do with each path

#### Things set in Distributions
- Price Class
- Alternate Domain Name
- SSL Certificate

#### Things set in Behaviour
- Path Pattern
- Viewer Protocol
- Restricted Access
- Cache directives
- Lambda@Edge


#### Caching Optimisation

Parameters can be passed on the url such as query string parameter.
An example is `?language=en` and `?language=es`

To get the cached copy, you need to use the same query strings. If you do
use them, ensure the strings are in the same order.

### AWS Certificate Manager (ACM)

- HTTP lacks encryption and is insecure
- HTTPS uses SSL/TLS layer of encryption added to HTTP
- Data is encrypted in-transit
- Certificates allow servers to prove their identity
- Signed by a trusted authority (CA).
- To be secure, a website generates a certificate and has a CA sign it. The
website then uses that certificate to prove its authenticity.

Create, renew, and deploy certificates with ACM.

Supported AWS services ONLY (CloudFront and ALB, NOT EC2) (Very Important)

If it's not a managed service, ACM doesn't support it.

Cloudfront must have a trusted and signed certificate. It can't be self signed.

ACM is a regional service, certs cannot leave the region they are generated or imported in.

TO USE A CERT WITH AN ALB IN A REGION, YOU NEED A CERT IN THE SAME REGION (Very very important)
  - Exception is CloudFront since us-east-1 is where distributions are stored

### CloudFront and SSL/TLS:
- SSL needed each application to have own IP
- SNI was added to let client know what site it was trying to access so server can respond with corresponding certificate
- For cloudfront to support legacy browsers, it needs dedicated IPs ($600/per month per distribution)

### Origin and Origin Groups
Origin groups are groups of origins that provide resiliency

Types of origins:
- S3 Bucket
- Media Endpoints
- everything else (web servers, S3 configured with static hosting is also viewed as a web server)

For S3, viewers and origin protocols are the same


### Origin Access Identity (OAI)

This identity can be associated with a cloud front distribution.

The edge locations gain this identity.

We then remove the explicit allows and only allow the OAI to use it.

Best practice is to create one per distribution to manage permissions.

### Private Distributions
Requests require signed cookie or URL

OLD:
- Cloudfront Key is created by an Account Root User
- The account is added as a trusted user

NEW:
- Trusted Key Groups added

SignedURLs provides access to one object ONLY

Cookies provides access to groups of objects

### Lambda@Edge
Lightweight lambda at edge

Adjust data between viewer & origin

Only AWS public

Uses:
- A/B testing: Viewer Request
- Migration between S3 Origins: Origin Request
- Different Objects based on Device: Origin Request
- Content By Country: Origin Request

  
### AWS Global Accelerator

Starts with 2 **anycast** IP address
1.2.3.4 & 4.3.2.1

Anycast IP's allow a single IP to be in multiple locations.
Routing moves traffic to closest location.

Traffic initially uses public internet and enters a global
accelerator edge location.

#### Key concepts

Move the AWS network closer to customers.

Global Accelerator moves the AWS network as close as possible.

Connections enter at edge, using anycast IPs. Transit over AWS backbone to 1+
locations.

Global Accelerator is a network product. Can use TCP, UDP.

Caching is mostly CloudFront. TCP and UDP is mostly Global Accelerator
