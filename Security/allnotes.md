# AWS Certified Security Specialty

## Security 101

### Security Basics

- CIA Model
    - Confidentiality (private information => Encryption/MFA/Passwords/IAM/Policies/SGs)
    - Integrity (Data Consistency => File permissions/VCS/Checksums/Certificate Manager/IAM/Bucket policies)
    - Availability (Autoscaling/Multi-AZ)
- AAA
    - Authentication (IAM)
    - Authorization (Policies)
    - Accounting (Cloudtrail)
- Non repudiation (can't deny that you did something)

### Security of AWS

Physical and Environmental Security
- Fire Detection and Suppression
- Power
- Climate and Temperature
- Management
- Storage Device Decommissioning

Business Continuity Management
- Availability
- Incident Response
- Company-wide Executive Review
- Communication (to customers/Service Health Dashboard)

Network Security
- Secure Network Architecture
- Secure Access Points
- Transmission Protection (SSL)
- Amazon Corporate Segregation
- Fault-Tolerant Design
- Network Monitoring and Protection (DDos Mitigation/ AWS Shield)

AWS Access
- Account Review and Audit
- Background checks
- Credentials Policy

Secure Design Principles (penetration Testing)

Change Management
- Software (reviews code)
- Infrastructure (notification via e-mail to customers)

AWS Compliance Programs:
- ISO 27001
- HIPAA
- SOC ..
- ...

### Security in AWS

- Visibility
  - AWS Config (Current Resources/History of configurations)
- Auditability
  - AWS Cloudtrail
- Controllability
  - AWS KMS (Multi-tenant infrastructure)
  - AWS CloudHSM (Dedicated Infrastructure/FIPS 140-2 Compliant)
- Agility
  - AWS Cloudformation
  - AWS Elastic Beanstalk
- Automation
  - AWS OpsWorks
  - AWS CodeDeploy
- Scale
- Others
  - AWS IAM
  - AWS Trusted Advisor
  - AWS Cloudwatch

## Identity Access Management, S3 & Security Policies

### IAM Root Users
- Manage Your Security Credentials (When new root user)
  - Change Password
  - Remove old MFA and Re-configure it
  - Inactivate Root Access Keys (You don't really need it)
  - Check other user accounts
  
### S3 Bucket Policies

IAM policies have size limits of up to 2 kb for users, 5 kb for groups and 10 kb for roles).
S3 supports bucket policies of up to 20kb.

S3 => Permissions:
- Block public access
- Bucket policy
- Object ownership
- Access control list (ACL)
- CORS configuration

Only the specific IP address has access to this bucket:

```json
{
  "Version": "2012-10-17",
  "Id": "S3PolicyId1",
  "Statement": [
    {
      "Sid": "IPAllow",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::DOC-EXAMPLE-BUCKET", 
        "arn:aws:s3:::DOC-EXAMPLE-BUCKET/*"
      ],
      "Condition": {
        "NotIpAddress": {"aws:SourceIp": "54.240.143.0/24"}
      }
    }
  ]
}
```

Bucket policies are aplied to individual buckets
ACLs apply to individual objects
IAM is applied over a broad range of services

Exmaple: Deny all users ```S3:*```, Allow user-x ```S3:*```
- Explicit deny always result in a deny
  - On IAM Deny and on S3 Allow
  - On IAM Allow and on S3 Deny

### S3 ACLs

- Use S3 ACLs only if you need to apply policies on the Object Level 
  (Otherwise use IAM or Bucket Policies)
- Bucket policies are limited to 20 kb, you could consider ACL in this case. 
  (You can also use it for policies on the bucket level)

Apply object ACL only via CLI or API

Get user canonical ID:
```shell
aws s3api list-buckets
```

When an object is public with IAM deny access to that object:
- You are able to view the object through the link provided
- You are not able to open the object as an authenticated user

### Conflicting Policies

- No policies => DENY
- deny policy => DENY
- No deny and allow => ALLOW
- deny and allow => DENY

### Forcing Encryption on S3

```json
{
     "Version": "2012-10-17",
     "Id": "PutObjPolicy",
     "Statement": [
           {
                "Sid": "DenyIncorrectEncryptionHeader",
                "Effect": "Deny",
                "Principal": "*",
                "Action": "s3:PutObject",
                "Resource": "arn:aws:s3:::<bucket_name>/*",
                "Condition": {
                        "StringNotEquals": {
                               "s3:x-amz-server-side-encryption": "AES256"
                         }
                }
           },
           {
                "Sid": "DenyUnEncryptedObjectUploads",
                "Effect": "Deny",
                "Principal": "*",
                "Action": "s3:PutObject",
                "Resource": "arn:aws:s3:::<bucket_name>/*",
                "Condition": {
                        "Null": {
                               "s3:x-amz-server-side-encryption": true
                        }
               }
           }
     ]
 }

```

Enforce HTTPS

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
}
```

### Cross Region Replication

- By default this is done using SSL (you don't need a policy to enforce this)
- 1 to 1 replication (object cannot be replicated more than once)
- versioning must be enabled
- must be a bucket in another region
- Amazon S3 must have permissions to replicate objects
- If the source bucket owner also owns the object, the bucket owner has
full permissions to replicate the object, if not, the object owner must
grant the bucket owner the READ and READ_ACP permissions via object ACL.
  
Cross Region replication Cross Accounts:
- IAM role must have permissions to replicate object in the destination bucket.
- In the replication configuration, you can optionally direct S3 to change
the ownership of the object replica to the AWS account that owns the 
destination bucket.
- You could use cross region replication cross account for cloudtrail-logs

What will be replicated:
- Any new objects after replication is turned on
- unencrypted objects and encrypted will be replicated
- S3 replicates only objects in the source bucket for which the bucket owner
has permissions to read objects and read ACLs.
- Delete markers

What will not be replicated:
- objects before CCR is turned on
- SSE-C encrypted objects
- SSE-KMS server-side encrypted objects, unless enabled
- no permissions bucket owner objects
- Deletes to a particular version of an object.

### Forcing S3 to user Cloudfront

- Create public bucket
- Origin Domain name => S3 Bucket
- You can import certificate from the ACM (different certificate than for ALB)
  (us-east-1 region) or use IAM (IAM CLI)
- Make your object in S3 public
- Restrict Bucket Access (yes)
- Origin Access Identity:
  - Create new Identity (Cloudfront user)
- Grant Read Permissions on Bucket (yes, Update Bucket Policy) <= So cloudfront
can read the objects
  
### S3 Pre-signed URLs

- Use SDK or CLI
```shell
aws s3 presign s3://bucket/hello.txt --expires-in 300
```

### Security Token Service (STS) With Active Directory

- Federation (typically Active Directory)
  - SAML
- Federation With Mobile Apps
  - Facebook/Amazon/Google or other OpenID providers
- Cross Account Access

Terminology
- Federation (combining or joining users one domain with another domain => IAM with AD, Facebook, etc)
- Identity Broker (service that allows you to add identity from point A to (federate) point B)
- Identity Store (AD, Facebook, Google, etc)
- Identities (user of a service like facebook)

### Web Identity Federation

Amazon Cognito

Able to fetch temp credentials for AWS after authenticating through a web-base identity provider 
(Facebook, Google, Amazon)

### Cognito user pools

- User pools: are user directories to manager sign-in/up functionality for mobile and web applications.
 Users can sign-in directly to the User pool, or indirectly via identity provider (facebook).
  Cognito acts as an Identity Broker between IDP and AWS. After successful authentication JWT token.
- Identity Pools: enables you to create unique identities for your users and authenticate them with
IDPs. With an identity, you can obtain temporary, limited-privilege AWS credentials to access other
  AWS services.
  
### Glacier Vault Lock

You can configure and enforce compliance controls for individual Glacier Vaults, using a
Vault Lock Policy
- Similar to an IAM policy
- Configure WORM (write once read many) archives
- Create data retention policies e.g. 5 years retention

```json
{
     "Version":"2012-10-17",
     "Statement":[
      {
         "Sid": "deny-based-on-archive-age",
         "Principal": "*",
         "Effect": "Deny",
         "Action": "glacier:DeleteArchive",
         "Resource": [
            "arn:aws:glacier:us-west-2:123456789012:vaults/examplevault"
         ],
         "Condition": {
             "NumericLessThan" : {
                  "glacier:ArchiveAgeInDays" : "365"
             }
         }
      }
   ]
}
```

How:
- Initiate the lock by attaching a vault lock policy to your vault (will trigger in progress state)
- 24 hours to validate lock policy
- You can abort when the policy doesn't work
- Once validated, Lock policies are immutable (can not be changed!)

### IAM credential report

IAM => Credential report

```shell
aws iam generate-credential-report
```

```shell
aws iam get-crendetial-report --output text --query Content | base64 -D
```

- CSV format
  - user creation
  - Password enabled
  - MFA enabled
  - MFA devices
  - Access Keys
  - Last login
  - ...

## Logging And Monitoring

### Cloudtrail

What is logged:
- Metadata arround API calls
- The identity of the API caller
- The time of the API call
- The source IP address of the API caller
- The request parameters
- The response elements returned by the service

What is not logged:
- SSH, RDP

CloudTrail event logs:
- S3
- You manage the retention period
- Delivered every 5 minutes (up to 15 minute delay)
- Notifications available
- Can be aggregated across regions/accounts
- enabled by default for 7 days

CloudTrail Digest logs:
- You need to enable it
- You can use the logs to validate the cloudtrail logs (using hash validation SHA-256 (with RSA))

Secure Cloudtrail logs:
- IAM policies (IAM Group?)
- S3 Bucket Policies
- Prevent delete (Can be done using MFA for people who can)
- Encrypted by default

### Cloudwatch

- Cloudwatch
  - Real Time
  - Metrics
  - Alarms
  - Notifications
  - Custom Metrics
- Cloudwatch Logs
- Cloudwatch Events
  - Cloudtrail as event source (f.e. EC2 Creation)
  - Cloudwatch Events target (f.e. lambda)
  - Event Pattern/Schedule

### AWS Config

Assess, audit, and evaluate:
- Configurations of AWS resources
- Continuously monitors and records (in S3)
- Automate evaluation of recorded configurations against desired configurations
- In time change

Rules:
- AWS Managed Rules (build-in)
- Customer managed rules (invoke lambda)

Conformance Packs: Monitors and remediate (f.e Operational Best Practices for Amazon S3)

Can be aggregated for multiple regions/accounts (also through Organizations)

Requirements:
- Read only permissions to recorded resources for AWS Config
- Write Access to S3 logging bucket
- Publish access to SNS
- Give Admins Access to AWS Config

### Set up Alert If the Root user Logs In

- Ensure Cloudtrail is enabled
- Configure Cloudtrail to log to a Cloudwatch Loggroup
- In Cloudwatch (select loggroup)
  - Create a metric filter
  - Give a metric namespace (f.e. MyCloudtrailLogs)
    ```json
    { $.userIdentity.type = "Root" && $.userIdentity.invokedBy NOT EXISTS && $.eventType != "AwsServiceEvent" }
    ```
- Create an alarm
  - Set a notification (SNS) if the threshold is bigger than 0 (set it to your emailaddress)
  
### Inspector

- Install AWS Inspector agent on the EC2 instance
- Assessment setup
  - Network assessment (no agent required)
      - ports
      - ...
  - Host assessment
    - OS security configurations
    - outdated packages
- Run for an hour
- Download report

### Trusted Advisor

- Cost optimization (Basic, Business and Enterprise)
- Performance (Basic, Business and Enterprise)
- Security (Business and Enterprise)
- Fault tolerance (Business and Enterprise)

## Infrastructure Security

### KMS

- KMS uses shared HSM
- Regional service (not like IAM)
- Symetric & Assymetric keys

Secure your bucket even if objects are public:
- Create KMS key (symetric)
- Assign Key to User (or Role) as Key administrator and Key User
- Create s3 bucket
- Upload file
- Configure Encryption on File using KMS key
- When opening the public link (if object is public), you will get an InvalidArgument Exception
- Only the assigned user and administrator can view this object

Deleting keys takes minimal 7 days

#### CMK

- Create external key (symetric)
- Assign Key to User (or Role) as Key administrator and Key User
- Download Wrapping key choosing wrapping algorithm (24 hours available)
- Extract zipfile
  - README
  - ImportToken
  - WrappingKey
- upload files to s3
- Login EC2
```shell
aws s3 cp s3://bucket /home/ec2-user --recursive
openssl rand -out PlaintextKeyMeterial.bin 32
openssl rsautl -encrypt \
  -in PlaintextKeyMeterial.bin \
  -oaep \
  -inkey [wrappingkeyfile]
  -keyform DER \
  -pubin \
  -out EncryptedKeyMeterial.bin
aws s3 cp /home/ec2-user s3://bucket --recursive
```
- Download EncryptedKeyMeterial.bin
- In KMS
  - Upload Wrapped Key material (EncryptedKeyMeterial.bin)
  - Upload Import token

Extra   
- You cannot create the key twice with the same files!
- Also CMK cannot be exported!
- Not automatic rotation for CMK

### KMS Key Rotation Options

- Prevent re-use of keys
- rotate keys on regular basis
- key rotation depends on local laws, regulations and corporate policies
- method of rotation depends on type of key
- AWS Managed
- Customer managed
- Customer Managed with imported key material

AWS Managed Keys:
- Rotated automatically every 3 years
- CMK due for rotation, a new key is created and marked as the active key for all new requests
- The old key remains available to decrypt andy existing ciphertext files that wer encrypted
- AWS handles everything for you
- You cannot manage rotation yourself
- AWS Managed Keys cannot be deleted

CMK:
- Once a year rotate automatically (disabled by default)
- AWS KMS generates new cryptographic material for CMK every year.
- The CMK's old backing key is saved to decrypt previously encrypted files
- Rotate On-demand manually
- Create a new CMK, then change your applications or aliasses to use the new CMK
- You control the rotation frequency
- Keys can be deleted (be careful)

CMK with imported key material:
- Automatic rotation not available
- Rotation manually
- Create a new CMK, then change your applications or aliasses to use the new CMK
- You control the rotation frequency
- Keys can be deleted (be careful)

Create Encrypted Root volume: Create snapshot, copy snapshot (You cannot use your own KMS key to copy
to another region)

### Key Pairs

Get authorized key file on EC2:
```shell
curl http://169.254.169.254/latest/meta-data/public-keys/0/openssh-key/
```

Lost private ssh key => create new AMI from snapshot and attach new KeyPair

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
- Regional Service

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

### Dedicated Instances vs Dedicated Hosts

Dedicated instances may share hardware with other instances from the same AWS account that
are not Dedicated instances.

With Dedicated Hosts you can use your existing server-bound software licenses and address
corporate compliance and regulatory requirements. As you can keep spinning up your instances
over and over again on the same physical hardware. Als you will have additional visibility.

### AWS Hypervisors

Windows EC2 instances can only run on HVM.

### KMS Grants

Grants are an alternative access control mechanism to a Key Policy
- Programatically delegate the use of KMS CMKs to other AWS principles (f.e. user in own account
  or another account)
- Temprary, granular permissions(encrypt, decrypt, re-encrypt, describekey etc)
- Grants allow access, not deny
- Use Key Policies for relatively static permissions & for explicit deny

How?
- Grants are configured programatically using the AWS CLI
- ```create-grant``` add a grant to CMK, specifies who can use it and a list of operations 
the grantee can perform
- ```list-grants```
- ```revoke-grant```
- ```grant token``` is generated & can be passed as an argument to KMS API

```shell
#Create a new key and make a note of the region you are working in 
aws kms create-key

#Test encrypting plain text using my new key: 
aws kms encrypt --plaintext "hello" --key-id <key_arn>

#Create a new user called Dave and generate access key / secret access key
aws iam create-user --user-name dave
aws iam create-access-key --user-name dave

#Run aws configure using Dave's credentials creating a CLI profile for him
aws configure --profile dave
aws kms encrypt --plaintext "hello" --key-id <key_arn> --profile dave

#Create a grant for user called Dave
aws iam get-user --user-name dave
aws kms create-grant --key-id <key_arn> --grantee-principal <Dave's_arn> --operations "Encrypt"

#Encrypt plain text as user Dave: 
aws kms encrypt --plaintext "hello" --key-id <key_arn> --grant-tokens <grant_token_from_previous_command> --profile dave

#Revoke the grant:
aws kms list-grants --key-id <key_arn>
aws kms revoke-grant --key-id <key_arn> --grant-id <grant_id>

#Check that the revoke was successful:
aws kms encrypt --plaintext "hello" --key-id <key_arn> --profile dave
```

### KMS ViaService

Policy Conditions can be used to specify a condition within a Key Policy or IAM Policy.
The condition must be true for the policy statement to take effect.
- You might want a policy statement to take effect only after a specific date has passed.
- Allow or deny an action based on the requesting service.
- Predefined Condition Keys (most important one kms:ViaService).

### Cross Account Access

Enable cross account permissions:
- The Key Policy (In account where key is stored)
- IAM Policies (In other account)

### Container Security

- Don't store secrets (use Secrets manager)
- Don't store IAM credentials (use IAM roles)
- Don't run as root
- One service per container
- Use trusted images
- ECR enables you to scan your images
- Use ECS interface endpoint to prevent traffic send over the internet
- Use Encryption in Transit (TLS)
  - Store certificate in containers
  - User parameter store
  - Best option is ACM (Amazon Certificate Manager)

### VPC Flow logs

- Not a packet capturing tool
- source
- destination
- port numbers
- protocol
- number of packets
- number of bytes
- start/end time
- Attached to VPC, Subnets, or ENIs
- Flow logs capture ingress and egress traffic
  - log accepted
  - log rejected
  - log all traffic
- Not real time
- Logs in Cloudwatch or S3 (IAM role)
  - Cloudwatch (each flow logs = log group, each ENI = log stream)
  - S3 (use default or custom formatting for logging output)

#### Reading VPC Flow Logs

Important: Source/destination addresses will always be the internal primary private ip address
associated with the EMI

![Image of instance types](https://github.com/jeffvdv/Mynotes/blob/master/Security/assets/vpc-flow-logs.png)

Not captured in VPC Flow Logs:
- AWS DNS
- License Activation
- Metadata from 169.254.169.254
- Amazon Time Sync Service
- AWS DHCP and Reserved IP Addresses (VPC router)
- Traffic between Endpoint ENI and NLB ENI

### Session manager

- You can have your session logs send to S3 / encrypted by KMS
- You can have you sessions logs send to Cloudwatch

### CloudHSM


#### Initial Setup

- Create cluster in VPC (select subnets)
  - VPC cannot be changed after creation
  - SG are automatically created (add sg for your ec2 instances to access cluster)
- Initialize Cluster in private subnet(s) (will provision a private IP)
- Download CSR (Certificate signing request)
- Sign using commands
- Upload certificate (CustomerHsmCertificate.crt)
- Upload Issuing certificate (CustomerCa.crt)

#### Setup Commands
```shell
# Create private key
openssl genrsa -aes256 -out customerCA.key 2048
# Generate public certificate
openssl req -new -x509 -days 3652 -key customerCA.key -out customerCA.crt
# rename to clustername
nano <cluster_id>_ClusterCsr.csr
# Sign Certificate
openssl x509 -req -days 3652 -in <cluster_id>_ClusterCsr.csr \
                              -CA customerCA.crt \
                              -CAkey customerCA.key \
                              -CAcreateserial \
                              -out <cluster_id>_CustomerHsmCertificate.crt
wget https://s3.amazonaws.com/cloudhsmv2-software/CloudHsmClient/EL7/cloudhsm-client-latest.el7.x86_64.rpm
```

#### Installing/Configuring Client

```shell
sudo yum install -y ./cloudhsm-client-latest.el7.x86_64.rpm
cp customerCA.crt /opt/cloudhsm/etc/customerCA.crt
sudo /opt/cloudhsm/bin/configure -a <cluster_IP>
/opt/cloudhsm/bin/cloudhsm_mgmt_util /opt/cloudhsm/etc/cloudhsm_mgmt_util.cfg
enable_e2e
listUsers
```

#### User Management & Generating & Exporting Keys

4 Main user types:
- Precrypto Officer (PRECO)
  - Initial user (login => change password => automatically promoted to CO)
- Crypto Officer (CO)
  - Performs user management operations. (create/delete users, change passwords)
- Crypto User (CU)
  - Key Management (Create/delete/share/import/export cryptographic keys)
  - Cryptographic operations (Encrypt/Decrypt/Signing/Verifying, ...)
- Aplliance User (AU)
  - perform cloning and synchronization operation (Keep CloudHSM cluster in sync)

```shell
loginHSM PRECO admin password
changePswd PRECO admin <NewPassword>
listUsers
logoutHSM
loginHSM CO admin acloudguru
createUser CU ryan acloudguru
listUsers
logoutHSM
quit
sudo service cloudhsm-client start
/opt/cloudhsm/bin/key_mgmt_util
loginHSM -u CU -s ryan -p acloudguru
genSymKey -t 31 -s 32 -l aes256
genRSAKeyPair -m 2048 -e 65537 -l rsa2048
genSymKey -t 31 -s 16 -sess -l export-wrapping-key
exSymKey -k <symmetric_key> -out aes256.key.exp -w <wrapping_key>
exportPrivateKey -k <private_key> -out rsa2048.key.exp -w <wrapping_key>
exportPubKey -k 22 -out rsa2048.pub.exp
logoutHSM
exit
```

## Incident Response & AWS In the Real World

### DDOS Overview

- use Bastion hosts for SSH/RDP
- Scaling
- Cloudfront
  - Geo Restriction/Blocking
  - Origin Access Identity (Restrict access to S3 by enforcing cloudfront in front of S3)
- Route53
  - Alias Record sets (redirect to Cloudfront or different ALB, ...)
  - Private DNS
- WAF
  - AWS WAF service (ALBs, Cloudfront)
  - WAF from Marketplace
- Learn normal behavior
- Have an Attack plan
- AWS Shield
  - Elastic Loadbalancer, Amazon Cloudfront, Route53
  - Protects against SYN/UDP Floods, Reflection attacks, and other layer3/4 attacks
  - Advanced provides enhances protections (ELB, Cloudfront, Route35) => 3000$/month
    - near-realtime monitoring network traffic/active application monitoring (notifications DDoS)
    - 24x7 DDoS response team
    - Protects AWS bill against higher fees due to DDoS attack
  
### EC2 has been hacked
- Stop instance
- Take snapshot of the EBS volume
- Deploy instance in isolated environment (Isolated VPC in private subnet/VPC flow logs)
- Access the instance using a forensic workstation (Wireshark/Kali Linux)
- Read through the logs to figure out how (Windows Event Logs)

### Leaked keys on Github
- Go to IAM (or if root, go to My Security Credentials)
- Make Key Inactive
- Create new Access Keys

### Pent Testing - AWS Marketplace

- Prohibited services
  - DoS, DDoS, Port Flooding, Protocol Flooding, Request flooding (Login req flooding, ...),
  DNS zone walking via Route53
- Permitted
  - EC2, NAT gateways, ELBs, RDS, Cloudfront, Aurora, API Gateway, Lambda, Lightsail, Elastic Beanstalk
- Ask AWS for a Pen Testing

### AWS Certificate Manager
- Route53 to Registered domains (to create a domain)
- Go to Certificate Manager (N Virginia)
  - Import domains or specify Registered domains (DNS validation or e-mail)
  - Add CNAME in Route53 if DNS validation
- Amazon Certificates Automatically renewed, unless imported certificate or associated to Route53 hosted zones.
- Amazon Certificates cannot be exported

### Perfect Forward Secrecy (PFS) and ALBs

- With you ALB, you can select a "Security Policy for TLS"
  - Always include ELBSecurityPolicy-2016-08 (includes ECDHE-ECDSA-AES128-...)

### API Gateway
- API Gateway returns 429 Too Many Requests error when limit is exceeded (Throttling)
- Default 10.000 requests/sec and burst to 5.000 requests across all APIs within AWS account.
- You can increase this limit
- You can enable API Gateway Caching (300 seconds by default)

### AWS Systems Manager Parameter Store

- Store passwords, license keys, connectionstrings, ...
- Reference these values using their name
  - EC2
  - CloudFormation
  - Lambda
  - EC2 Run Command etc.
  
### AWS Systems Manager EC2 Run Command

- Attach instance profile => AmazonEc2RoleforSSM
- SSM agent needs to be installed on your EC2 instances

### Compliance in AWS

Compliance Frameworks
- PCI DSS (Secure Network and Systems/ Credit Card holder data protection)
- ISO
- HIPAA (Health Information U.S.)
- FedRAMP

## Updates based on Student feedback

### Macie

AI detecting sensitive data in S3

- Enable Macie - Service-linked-role for permissions (Cloudtrail & S3) 
- Create job
  - select s3 bucket
  - Scope (how frequently)
  - Sampling depth (1 -100%)
  - Custom data identifiers (own sensitive data)
  - Job name

### GuardDuty

Used ML:
- Unusual API calls, calls from a known malicious IP
- Attempts to disable CloudTrail logging
- Unauthorized deployments
- Compromised instances
- Reconnaissance by would be attackers
- Port scanning, failed logins

Monitors:
- Monitors Cloudtrail logs, VPC flow logs and DNS logs
- Receives feeds from Third Parties like Proofpoint, CrowdStrike and AWS Security (known malicious domains/Ips)
- Alerts appear in the GuardDuty console & Cloudwatch Events

Extra:
- Enable GuardDuty => Service-linked-role (access to EC2)
- You can Whitelist IP addresses/ Add Threatlist with own IP addresses
- You can Add multiple accounts to have one center for GuardDuty

### Secrets Manager


### Simple Email Service
- Standard SMTP interface or SES API
- Configure Security group associated with EC2 instances to allow communication with the SMTP endpoint
- Port 25 is the default (To avoid timeouts use either 587 or 2587)

### Security Hub
- Central Hub for Security Alerts
  - GuardDuty
  - Macie
  - Inspector
  - IAM Access Analyzer
  - Firewall Manager
  - 3rd party tools
  - Sends to CloudWatch Events
- Automated Checks
  - PCI-DSS (Payment Card Industry)
  - CIS (Center for Internet Security)
- Ongoing Security Audit For your Accounts

### Network Packet Inspection

Use 3rd party solution

### Active Directory Federation

ADFS

### AWS Artifact

Compliance Documents

### Troubleshooting

Troubleshooting CloudWatch:
- User does not see CW dashboard
  - Does IAM user have the correct permissions (allow read CW)
- EC2 instance unable to send CW logs
  - CW agent installed?
  - CW agent running?
  - Instance role write permissions cloudwatch attached to EC2?
- Unauthorized user creating EC2 instance (Trigger CloudTrail to CW events to lambda)
  - Check CW Events has permissions to invoke the event target (lambda)
  - Check Lambda has permissions to terminate EC2 - Execution Role

Troubleshooting Logging:
- CloudTrail logs not appearing in S3 Bucket when executing lambda and s3
  - Is Cloudtrail enabled?
  - Have you provided the right bucketname?
  - Is the S3 Bucket Policy Correct?
  - S3 and Lambda data events are high volume so they are not enabled by default
    - You have to enable this in CloudTrail
    - You can enable this only for s3 buckets and lambda's you specify
- Auditor not able to access Cloudtrail logs
  - Does Auditor account have read access to CloudTrail?
  - AWSCloudTrailReadOnlyAccess IAM Policy

Troubleshooting Authentication & Authorization
- STS:AssumeRoleWithWebIdentity (Web Identity Provider f.e facebook)
- STS:AssumeRoleWithSAML
- STS:AssumeRole

Troubleshooting Cross Account
- Production Account has s3 bucket. Dev Account want read access to S3 in Prod
  - Check In Dev that it has the STS AssumeRole to Assume Read-onlyaccess to s3 prod
  - Check that Dev Account is in the trusted entity of the prod account
  - Check that IAM role has permissions to Read the S3 Bucket
- Cross Account CMK

Troubleshooting lambda
- Lambda execution role (like instance profile)
- Function policy (Policy to invoke this lambda f.e. CW events trigger)

Troubleshooting CMKs in KMS
- IAM policy attached to f.e user (kms:ListKeys, kms:Encrypte, ...)
- In CMK, In Key Policy we have Key Admins, Key Users, and trusted external accounts

## Exam Readiness

https://www.aws.training/Details/eLearning?id=34786