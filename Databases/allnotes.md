# AWS Certified Database - Specialty

## AWS Databases

- Relational
  - Best with structured data, Atomic operations (Transaction complete!)
  - Bad for semi-structured or sparse data
  - Use cases: Data warehousing, Finance, CRM, ...
- Key-Value
  - Extremely scalable, High throughput, low latency, flexible schema
  - Not great for analytics, Should knw access pattern
  - Use cases: Sessions, e-commerce
- Document
  - JSON-like documents, flexible schema, aggregate across documents
  - Not great for highly relational data
  - Use cases: Blogs, catalog information
- In-Memory
  - Microsecond latency, incredible fast compared to disk
  - Not great for persisted data to disk
  - Use cases: Sessions, Caching, Gaming-leader board, bidding applications
- Graphing
  - Easily relate sets of data, flexible schema
  - Not great for High volume of transactions
  - Use cases: Social networking friend requests, Fraud detection
- Time-series
  - Great for monitoring, aggregate across documents, great for time-ordered data
  - Not great for data that is not time ordered
- Ledger
  - Immutable and transparant, Highly scalable
  - Not great for Decentralized architectures
  - Use cases: Insurance claims, History finance debt application
  
OLTP => Online transaction processing (Fast queries, higher volume)
OLAP => Online analytics processing (Complex queries that aggragate data, queries are generally slower)

### Understanding RTO and RPO

RPO => Recovery Point Objective (The maximum amount of time or data we can lose without
impacting business operations)

RTO => Recovery Time Objective (The maximum amount of time the recovery process can take -
or, "How long can we be down?")

Disaster Recovery => RPO + RTO

### AWS Database Services Overview
- RDS
  - Engines:
    - MySql
    - Postgress
    - Oracle
    - SQL Server
    - MariaDB
  - Features:
    - Support Read Replicas (also for Data Recovery)
  - Automated Backups
  - Scalable Storage And Compute
  - Multi-AZ (Failover)
- Aurora
  - Amazon Aurora Engine (MySql and Postgress)
  - Scalable Reads (up to 15)
  - Automated Backup
  - Auto Scaling Cluster Volume (Cluster volumes scale in 10 GB increments)
  - Supports Multi-Master
- Redshift
  - Used for Data Warehousing
  - Supports Clustering
  - Columnar Type
- DynamoDB
  - Key-Value (NoSQL)
  - Real-Time Data Processing (track changes and take action based on those changes)
  - Features:
    - Scalable Read/Writes
    - DAX (caching)
    - Multi Region/Multi-Master
- Neptune
  - Graph
  - Supports Gremlin and SPARQL
  - Use Cases: Fraud detection, social networking, knowledge graphs, or even network security
- DocumentDB
  - NoSQL
  - Mongo
  - JSON documents
  - Use cases: Blogs/video, cataloging

![Image of Databases](https://github.com/jeffvdv/Mynotes/blob/master/Databases/assets/Databases_overview.png)







