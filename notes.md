# Monitoring & Reporting

## EC2
Default Cloudwatch metrics EC2:

- CPU
- Disk
- Network
- Status Check

Standard monitoring is 5 minutes

Detailed monitoring is minimum granularity of 1 minute

Create EC2 role => Access to Cloudwatch

Custom metrics (Memory)

```bash
sudo apt-get update
sudo apt-get install unzip
sudo apt-get install libwww-perl libdatetime-perl

curl https://aws-cloudwatch.s3.amazonaws.com/downloads/CloudWatchMonitoringScripts-1.2.2.zip -O

unzip CloudWatchMonitoringScripts-1.2.2.zip && \
rm CloudWatchMonitoringScripts-1.2.2.zip && \
cd aws-scripts-mon

/home/admin/aws-scripts-mon# ls
awscreds.template    LICENSE.txt		NOTICE.txt
AwsSignatureV4.pm    mon-get-instance-stats.pl
CloudWatchClient.pm  mon-put-instance-data.pl

echo "*/1 *   * * *   root    /home/admin/aws-scripts-mon/mon-put-instance-data.pl --mem-util --mem-used --mem-avail" >> /etc/crontab
```

Go to Cloudwatch => Metrics => Linux System => InstanceId

## EBS

### Compare Volume types

![Image of instance types](https://github.com/jeffvdv/Mynotes/blob/master/Screenshot%202020-06-28%20at%2020.09.45.png)

![Image of instance types](https://github.com/jeffvdv/Mynotes/blob/master/Screenshot%202020-06-28%20at%2020.13.58.png)

### IOPS

gp2:

- 3 IOPS/Gb
- Burst up to 3000 IOPS
- I/O credits
- Burst up to 2997 IOPS when using 1 Gb Volume
- max of 10 000 IOPS (more => use Provisioned IOPS)
- Burn all I/O credits if burst for 30 minutes

If you creat a volume from snapshot from s3 => use pre-warming ebs volume for maximum performance => 
What this basically means is just read every data block which has data on your ebs volume before using it.

### Metrics

| Metric                     | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
|----------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| VolumeReadBytes            | Provides information on the read operations in a specified period of time. The Sum statistic reports the total number of bytes transferred during the period. The Average statistic reports the average size of each read operation during the period, except on volumes attached to a Nitro-based instance, where the average represents the average over the specified period. The SampleCount statistic reports the total number of read operations during the period, except on volumes attached to a Nitro-based instance, where the sample count represents the number of data points used in the statistical calculation. For Xen instances, data is reported only when there is read activity on the volume.<br>The Minimum and Maximum statistics on this metric are supported only by volumes attached to Nitro-based instances.<br>Units: Bytes     |
| VolumeWriteBytes           | Provides information on the write operations in a specified period of time. The Sum statistic reports the total number of bytes transferred during the period. The Average statistic reports the average size of each write operation during the period, except on volumes attached to a Nitro-based instance, where the average represents the average over the specified period. The SampleCount statistic reports the total number of write operations during the period, except on volumes attached to a Nitro-based instance, where the sample count represents the number of data points used in the statistical calculation. For Xen instances, data is reported only when there is write activity on the volume.<br>The Minimum and Maximum statistics on this metric are supported only by volumes attached to Nitro-based instances.<br>Units: Bytes |
| VolumeReadOps !            | The total number of read operations in a specified period of time.<br>To calculate the average read operations per second (read IOPS) for the period, divide the total read operations in the period by the number of seconds in that period.<br>The Minimum and Maximum statistics on this metric are supported only by volumes attached to Nitro-based instances.<br>Units: Count                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| VolumeWriteOps !           | The total number of write operations in a specified period of time.<br>To calculate the average write operations per second (write IOPS) for the period, divide the total write operations in the period by the number of seconds in that period.<br>The Minimum and Maximum statistics on this metric are supported only by volumes attached to Nitro-based instances.<br>Units: Count                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| VolumeTotalReadTime        | Note<br>This metric is not supported with Multi-Attach enabled volumes.<br>The total number of seconds spent by all read operations that completed in a specified period of time. If multiple requests are submitted at the same time, this total could be greater than the length of the period. For example, for a period of 5 minutes (300 seconds): if 700 operations completed during that period, and each operation took 1 second, the value would be 700 seconds. For Xen instances, data is reported only when there is read activity on the volume.<br>The Average statistic on this metric is not relevant for volumes attached to Nitro-based instances.<br>The Minimum and Maximum statistics on this metric are supported only by volumes attached to Nitro-based instances.<br>Units: Seconds                                                   |
| VolumeTotalWriteTime       | Note<br>This metric is not supported with Multi-Attach enabled volumes.<br>The total number of seconds spent by all write operations that completed in a specified period of time. If multiple requests are submitted at the same time, this total could be greater than the length of the period. For example, for a period of 5 minutes (300 seconds): if 700 operations completed during that period, and each operation took 1 second, the value would be 700 seconds. For Xen instances, data is reported only when there is write activity on the volume.<br>The Average statistic on this metric is not relevant for volumes attached to Nitro-based instances.<br>The Minimum and Maximum statistics on this metric are supported only by volumes attached to Nitro-based instances.<br>Units: Seconds                                                 |
| VolumeIdleTime             | Note<br>This metric is not supported with Multi-Attach enabled volumes.<br>The total number of seconds in a specified period of time when no read or write operations were submitted.<br>The Average statistic on this metric is not relevant for volumes attached to Nitro-based instances.<br>The Minimum and Maximum statistics on this metric are supported only by volumes attached to Nitro-based instances.<br>Units: Seconds                                                                                                                                                                                                                                                                                                                                                                                                                           |
| VolumeQueueLength !        | The number of read and write operation requests waiting to be completed in a specified period of time.<br>The Sum statistic on this metric is not relevant for volumes attached to Nitro-based instances.<br>The Minimum and Maximum statistics on this metric are supported only by volumes attached to Nitro-based instances.<br>Units: Count                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| VolumeThroughputPercentage | Note<br>This metric is not supported with Multi-Attach enabled volumes.<br>Used with Provisioned IOPS SSD volumes only. The percentage of I/O operations per second (IOPS) delivered of the total IOPS provisioned for an Amazon EBS volume. Provisioned IOPS SSD volumes deliver their provisioned performance 99.9 percent of the time.<br>During a write, if there are no other pending I/O requests in a minute, the metric value will be 100 percent. Also, a volume's I/O performance may become degraded temporarily due to an action you have taken (for example, creating a snapshot of a volume during peak usage, running the volume on a non-EBS-optimized instance, or accessing data on the volume for the first time).<br>Units: Percent                                                                                                        |
| VolumeConsumedReadWriteOps | Used with Provisioned IOPS SSD volumes only. The total amount of read and write operations (normalized to 256K capacity units) consumed in a specified period of time.<br>I/O operations that are smaller than 256K each count as 1 consumed IOPS. I/O operations that are larger than 256K are counted in 256K capacity units. For example, a 1024K I/O would count as 4 consumed IOPS.<br>Units: Count                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| BurstBalance               | Used with General Purpose SSD (gp2), Throughput Optimized HDD (st1), and Cold HDD (sc1) volumes only. Provides information about the percentage of I/O credits (for gp2) or throughput credits (for st1 and sc1) remaining in the burst bucket. Data is reported to CloudWatch only when the volume is active. If the volume is not attached, no data is reported.<br>The Sum statistic on this metric is not relevant for volumes attached to Nitro-based instances.<br>If the baseline performance of the volume exceeds the maximum burst performance, credits are never spent. The reported burst balance is either 0% (Nitro-based instances) or 100% (non-Nitro-based instances). For more information, see I/O Credits and burst performance.<br>Units: Percent                                                                                         |

### Status

| Volume status     | I/O enabled status                                                                                                                       | I/O performance status (only available for Provisioned IOPS volumes)                                                             |
|-------------------|------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------|
| ok                | Enabled (I/O Enabled or I/O Auto-Enabled)                                                                                                | Normal (Volume performance is as expected)                                                                                       |
| warning           | Enabled (I/O Enabled or I/O Auto-Enabled)                                                                                                | Degraded (Volume performance is below expectations)<br>Severely Degraded (Volume performance is well below expectations)         |
| impaired          | Enabled (I/O Enabled or I/O Auto-Enabled)<br>Disabled (Volume is offline and pending recovery, or is waiting for the user to enable I/O) | Stalled (Volume performance is severely impacted)<br>Not Available (Unable to determine I/O performance because I/O is disabled) |
| insufficient-data | Enabled (I/O Enabled or I/O Auto-Enabled)<br>Insufficient Data                                                                           | Insufficient Data                                                                                                                |

### EBS modifications

You can adjust the volume type, size and IOPS on the fly.
If you adjust the volume size, you must manually expand the filesystem.

## ELB

Cloudwatch monitors performance (Metrics)

Cloudtrail monitors API calls to AWS (Audit)

Cloudwatch monitoring is enabled by default when creating a loadbalancer.

Access logs are not enabled by default (Stored in S3). You can use Sumologic or AWS Athena to query these logs.
Once EC2 instances have been deleted, there is no way to recover nginx access logs if you want to debug.

You can use Request tracing on an ALB. It adds or updates the X-Amzn-Trace-id header before sending it through.

## ElasticCache

Standard monitoring:

- CPU utilization
- Swap usage
- Evictions
- Concurrent Connections

### CPU
Memcached is multi-threaded and can handle loads of up to 90%. Add more nodes to the cluster when it exceeds 90%.

Redis is not multi-threaded. To determine the threshold in which to scale, take 90 and devide by the number of cores.

### Swap

If you use 4 Gb of RAM, use 4 Gb of Swapfile

Memcached should have arount 0 swap and should not exceed 50Mb, If it does increase hte memcached_connections_overhead parameter
(defines the amount of memory of reserved memcached connections and other miscellanous overhead).
Increase the memory of your memcached.

With Redis no SwapUsage metric is shown, instead it uses reserved-memory

### Evictions
Evictions => remove data when no new data can be stored.

Memcached => no recommended setting, either scale up or scale out.

Redis => only option the scale out by adding read replicas

### Concurrent Connections
No recommended settings. If there is a spike, this is due a spike in traffic or your application
is not releasing connections as it should (Set an alarm on the number of connections).

## Multiple Regions & Custom Dashboard

Dashboards are internationally! You have to change region if you want a widget with metrics from that region.

## Create a billing alarm

Cloudwatch => Create billing alarm => Select time check => Define threshold in USD => Select SNS create topic => Enter your e-mailadres
=> Confirm email-address.

## AWS Organizations

Allows you to manage multiple AWS accounts:

- Centrally manage policies across multiple accounts (IAM groups)
- Control access to AWS services (Service control policy => Allow or Deny AWS services)
- Automate AWS account creation and management
- Consolidate Billing across multiple AWS accounts (good for discounts)

Create an organization inside helpful tips, 2 choices:

- Enable all features
- Enable only consolidated billing

Policies:

- Deny (a list of denied services)
- Allow (a list of allowed services)

## AWS Rescource Groups & Tagging

Resource groups make it easy to group your resources using tags that are assigned to them.
You can group resources that share one or more tags.

Resource groups contain information such as:

- Region
- Name
- Health checks

Specific information:

- EC2: Public and private IP
- ELB: Port configurations
- RDS: Database engine etc

Create resource group based on tags or cloudformation stack based.
Add tags and give your group a name.

Via AWS System Manager your can execute automation on resource groups (f.e. Stop Ec2 instances).

## Cost explorer & Cost Allocation Tags

You can create csv reports.
In Cost allocation tags you can set the tags where you want to know the cost of in cost explorer.
It is across multiple accounts.

## AWS Config

Aws config is a fully managed service that provides you with an AWS resource inventory,
configuration history, and configuration change notifications to enable security and governance.

- Configurations snapshots and logs config changes of AWS resources
- Automated compliance checking

Enable it per region. AWS config stores everything in S3 and could trigger a lambda or pulled by Lambda.
As soon as a rule is being broken an SNS notification will be send to someone (email).

Terminology:

- Configuration Items: point in time attributes of resource
- Configuration Snapshots: Collection of Config Items
- Configuration Streams: Stream of changed Config Items
- Configuration History: Collection of Config Items for a resource over time
- Configuration Recorder: The configuration of Config that records and stores Config Items
    => Logs config for account in Region and Stores in S3, Notification through SNS

What we can see:

- Resource Type
- Resource ID
- Compliance
- Timeline:
    - Configuration Details
    - Relationships
    - Changes
    - Cloudtrail Events
    
Compliance checks:

- Trigger:
    - periodic
    - Configuration snapshot delivery
- Managed Rules:
    - About 40
    - Basic, but fundamental
    
AWS Config needs Read only permissions to the recorded resources, write access to S3 logging bucket
and publish access to SNS.

## Health Dashboards
(https://status.aws.amazon.com/)

Service Health Dashboard => Show the health of each AWS Service as a whole per region

Personal Health Dashboard => AWS Personal Health Dashbaord provides alerts and remediation guidance
when AWS is experiencing events that may impact you.

