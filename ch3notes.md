# High availability

## Elasticity (Short term) & Scalability (Long term)

EC2:

- Scalability: Increase instance size
- Elasticity: Increase number of instances

DynamoDB:

- Scalability: Unlimited amount of storage
- Elasticity: Increase additional IOPS for additional spikes in traffic, decrease when spike stops.

RDS:

- Scalability: Increase instance size
- Elasticity: not very elastic, can scale RDS base on demand

Aurora => Scalability: Modify instance type, Elasticity: Aurora Serverless

## RDS and Multi-AZ Failover

Automatically failover to other AZ (which already replicated) for Disaster Recovery.
Small outage of 1 minute can happen during failover. It update the private DNS for the database endpoint.

MySQL, MariaDB, Oracle and PostgreSQL engines utilize synchronous physical replication

SQL Server engine uses synchronous logical replication.

Advantages:

- Backups are taken from secondary which avoids I/O suspension to the primary
- Restore's are taken from secondary which avoids I/O suspension to the primary

! You can force a failover from one AZ to another by rebooting your instance. Via 
AWS console or by using RebootDBInstance API call.

## RDS & Using Read Replicas

Create a Read Replica via the AWS console or by CreateDBInstanceReadReplica API.
It will use engine native asynchronous replication.

Useful for Read-heavy databases to scale. Point your application to read endpoint of the
RDS instance. (Business reporting or data warehousing)

For Bi solutions => create read replica or import data into Redshift.

Aurora uses SSD backend virtualized storage layer for database workloads.
Aurora replicas share the same underlying storage as the source instance,
lowering the costs and avoiding to need to copy data to the replica nodes.

## Creating Read Replicas

- AWS takes a snapshot of the database
- If Multi-Az is not enabled, AWS will take the snapshot of your primary database.
This will create a brief I/O suspension for around 1 minute
- If Multi-Az is enabled, AWS will take the snapshot from your secondary database.

Read Replicas can be promoted to its own database. Will break the replication.

You can also create a read replica of a read replica in a other region. Can have latency.

You can have up to 5 Read replicas for Mysql, Postgresql and Mariadb

DB snapshots and Automated Backups cannot be taken of read replicas.

Key Metric to look for is REPLICA LAG (How long does the replication take)

## RDS & Using Read Replicas

- Storage autoscaling (start storage and threshold)
- Deletion protection
- You can not create a read replica if automated backups are turned off
- creating mutli-az could have potential downtime.
- You cannot create read replica of read replica when automated backups are turned off

```bash
aws rds describe-db-instances --region eu-west-1
```

## RDS - Encryption RDS Snaps

- Take a snap of existing RDS instance
- Copy the snap to the same/different region. (enable encyption)
- Encrypt the copy during the copy process.
- Restore the snap

You can share DB encrypted snapshot using AES-256 encryption with other AWS accounts:

- Create a Custom KMS encryption key
- create RDS snap using custom key
- share the custom AWS KMS encryption key
- Use AWS Management console, cli or rds api to share the encrypted snapshot.

You cant share the following:

- encrypted snapshot as public
- Oracle or Microsoft SQL Server snapshot that are encrypted using Transparent Data Encryption (TDE).
- Snapshots encrypted using default AWS KMS encryption key.

## Which Services have Maintenance windows

- RDS
- Elasticache
- Redshift
- DynamoDB DAX
- Neptune
- Amazon DocumentDB

## Elasticache

Redis has Master/Slave replication and Multi-AZ, Memcached doesn't

If your databases is stressed out by read operation, use ElastiCache or Redshift for OLAP transactions.

## Aurora

Supports:

- MySQL (5 times better performance)
- PostgreSQL (3 times better performance)

Specs:

- Starts with 10 GB storage and increments per 10 GB up to 64 TB
- Compute resources can scale up to 64 vCPUs and 488 GiB Memory
- 2 copies of your data in each AZ, with minimum of 3 AZ. 6 copies of you data.
- Can lose 2 copies of your data without effecting write availability
- Can lose 3 copies of your data without effecting read availability
- Self Healing are continuously scanned for errors
- Cluster volume across AZ's

Aurora Replica:

- Aurora replicas (up to 15)
- Mysql read replicas (up to 15)

100% of CPU utilization?

- Write causing issue: Scale up (increase instance size)
- Read causing issue: Scale out ( increase number of read replicas)

Aurora Serverless: Automatically scale up, shuts down base on capacity. You pay per second
basis for database capacity, and you can migrate between standard and serverless configuration
with a few clicks in the Amazon RDS Management Console.

Lowest number tier will be used for the failover (failover priority in configuration).

You can set up a read replica cross-region. Be aware to turn on Multi-AZ for this.
If the replication is disrupted, you have to set this up again.

Encryption at is rest is set by default.

## Troubleshooting Autoscaling

- Associated Key Pair does not exist
- Security group does not exist
- Autoscaling config is not working correctly
- Autoscaling group not found
- Instance type specified is not supported in the AZ
- AZ is no longer supported
- Invalid EBS device mapping
- Autoscaling service is not enabled on your account (check IAM)
- Attempting to attach an EBS block device to an instance store AMI

## Cloud Front & Cache Hit Ratios

- Edge location: location where the content will be cached
- Origin
- Distribution: name given to the CDN
- Web Distribution: For websites
- RTMP: Used for media streaming

Maximize cache hit ratios:

- Specify how long cloudfront caches object (Cache-Control max-age)
- Caching Based on Query String parameters
- Caching Based on Cookie Values
- Caching Based on Request Headers
- Remove Accept-Encoding Header when compression is not needed
- Serving Media content using http


