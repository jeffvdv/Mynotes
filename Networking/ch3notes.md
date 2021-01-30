# Configure Network Integration with Application Services

## VPC DHCP

- Client (UDP 68)
- Server (UDP 67)
- Discover
- Offer
- Request
- Acknowledge

### Reserved Addresses

- .0 Network Address
- .1 VPC Router (including DHCP)
- .2 Reserved for DNS
- .3 Future Use
- .255 Broadcast

### DHCP Option Sets

Only 1 DHCP option set per VPC

- Domain name (custom domain name for internal use)
- Domain name servers
- NTP servers
- NetBIOS name servers
- NetBios node type

## Elastic Load Balancer ALBs, NLBs and Classic

- External or Public facing loadbalancers
  - public subnet
  - subnet must be sized /27 or larger (you cannot use /28 or smaller)
  - the subnet must have at least 8 available IP addresses
- Internal Load balancer (private VPC)

ALBs and NLBs support SNI or the ability to have multiple certificates per listener. CLBs do not.

## Cloudfront

- You can have only 1 geo restriction per distribution
- Either whitelist or blacklist
- You can also configure WAF to block
- Geo restriction only by country
- OAI - Origin Access Identity (when a private s3 bucket is used)

### Cloudfront Behaviors

- HTTP, HTTPS, Redirect HTTP to HTTPS
- Allowed HTTP methods (GET, HEAD, OPTIONS, ...)
- Field-level encryption (extra security layer on top of https)
- Cached HTTP Methods
- Cache Based on Selected Request Headers
- Object caching
    - Use Origin Cache Headers
    - Customize
    - TTL settings
    - Forward Cookies
    - Query String Forwarding and Caching
    - Smooth Streaming
    - Restrict Viewer Access (Use signed urls or signed cookies)
    - Compress Objects Automatically
    - Lambda Function Associations

### Signed URLS and Cookies

- Signed URLS: Supported by web and RTMP origins. Used to control access to individual files. 
  Signed Urls will change your url
- Signed Cookies: Is only supported by web origins and allows access for large groups of 
  files. Urls are not changed.
- Origin Access Identities (OAI): Restrict Access to an S3 bucket to only a special Cloudfront
  user associated with you distribution.

### SSL/TLS encryption

- AWS Certificate Manager
    - Default Cloudfront Certificate
    - Custom SSL Certificate
    - If using ELB as origin (certificates must be issued with ACM)
    - If not using an ELB, certificates must be issues by a CA

## Lambda@Edge


## VPC Enpoint Services with AWS PrivateLink


## Amazon Workspaces

![Image of instance types](https://github.com/jeffvdv/Mynotes/blob/master/Networking/assets/workspaces-architecture.png)

- Desktop as a service (Windows desktop)
- spin up in minimum 2 AZs
- VPC managed by AWS (won't see vpc)
- ENIs will be launched in our VPCs (connected to SimpleAD)
- Another ENI for connecting to WorkSpaces (public IP or private IP)
- Subnet max /17 to min /28 CIDR subnet (Reserve 5 IP addresses in each subnet + 1 
  Directory service address in each subnet)
- Private ip use NAT translation (additional IP)
- As a Directory Service you can use:
  - AWS Managed Microsoft AD
  - Simple AD 
    - small - 500 users, 2000 objects
    - large - 5000 users, 20000 objects
  - AD Connector (Proxy for redirecting requests to existing AD without caching information)
- 1 Directory can only be linked to 1 Workspace

## Amazon AppStream 2.0

- Managed Application Service (3D design, Adobe, IDEs, ...)
- You should use 2 AZ's
- You should see an ENI
- AD or SAML authentication
