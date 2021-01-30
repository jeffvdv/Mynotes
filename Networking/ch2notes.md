# Design and implement AWS networks

## VPC Flow logs

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
    
### Reading VPC Flow Logs

Important: Source/destination addresses will always be the internal primary private ip address
associated with the EMI

![Image of instance types](https://github.com/jeffvdv/Mynotes/blob/master/Networking/assets/vpc-flow-logs.png)

Not captured in VPC Flow Logs:
- AWS DNS
- License Activation
- Metadata from 169.254.169.254
- Amazon Time Sync Service
- AWS DHCP and Reserved IP Addresses (VPC router)
- Traffic between Endpoint ENI and NLB ENI

## Network performance

Options for High Compute Network performance:
- Instance types with enhanced networking
- Placement Groups (using clustering placement groups packs the instances close together)
  - Cluster <=
  - Partition
  - Spread
- Enabling Enhanced Networking (support 9001 MTU or jumbo frames)
