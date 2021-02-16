# AWS Certified Cloud Practitioner

- IAM credential Reports
    - Passwords
    - Access keys
    - MFA

Resources - 
The user, group, role, policy, and identity provider objects that are stored in IAM. As with other AWS services, you can add, edit, and remove resources from IAM.

Identities - 
The IAM resource objects that are used to identify and group. You can attach a policy to an IAM identity. These include users, groups, and roles.

Entities - 
The IAM resource objects that AWS uses for authentication. These include IAM users, federated users, and assumed IAM roles.

Principals - 
A person or application that uses the AWS account root user, an IAM user, or an IAM role to sign in and make requests to AWS.

## Cloud Concepts and Technology

### S3

- File can be from 0 Bytes to 5 TB
- Unlimited storage
- Universal (unique name => https://s3-[region].amazonaws.com/[bucketname])
- HTTP 200 when upload successful

Objects:
- Key
- Value
- Version ID
- Metadata
- Subresources:
    - Access Control Lists
    - Torrent
    
Features:
- Tiered Storage Available
- Lifecycle Management
- Versioning
- Encryption
- Secure you data using Access Control Lists (per file) and Bucket Policies (bucket level)

S3 Storage Classes:
- S3 Standard
- S3 - IA
- S3 One Zone - IA
- S3 Intelligent Tiering
- S3 Glacier
- S3 Glacier Deep Archive (Retrieval 12 hours)
- S3 Outputs (Object storage on premise)

Costs:
- Storage
- Requests
- Storage Management Pricing
- Data Transfer Pricing
- Transfer Acceleration (Uses Cloudfront, Uploads to Edge location instead)
- Cross Region Replication Pricing

#### ACL

#### Bucket Policies

### Cloudfront
- Web Distribution
- RTMP (For Adobe Flash media streaming)
- Cost for clearing cache

### EC2
- On Demand
- Reserved (contract 1 -3 years)
  - Standard Reserved Instances (up to 75%)
  - Convertible Reserved Instances (up to 54%) (change to other instance family)
  - Scheduled Reserved instances (in Time windows)
- Spot
- Dedicated Hosts (bound software licenses)

EBS:
- gp2
- io1
- st1 (throughput optimized HDD)
- sc1 (Cold HDD) f.e file servers

### Non-SQL DB's
- DynamoDB
- OLTP (Online Transaction Processing)
  - Order number 2120121 => Pulls up a row of data such as Name, Date, Address, ...
- OLAP (Online Analytics Processing)
  - Pulls large numbers of records (Sum of Radios sold in EMEA, Sales_price - unit_cost)
- Redshift used for OLAP

### Neptune
- Graph Database

### Global services
- IAM
- Route53 
- Cloudfront
- SNS
- SES
- S3 (but created in a Region)

### What AWS Services can be used on premise
- Snowball
- Snowball Edge (lambda on premise)
- Storage Gateway (Physical or virtual - caching files on premise to S3)
- CodeDeploy
- Opsworks
- IoT Greengrass

### AWS Systems Manager
- install agent
- Run Command

### Personal Health Dashboard
- Create Notification if anything could impact your services

### Global Accelerator
- create accelerators to improve availability and performance
- uses AWS backbone network

### AWS Lightsail

## Pricing and billing

### AWS Pricing

#### Different Pricing Models

- Capex (Capital Expenditure) - Pay up front fixed cost
- Opex (Operational Expenditure) - Pay for what you use

#### Free services

- VPC
- Elastic Beanstalk
- Cloudformation
- IAM
- AutoScaling
- Opsworks
- Consolidated Billing

### AWS Budgets vs Cost Explorer

- AWS budgets - set custom budget alerts when exceeds amount
- Cost Explorer - UI visualize usage

#### AWS Support Levels
- Basic
- Developer (Business hour Tech support via mail, 1 Person/unlimited cases)
- Business (24x7 Tech support via mail, chat & phone)
- Enterprise (24x7 Tech support via mail, chat & phone, TAM)

#### Tagging & Resource Groups

- Search resources by tag
- Add new tags to resources
- Saved Resource groups by tag/resource type
- Classic groups are region specific

#### AWS Quick starts

- pre-defined setup via Cloudformation

#### AWS Landingzone

- Creates AWS organizations account
- Created Shared services Account
- Log Archive Account
- Security Account

#### AWS Partner Program

2 Kinds of partnership:
- Consulting
- Technology

Partner tiers:
- Select
- Advanced
- Premier

#### AWS Calculators

2 types:
- AWS simple Monthly Calculator (calculator.s3.amazonaws.com)
- AWS Total Cost of Ownership Calculator (Compare on-premise vs Cloud - aws.amazon.com/tco-calculator)

## Security in the Cloud

### Compliance on AWS & AWS Artifact

- go to https://aws.amazon.com/compliance/
- Artifact (On the AWS console download the compliance report)

### Shared Responsibility Model

- Responsibility AWS vs Customer

### AWS WAF & AWS Shield

- WAF (Layer 7 Firewall, X-scripting, SQL injection)
- AWS shield (DDos Attacks - Standard/Advanced)

### AWS Inspector vs AWS Trusted Advisor

- AWS Inspector (Security and compliance reporting through agent on EC2)
- Trusted Advisor (Reduce Cost, improve security, performance, service limits, fault tolerance)
  - Core checks and recommendations
  - Full Trusted Advisor - Business and Enterprise Companies only

### Cloudwatch vs AWS Config

- AWS Config (See Configurations over time in AWS account)

### AWS Penetration Testing

- Simulated cyber attack
- without approval:
  - EC2, NAT gateways, Loadbalancers
  - RDS/Aurora
  - CloudFront
  - API Gateway
  - Lambda
  - Lightsail
  - Elastic Beanstalk
- Following are not allowed (contact AWS for other):
  - DNS Zone walking
  - DDos
  - Port Flooding
  - Protocol Flooding
  - Request Flooding

### KMS vs HSM

- KMS - Region Secure key management (CMK f.e S3) - shared hardware
- CloudHSM - Physical Dedicated Hardware (FIPS 140-2 Level 3)

### Secrets Manager vs Parameter Store

- Parameter store (Passwords, Database connection strings)
  - Plaintext or KMS encrypted
  - TTL to expire (passwords)
  - No cost - limited to 10.000 parameters per account
- Secrets Managers (Charge per secrets stored and per 10.000 API calls)
  - rotate secrets
  - new key/password in RDS for you
  - Generate random secrets

### GuardDuty

- Uses ML algorithms anomaly detection
- 30 Free trial
- Input Data (Cloutrail event logs, VPC flow logs, DNS logs)

### AWS Security Hub

- security alerts across multiple AWS accounts
  - aggregates, organizes, and prioritizes
  - from multiple AWS services (GuardDuty, Inspector, Macie, IAM Analyzer, and AWS Firewall Manager)

### Compromised IAM Credentials

- Invalidate those credentials

### Athena vs Macie

Athena:
- Query logs stored in S3
- Generate Business reports (s3)
- Analyse AWS cost and Usage reports
- Run queries on click-stream data

Macie:
- Uses AI to recognize if your s3 objects contain sensitive data such as PII
- Dahsboards, reporting and alerts
- Works directly with data stored in S3
- Can also analyze Cloudtrail logs
- Great for PCI-DSS and preventing ID theft

## Advanced AWS Concepts

### AI Services

- Lex (Chatbots Alexa)
- Polly (Text to Voice)
- Transcribe (Speech to Text)
- Rekognizion (Converting images to tags/text)

### EC2 Licensing

- Dedicated hosts for Licensing on Physical Machines

### Different Compute Services

- EC2
- Lightsail
- Lambda
- Batch
- Elastic Beanstalk
- Serverless Application Repository
- AWS Outposts
- Ec2 Image Builder
