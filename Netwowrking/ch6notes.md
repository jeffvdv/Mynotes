# Transitive Networking

## Inter-VPC connectivity

### VPC Sharing

VPC sharing allows a subnet in one AWS account to be shared with other AWS accounts:
- Accounts must be part of the same AWS Organization.
- Subnets are shared via AWS Resource Access Manager.
- Once shared, other accounts may provision resources within that subnet as if it were native to their account.
- Only the owner account has management control over that subnet.

### VPC Peering

- VPC peering connections do not allow transitive connectivity.
- VPC peering is not restricted by AWS account or region.
- Data transfer charges apply to inter-VPC traffic.
- Minimum required configuration allows broad inter-VPC connectivity. (all access to that vpc)

### VPC Endpoint Services

- Exposed an NLB-frontend application to slected consumers in the same region.
- Consumers connect to application by creating a VPC interface Endpoint.
- Access can be restricted by AWS account, IAM user, or IAM role.

### Transit VPCs (older way)

- Transit VPCs contain an EC2-hosted VPN solution
- EC2 VPN solution is the customer gateway for AWS VPN connections to spoke VPCs.
- VPN connections over the internet allow VPCs to be in any AWS account in any region.

#### Transit VPC Routing

[image Transit-VPC-routing.png]

- Dynamic routing is strongly recommended but is not a requirement.
- Spoke VPC VGWs advertise their local CIDR prefix to transit VPC VPN systems.
- VPN systems advertise all prefixes back to spoke VPCs.
- Spoke VPC VGWs route traffic to other spoke VPCs via transit VPCs VPN systems.
- Transitive connections are controlled by restricting the advertisement of route prefixes from the transit VPC.

## Transit VPCs and Hybrid Connectivity

### VPNs

- VPNs form on-prem networks should connect to the VPN system in the Transit VPC.
- Optionally, AWS VPN connections may be established with trusted spoke VPCs.
- AWS VPN connections to a VGW in the transit VPC cannot forward traffic to other VPCs.

### Direct Connect

- Public VIFs - Allows access to AWS public services without traversing the internet.
- Private VIFs 
    - Can connect to a single VGW to access attached VPC.
    - Can connect to a Direct Connect Gateway
    - Up to 10 VGWs in any public region may connect to a DX Gateway (Except China)

[image Floating-VGW-DX-connection.png]

- If Direct Connect traffic must pass through the transit VPC, then a detached VGW must be
created in the region the DX location connects to.
- Private VIF is associated with detached VGW.
- The VPN systems in the transit VPC connect with AWS VPN connections
- Must enabled dynamic routing.
- If spoke VPCs must be accessed directly, new private VIFs could be connected to their
VGWs directly.
- Direct Connect Gateway can be used to connect to multiple spoke VPCs
- DX Gateway cannot associate with floating VGW.

### Jumbo Frame Roundup

- Traffic with a larger MTU than the network can support will be fragmented.
- Enabling "Do Not Fragment" IP header flag will cause large traffic to be dropped instead.
- Default size: 1500 bytes
- Max supported MTU sizes:
    - VPC: 9001
    - DX Private VIF: 9001
    - DX Transit VIF: 8500
    - DX Public VIF: 1500
    - VPC Peering: 1500
    - Internet: 1500
    
## Transit Gateway Configuration

Within a single region, TGWs can attach to:
- VPCs from multiple AWS accounts
- VPN Customer Gateways
- Direct Connect Gateways (Transit VIF)
- can peer with TGWs in other regions.

### Create

- All properties are optional.
- Only tags may be modified after creation.
- Amazon side ASN must be private ASN (16- or 32 bit).
- Accounts can have up to 5 TGWs per region.

Default settings:
- ASN 64512
- Cross VPC public DNS name resolution enabled
- Equal Cost Multipath enabled for VPN attachments
- New attachments use -and prefixes are propagated to - the default route table
- Attachment requests from other AWS accounts must be manually approved.

### Attachments

- TGWs are connected to AWS network objects via attachments
- TGW attachments are AWS objects with their own AWS-assigned numbers (tgw-attach-xxxx)
- TGWs can have attachments to VPCs, VPN Customer Gateways, DX Gateways and peered TGWs.
- Traffic from an attached network arrives at the TGW from that network's attachment.
- Customers are billed hourly per operational TGW attachment.
- VPC:
  - Attachments to VPCs must connect to a subnet in at least 1 AZ.
- VPN:
  - VPN attachments require a customer gateway to be configured.
  - Creating a VPN attachment creates an AWS VPN connection.
  - Only the basic VPN tunnel options are available during creation.
  - After creation, subsequent management is performed from the site-to-site VPN connections console.
  - If static routing is selected, static routes are added via the TGW route tables
  - All appropriate AWS VPN connection charges will be applied.
- DX Gateway:
  - Attachments to a DX gateway are created from the Direct Connect Service.
  - DX connections to TGWs requires the creation of a Transit VIF.
  - The DX Gateway and TGW must be configured with different ASNs.
- TGWs:
  - TGWs in different regions may be joined by a Peering Connection attachment.
  - Requires static routes
  
### Route Tables:

- TGW route tables identify which attachment outbound traffic should be forwarded to.
- Each TGW begins with a single initial route table (default).
- Each TGW can support up to 20 route tables.
  
### Routes

- Routes in a TGW route table pair a destination CIDR with a TGW attachment ID that matching
traffic will be forwarded to.
- Traffic arriving from an attached network object will be resolved using a route from the
route table associated with that object's attachment.
- Traffic to destinations not represented int he route table will be dropped.

### Associations

- Each object attached to a TGW will be associated to a single route table.
- Incoming attachment traffic is forwarded according to the associated route table.
- A TGW route table may be associated with more than one attachment.

### Propagations

- Attachments may be configured to automatically propagate known CIDR ranges into one or 
more TGW route tables.
- Attachments do not need to be associated with a route table in order to propagate to it.
- VPCs will propagate their local CIDR.
- VPN CGWs and DXGWs will propagate customer prefixes advertised via BGP.
- Disabling TGW route propagation will require static route configuration.

### Static routes

- Static routes may be added to point select traffic towards a specific attachment.
- Only one "default" route per route table
- Blackhole routes drop traffic matching the CIDR.
- Required to forward traffic to a peered TGW.

### Propagations to attached networks

- Routes from VPCs to TGW attached networks must be manually added to VPC route tables.
- Routes form VPN CGWs to the TGW will be learned via BGP.
- Prefixes advertised via DXGWs are specifically declared when configuring the TGW attachment.

### Monitoring

- Cloudwatch metrics:
  - BytesIn/Out
  - PacketsIn/Out
  - PacketDropCountBlackhole
  - BytesDropCountBlackhole
  - BytesDropCountNoRoute
  - PacketDRopCountNoRoute
- Flow logs only within the attached VPCs
- Greater control or visibility into transitive traffic requires a transit VPC.

### Billing

- TGW Attachments billed per partial hour:
  - VPC owner (VPC attachment)
  - TGW owner (VPN attachment)
  - DX owner (DX attachment)
  - Both TGW owners (TGW peering)
- Charge per GB of data processed by TGW.
- No charge for data processed via peering.
- VPN connection charges

## Transit Gateway Routing

Propagations 

=> Identify which TGW attachments automatically add prefixes to that route table.

=> An attachment may propagate to many route tables

Associations 

=> Identify which TGW attachments use that route table to determine the destination
attachment for outbound traffic.

=> An attachment may only be associated with a single route table.

### Isolated VPCs

[image isolated-vpcs.png]

### Routing conflicts

1. Route with longest prefix
2. Static routes over propagated routes
3. VPC over DX over VPN

## Network size limits

### VPC Peering

- Only connects 2 specific VPCs
- A VPC may support up to 50 VPC Peering Connections
- Routes must be manually added to VPC route tables to support traffic flow.

### VPC Endpoint Services

- NLBs can handle around 55.000 simultaneous connections.

### AWS Site-to-Site VPN

- Default per-Region limits (soft limits):
  - 5 VGWs
  - 10 VPN connections per VGW
  - 50 Site-to-Site VPN connections and Customer Gateways

### Transit VPCs

Limited by:
- 3rd party VPN platform
- VPC route table limits:
  - 50 non-propagated (static)
  - 100 propagated
- 100 dynamic routes for VGWs

### DX Gateways

- Up to 30 Private VIFs
- Up to 10 VGWs

OR

- Up to 30 Transit VIFs
- Up to 3 TGWs

### Transit Gateways

- Can support up to 5000 attachments (max of 20 DX Gateway attachments)
- Up to 50 peering attachments.
- Up to 20 Route tables per TGW
- Up to 10.000 static routes per TGW



