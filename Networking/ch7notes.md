# Design and implement for Security and Compliance

## Traffic Control

### AWS Shield:
- DDos protection
- Hosted cloudfront, Global Acceleration and Route53 Edge locations
- Covers 96% of known layer 3 and 4 attacks

2 pricings:
- Standard tier (automatically)
- Advanced tier:
    - Additional protection
    - Detailed monitoring
    - WAF no charge
    - 24/7 DDos response team
    - EDos (economic denial of sustainability) coverage (when scaling costs occur)
  
### WAF:
- Layer 7
- Allow/Deny HTTP/HTTPS using ACL
- Applicable to API Gateway, Cloudfront, ALB (ALB and API are region restricted)
- newer version has been released (WAF and WAF classic)

Conditions:
- Cross-site scripting
- Country of origin
- IP address
- Size of request properties
- SQL queries
- String/Regex

Each condition contains one or more filters.
Multiple conditions are OR-ed

Rules:
- One or more conditions
- Specify match or not match
- Multiple conditions are AND-ed
- normal/rate-based (cloudwatch)
- rate-based will result (count) after 5 minutes (cannot be changed)

Customers may purchase managed rule-sets

ACL:
- one or more rules
- allow/deny/count
- Sequential order (count exception will only count, other acls still apply till match)
- Default actions

Price:
- Per ACL/month
- Per Rule/month
- Per 1 million requests

Limits per account/per region:
- 50 ACLS
- 100 Rules
- 5 rate-based rules
- 100 of each condition type except
- 10 regex conditions (cannot be increased)

Maximum 10 conditions per rule

Maximum 10 rules per ACL

### Cloudfront

Enforce traffic going through cloudfront
- S3 bucket origins - restrict traffic using CF Origin Access Identity (bucket policy)
- Custom origins (ec2, albs) - only accept requests that include signed URLs, cookies, or
custom headers added by the CF distribution.

### S3

- S3 access points allow the creation of customized access points for an s3 bucket
- Each access point:
  - has a unique host name
  - has distinct permissions and network controls
  - can be limited to VPCs
- Can be managed by AWS Organizations SCP's

### Security groups

- Applied to ENIs
- Up to 5 per ENI

### ALB

Authentication can be offloaded to the ALB
- Traffic matching a listener rule is send to a configured identity provider

## Traffic Protection

- PKI (Public Key Infrastructure) is a collection of systems used to verify identities
and secure electronic data transfer
- Certificates are used by PKIs to verify identities and ownership of encryption keys

### Digital certificates

contains information to validate the identity of the holder as well as the issuer:
- Name of the certificate holder
- Other information about the holder
- Purpose of the certificate
- Expiration date
- Identity of the issuer (Let's Encrypt)
- Means of validating the authenticity of the certificate

###SSL/TLS

- AWS recommends using TLS 1.2 wherever possible

### AWS Certificate Manager (ACM)

- Certificates in AWS can be managed using either ACM or IAM
  - ACM not available in all regions
  - IAM cannot be used to create certificates (only imports)
  - ACM certificates cannot be imported by IAM
  - IAM certificates cannot be managed from the Console
  - Certificates in format X.509 can be imported into ACM
- Services that integrate with ACM
  - ELB
  - Cloudfront
  - API Gateway
  - CloudFormation
  - Elastic Beanstalk
  
Validation of domain ownership:
- DNS (Add TXT record to zone)
- Email (automatically sent to zone contacts)

Important notes:
- Certificates are provisioned on a per-region basis
- Certificates used by Cloudfront must be provisioned in us-east-1
- ACM certificates can only be used by AWS services integrated with ACM
- ACM certificates and their private keys may not be downloaded

### AWS Certificate Manager PRivate Certificate Authority (ACM PCA)

- Crate your own private CA infrastructure
  - SSL/TLS certificates to identify internal resources
  - $400/month per CA
  - Charge per private certificate

### S3 Https condition policy

```json
{
  "Id": "ExamplePolicy",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowSSLRequestsOnly",
      "Action": "s3:*",
      "Effect": "Deny",
      "Resource": [
        "arn:aws:s3:::DOC-EXAMPLE-BUCKET",
        "arn:aws:s3:::DOC-EXAMPLE-BUCKET/*"
      ],
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      },
      "Principal": "*"
    }
  ]
```

### Protecting infra-AWS traffic - Cloudfront

- Settings configured per origin
- Origin Protocol Policy
  - HTTP only (default)
  - HTTPS Only
  - Match Viewer
- Origin-type specifics:
- Certificate at custom origin servers must be from a Mozilla-trusted CA
- S3 bucket origins are always "Match viewer"
- S3 buckets as websites do not support HTTPS

## Traffic Awareness

### VPC Traffic Mirroring

- Duplicates inbound and outbound traffic processed by a single ENI to either another ENI
or a NLB
- Only one source ENI per mirror session
- Multiple sessions per target
  - NLB - unlimited
  - ENI - 100 max for dedicated instances, otherwise max of 10
- Mirror sessions may include a filter with up to 10 rules to accept or rejct matching traffic
- Traffic is analyzed at customer-managed instances
- Source and target can be in different AWS accounts
- Source and target must be in VPCs connected by peering or TGW
- All VPC routing must be correctly configured
- Mirrored traffic is not captured in flow logs
- Production traffic has highter priority than mirrored traffic
- Hourly charge per source ENI

### Cloudwatch

WAF:
- AllowedRequests
- BlockedRequests
- CountedRequests
- PassedRequests

Shield:
- DDoSDetected
- DDoSAttackBitsPerSecond
- DDoSAttackPacketsPerSecond
- DDoSAttackRequestsPerSecond

Route53 & Cloudfront metrics located in us-east-1

VPC Traffic Mirroring Metrics (in EC2):
- NetworkMirrorIn/Out
- NetworkPacketsMirrorIn/Out
- NetworkSkipMirrorIn/Out
- NetworkPacketsSkipMirrorIn/Out

Note: Always at standard 5 minute reporting (even if enhanced monitoring is enabled)

### AWS GuardDuty

- AWS-managed threat detection
- Automatically analyzes data from Cloudtrail, VPC flow logs, and AWS DNS resolvers to
identify known threats and suspicious behaviors
- GuardDuty findings can be viewed in the Console, CLI, API and Cloudwatch Events
- Pricing based off the amount of data analyzed.

### AWS Inspector

- Performs network accessibility and host vulnerability assessments on EC2 instances
- OS agent must be installed for host vulnerability and process-level network accessibility
assessments.
- Assessments use rule packages created and maintained by AWS security researchers
- Findings include details from both Common Vulnerability Scoring System and the Center
of Internet Security
- Recommends steps to fix issues
- Can be run as needed or scheduled
- Pricing is per assessment, per package

### EC2

- Packet Capture and Analysis
  - Can only capture traffic arriving at local interfaces
  - ENI can be a target for VPC traffic mirroring sessions
    - Only supported on Nitro based images
    - Packet capture software must support VXLAN decapsulation
- Intrusion Detection/Prevention
  - Often sandwiched between ELB layers
  - IDS monitors netowrk and reports events
  - IPS can modify traffic or service configurations in response to events
    - Agent software might be required to configure services
  - Service appliances available in the AWS AMI Marketplace
  
## Governance and compliance



