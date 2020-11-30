# Design and implement AWS Networks

## AWS Global Infrastructure

- private services are services contained within a vpc
- public services (accessible from the internet):
    - S3
    - Cloudwatch
    - Cloudformation
    - DynamoDB
    - Workspaces
- services outside of region:
    - Route53
    - Cloudfront Edge Locations
    - Aws Direct Connect Location
    - Other AWS regions
    
## VPC Basic Networking Design

Creating VPC:
- VPC Name Tag
- CIDR Block
- Dual Stack (optional IPv6 fixed /56 CIDR block)
- Tenancy:
    - default
    - dedicated (hardware)
    
IPv6 addresses is restricted by the number of IPv4 addresses. IPv6 is always public.

## Subnets, VPC Routers, and Route Tables

### Subnets

https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html#VPC_Sizing

### Route tables 

https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html

## Elastic Network Interface, Elastic IP, and Internet Gateway

### Elastic Network Interface

https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html#AvailableIpPerENI

- Inside a VPC
    - Primary IPv4 address (assigned via DHCP)
    - MAC address
    - At least one security group

### Internet Gateway and Dual-Homed instances

https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html

https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html#scenarios-enis

