# AWS Direct Connect and Hybrid DNS

When VPN is not a solution:
- Established connection only from on-prem to AWS
- VPN traffic used public infrastructure
- VPN's "out to internet" data transfer billing
- VGW is limited to maximum of 1.25 Gbps for all VPN connections

DX connections:
- always on in both directions
- Dedicated infrastructure
- Reduced rates for Data transfer
- Flat hourly port charge
- 10 Gbps max speed
- Multiple connections/simultaneously

Hybrid DNS:
- private DNS resolutions on-prem/AWS

## AWS Direct Connect Locations & Hardware
- Dedicated high throuput low latency connection to an AWS Region
- Access via globally places DX locations (Gov included)
    - Own hardware implementation in DX location
    - Work with DX partner
- Customer traffic is isolated using VLANs

Requirements:
- Single-mode fibre
- Either 1000BASE-LX (for 1 Gbps) or 10GBASE-LR (for 10 Gbps) transceivers
- Disable auto-negotiation on all ports used with DX
- 802.1Q VLAN encapsulation must be supported across entire connection
- Devices must support BGP and BGP MD5 authentication
- Bidirectional Forwarding Detection (BFD) is supported but not required

Customer responsible for connecting their on-prem to DX Location.
DX location Hardware depends using your own or partner

## DX Connections

### Dedicated Connections

- AWS hardware at DX Location connects directly to customer-managed hardware at DX location
- Support either 1 Gbps or 10 Gbps connection speeds to AWS Region
- Soft limit of 10 dedicated connections per Region, per account

How:
- Create new connection request using Console, CLI or API
- Information:
    - Name tag
    - DX location
    - Sub-location if applicable
    - Port speed
    - Direct Connect Partner if applicable
    - Optional additional tags
- New requests may take up to 72 hours for AWS to process
- Only tags and its values can be modified afterwards
- Letter of Authorization - Connecting Facility Assignment (LOA-CFA)
    - Authorizes DX location to connect your hardware to a specific port on AWS-owned hardware
    - Valid for 90 days
    - Contact AWS support if download link is not available after 72 hours
- Send LOA-CFA to DX location to initiat the process (send to AWS DX Partner if applicable)

### Hosted Connections

- AWS hardware at DX location connects directly to AWS DX Partner-managed hardware
- Wider range of bandwith options:
    - Mbps - 50 - 500
    - Gbps - 1 -10 (only supported by certain partners)

Once AWS Partner creates the connection, it must be accepted by the AWS customer.

## Virtual Interfaces

Private VIF:
- connects to VGW
- connects to a single Direct Connect Gateway (multiple regions)

Transit VIF:
- connecting DX to TGW

Public VIF:
- AWS public services (like S3)

L2 networks can be logically devided into VLAN's

VIF configuration:
- VIF type
- VIF name
- DX connection name
- VIF owner
- Gateway (private and transit VIFs)
- VLAN ID
- BGP ASN
- other BGP settings
- Jumbo MTU (private and transit VIFs)
- Tags

After creation:
- only tags may be edited afterwards
- Router configuration file may be downloaded

DX dedicated connections support:
- Up to 50 public or private VIFs
- Only 1 Transit VIF
- Hard limits

DX hosted connections only support a single VIF

A Hosted VIF:
- share that connection with other accounts
- allows a DX connection owned by one AWS account to be used by a different account.
- are created by the owner of the DX connection and offered to the other AWS account.
- the router configuration file can only be downloaded by the creator, not the consumer.

## Virtual LANs

VLANs needs to be configured at DX location and on-prem.

untagged/access ports => Ports belonging to a single VLAN

Traffic entering an untagged port may only be sent to another port that is a member of the same VLAN

tagged/trunk ports => ports belonging to more than 1 VLAN (used for communication between switches)

untagged ports:
- send traffic with standard ethernet frame
- knows the VLAN by the ingress port

tagged ports:
- traffic leaving a tagged port is using a 802.1Q ethernet frame
- contains 802.1Q Tag

Hosted DX connection only supports one VIF:
- establish multiple hosted DX connections
- Use aggregated VLANs!

### Aggregated/Nested VLANs

- On-prem and DX location devices are configured to recognize a specific "type" of VLAN tag
- Tagged traffic with configured type is handled normally (8100 default to most switches)
- Tagged traffic of any other type is treated as untagged traffic

At the provider incomming traffic from Customer VLAN:
- the 802.1Q Tag Frame is not recognized and is again encapsulated with an 802.1Q frame 
to forward traffic within the VLAN of the provider switches.

## Virtual Interfaces & BGP

### BGP Prefix Advertisements

#### Customer to AWS:

- VIFs have maximum number of prefixes that can be advertised:
  - Private VIFs - 100
  - Public VIFs - 1000
- Exceeding this limit will cuase the BGP sessions to go to the IDLE state

#### AWS to Customer:
- VGWs associated with private VIFs advertise all known routes.
- VGWs associated with DX gateways must specify allowed prefixes to be advertised.
  - Only CIDRs matching - or smaller than - listed prefixes will be advertised
  
- For public VIFs, AWS advertised prefixes for:
  - All public services in all public AWS Regions
  - Non-Region services such as Cloudfront and Route53
- Control outbound traffic to these prefixes at your on-prem router.
  - Filter outbound traffic with ACLs or firewalls
  - Filter learned prefixes using BGP communities
  
#### BGP communities:

- A means of labelling BGP prefixes
- BGP routers can be configured to handle incoming or outgoing prefixes based on their community values.
- Comprised of 16-bit ASN and a 16-bit, organization-defined number.

AWS automatically applies the following communities to prefixes advertised to public VIFs:
 
- 7224:8100 - routes to services from the same region as the DX connection.
- 7224:8200 - routes to services from the same continent as the DX connection.
- No value - routes to global services.

AWS advertised prefixes also include the “no_export” BGP community.

Customer to AWS:

- Customer prefix advertisement is controlled at public VIF
- Use BGP communities to control where AWS can propagate customer prefixes:
  - Local AWS Region - 7224:9100
  - All Regions in a continent - 7224:9200
  - All Public Regions - 7224:9300

## Link Aggregation Groups (LAGs)

What:

- A collection of multiple physical links combined into a single, logical link.
- Traffic sent to the LAG is distributed across all member links.
- Aggregates throughput of member links.
- Provides resiliency in the event of member link failure.

Requirements:

- All DX connections in a LAG must:
  - Use the same bandwidth.
  - Terminate at the same DX location.
- Maximum of four connections per LAG.
- Maximum of 10 LAGs per Region.

Creation:

- Use existing connections, request new connections, or a mix of both.
- You cannot create a LAG with new connections if you would exceed the overall connection limit for the Region.
- Adding existing connections to a LAG will temporarily interrupt connectivity.

Properties:
- LAG name
- Existing connections to use
- Number of new connections to request
- Minimum links
- Optional tags. 
- Minimum links identifies the minimum number of functional connections necessary for the entire link to be functional.
  - If the number of active links drops below the minimum, the entire LAG connection will become non-operational.
  - Default value is 0 (no minimum).
- New or existing connections may be added to existing LAGs.
- Connections may not be removed from LAGs if it crosses the minimum links threshold.

LAGs and VIFs:
- VIFs may be attached to a LAG instead of a single DX connection.
- VIFs may be attached to a LAG instead of a single DX connection.
- A corresponding customer LAG must be created at the on-prem hardware. (see diagram ACG)
- Add LAG primary port to VIF VLAN.

## Direct Connect Gateways

- Global services that facilitate DX connectivity to multiple AWS regions
- A bridge between Private or Transit VIFs and AWS networking objects
- No interaction with public VIFs
- Free to use

### Private VIFs and DX Gateways

- By itself, a private VIF can only connect to a single VGW
- DX gateways may connect a single private VIF with up to 10 VGWs in any public region
- Up to 30 private VIFs may connect to the same DX gateway

#### How to connect to different regions

- Create a private VIF
- Connect that private VIF to a DX gateway
- Associate DX gateway to with the VGWs that are attached to the VPC in the different regions.
- VPCs connecting through a DX gateway cannot have overlapping IP ranges
- Only sessions via a single VIF to one connected VPC at a time are allowed. (f.e. multiple VIFs talking to the same VPC)
- You cannot send traffic:
  - from one associated VPC to another
  - from one connected VIF to another
  - from a connected VIF through a VPN connection using an associated VGW

### DX Gateways and Transit VIFs

- DX gateways are also used when attaching DX connections to Transit Gateways
- Attach a Transit VIF to the DX gateway that is then attached to a Transit Gateway
- A DX Gateway may connect with either:
  - Private VIFs and VGWs
  - Transit VIFs and TGWs
  - but noth both !!!
  
### Other stuff to know about DX Gateways

- VGWs in one AWS account may request association with a DX gateway in a different account.
- DX gateways may only be associated with VGWs attached to a VPC
- Each VIF and VGW may only be associated with a single DX gateway
- A VGW can be simultaneously attached to both a single private VIF and a single DX gateway (connected to a different private VIF)
- DX Gateways may be associated to up to 3 TGWs

### Configure DX Gateway

- Name
- Amazon side ASN
- Attach to VIF
- Associate with VGW or TGW

## Well-architected Direct Connect

(see slides)

## Hybrid DNS

- Private DNS resolution across hybrid networks:
  - AWS resources are able to use on-prem DNS zones.
  - On-prem resources are able to use Route53 private zones or VPC DNS
  
### Route53 Resolver

- Provides default DNS resolution within VPCs
- "VPC + 2" IP address.
- Part of EC2 service hardware
  - Good availability and performance (cache within AZ)
  - Only accessible froom within AWS infrastructure
- Route 53 Private Zones must be associated to VPCs
- Resolver search sequence:
  - Route 53 Private Zones
  - VPC DNS domain
  - Public DNS

### The hybrid DNS Challenge

- EC2 instances always use Route 53 Resolver by default
- On-prem systems cannot reach Route 53 Resolver

### Customer-implemented DNS Resolvers

- EC2 hosted DNS resolvers are provisioned within VPC
- VPC configured to use EC2 resolver instead of Route 53
- Resolver forwards matching request to on-prem DNS
- On-prem DNS resolvers configured to forward matching request to EC2 resolver


- AWS Directory services Simple AD can provide the same functionality
- Configure on-prem DNS to forward to Simple AD DNS addresses.

#### Problems

- Single Ec2-ENI limited to 1024 queries/sec
- only 1 AZ, you would need HA
- Most DNS clients don't load-balance across multiple servers.
- DNS resolver services can loadbalance

### Route53 Resolver Endpoints

- Provides IP accessible endpoints to the AWS Route 53 Resolver service.
- Enpoints support 2 to 8 ENIs
- Each ENI supports up to 10.000 queries/second
- Endpoints are created within a single VPC, but may be used by other VPCs in the same Region
- Secured by a single VPC security group
  - Group assignment cannot be changed after creation.
- Each endpoint can handle either inbound or outbound DNS requests.
- Endpoint ENI's are AZ-scoped resources (use-multi-az)
- ENIs use either dynamic or customer-assinged IP addresses
- IP addresses are persistent for the lifetime of the endpoint
- Pricing per ENI, per hour

### Inbound Endpoints

- Handles request forwarding from on-prem to AWS DNS resolver
- Requests are sent to the IP address fo an ENI
- Request from on-prem DNS resolver instead of client for better performance
- Private hosted zones must be associated to the VPC where resolver endpoints reside

### Outbound Enpoints

- Handle forwarding for requests originating within AWS
- Can be associated with multiple VPCs in a Region
- Specify forwarding action for requests matching defined FQDN Patterns.
- Forward rules forward requests to IPv4 address of on-prem DNS
- System rules forward requests to Route 53 Resolver

#### Traffic rules

- System rules are automatically created for:
  - Private hosted zones
  - VPC domain names
  - Publicly reserved domain names
  
If rules conflict, resolver prefers:
  - Most specific FQDN
  - Forward rules over system rules






