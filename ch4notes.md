# Storage & Data Management

## S3

- Files can be from 0 Bytes to 5 TB
- Unlimited storage
- https://s3-eu-west-1.amazonaws.com/{yourname}

Objects consist of:

- Key
- Value
- Version ID (Important when versioning is enabled)
- Metadata (Data about data you are storing)
- Subresources (Bucket specific configuration)
    - Bucket policies, Access control lists
    - CORS
    - Transfer acceleration
    
Charged for:

- Storage per GB
- Requests (Get, Put, Copy, etc.)
- Storage Management pricing:
    - Inventory, Analytics, and Object tags
- Data Management pricing:
    - Data transferred out of S3
- Transfer Acceleration
    - Use Cloudfront to optimize

### Example Life Cycle policies

- Transition objects to IA storage class 90 days after you created them (f.e. logs)
- Archive objects to Glacier 1 year after creation
- Expire object 1 year after creating them (S3 will auto-delete)
    => Server access logging in s3 can accumulate many log files
    
## MFA Delete

- When versioning on a bucket is enabled, a delete action doesn't delete the object version,
but applies a delete marker.
- To permanently delete, provide object version id in the delete request

MFA delete provides and additional layer of protection to s3 Versioning.
Once enabled, MFA Delete will enforce 2 things:

- You will need a valid code form your MFA device in orderr to permanently delete an object version
- MFA also needed to suspend / reactivate versioning on an S3 Bucket

## S3 Encryption

Types of encryption:

- In Transit (SSL/TLS)
- At Rest:
    - Server Side Encryption
        - S3 Managed keys - SSE-S3 (master key encryption + rotate)
        - AWS Key Management Service, Managed Keys, SSE-KMS (envelope key)
        - Server Side Encryption with Customer Provided Keys - SSE-C (you own key and rotation)
    - Client Side Encryption (encrypt before upload)
    
Enforce Encryption:
- x-amz-server-side-encryption:AES256 (SSE-S3)
- x-amz-server-side-encryption:aws:kms (SSE-S3)

=> Use a bucket policy which denies PUT requests which doesn't include x-amz-server-side-encryption

The Expect: 100-continue Header in the PUT request is for accept of deny the body sent to S3

## EC2 Volume Types

- Instance store is know as ephemeral storage which is non-persistent
- EBS is Elastic Block storage which allows persistence

Root device can be EBS or Instance Store:

- Instance store root has maximum size of 10 Gb
- EBS can be up to 1 or 2 Tb
- All Instance store volumes are removed on termination of EC2
- Instance store persist with reboot of EC2

## Upgrading EC2 Volume Types

Create ec2 will create all ebs on it in the same AZ.
Change size and volume type on the fly.
Snapshots exists on S3 (not visible), there incremental.
Snapshots needs to be encrypted to be shared with other AWS accounts.

Migrating EBS to another AZ:
- create snapshot of EBS
- create volume from snapshot

Migrate to other region:
- Copy snapshot (select region)
- Create Image of ebs snapshot
- Launch EC2







