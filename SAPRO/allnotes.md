# AWS Certified Solutions Architect - Professional

## Ch2: Data Stores

### Concepts

- Persistent Data Store (Durable)
  - Glacier
  - RDS
- Transient Data Store (Temporarily, passed along)
  - SQS
  - SNS
- Ephemeral Data Store (Data is lost when stopped)
  - EC2 Instance Store
  - Memcached
  

- IOPS => Measure of how fast we can read/write to a device
- Throughput => Measure how much data can be moved at the time


- ACID Consistency Model (relational databases)
  - Atomic: Transactions are "all or nothing"
  - Consistent: Transactions must be valid
  - Isolated: Transactions can't mess with one another
  - Durable: Completed transaction must stick around
