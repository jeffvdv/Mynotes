# Chapter 4: Hybrid Networking Basics and VPN's in AWS

## Virtual Private Gateway
Send traffic to the outside world:
- Internet Gateway
- Virtual Private Gateway <=
- Transit Gateway

What is Virtual Private Gateway:
- Acts as a router between you VPC and non-AWS-managed networks
- Can be associated with multiple external connections
- Can be attached only to one VPC at the time

2 types of connections:
- Site-to-Site VPN
- Direct Connect

Configuration:
- assign a name
- assign an ASN
- attach to a VPC
- once created, properties can't be modified

ASN:
- public ASN numbers => controlled by IANA
- private ASN numbers (64512 - 65534) (4200000000+)
- 16 bit and 32 bit
- default for VGW is 64512

## AWS Hybrid Route Learning
2 route learnings:
- static (manual configured)
- dynamic (routing protocols => BGP)

Site-to-Site VPN:
- static
- dynamic

Direct Connect:
- only dynamic

Route propagation:
- Can be enabled in route table propagation tab
- All routes learned by the VGW are shared with the route table

When networks overlap:
- Most specific route is usually preferred, but VPC local is always preferred over any overlapping propagated routes
- VPC static routes are preferred over matching propagated routes
- Direct connect > Static VPN > BGP VPN

## Border Gateway Protocol

- TCP port 179
- eBGP exterior
- iBGP interior (between 2 BGP routers within the same AS)
- BGP peering and network advertisement must be manually configured.
- Does not care how peering are physically connected

## BGP Prefixes and Preferences

BGP table:
- Network (which network it is directly connected to)
- Next Hop (router id or 0.0.0.0)
- Metric (default 0)
- LocPref (default 100)
- Weight (default 32768, weight 0 if learned)
- Path (default i <= internal route)

![Image of instance types](https://github.com/jeffvdv/Mynotes/blob/master/Networking/assets/BGP-prefixes-routing.png)

Selection best route:
1. Heighest weight
2. Highest Local Preference
3. Shortest AS Path
4. eBGP preferred over iBGP
5. Lowest Metric Value

- Only the best route is shared for a connection
- VGW automatically shares all it knows

### CloudHub
Enables VGW to automatically advertise learned BGP routes over all
connection supporting dynamic routing.

## BGP Prefix Preference Control
BGP cannot sense network quality

- heighest weight is only used on the local BGP router (not shared)
- heighest local preference is shared only to other iBGP routers with the same ASN 
(remember to change the weight to 0 on the other router)
- VGW cannot not be configured => How can we modify the traffic?
    - on the on premise router, configure the path you don't want to be preferred 
    - prepend-path your path with an additional entry of the same ASN (max 16 times)
        - This is parmenent! and is advertised to other possible connected BGP routers
    - An alternative to pathprepending, increase the metric of the path you don't prefer
      (only shared with peered BGP routers)

## VPN and IPSec Overview
2 VPNs:
- Site-to-Site VPN (tunnel)
- Client-to-Site VPN

- AWS only support IPSec VPN tunnels
- VPN endpoint systems on each network must be pre-configured:
    - Identity of other endpoint
    - Shared authentication method
        - pre-shared string
        - pre-shared certificate
        - PKI infrastructure using assymetric keys
    - Security policies
    - Kind of traffic:
        - Policy-based
        - Route-based

Sequence of events:
- "interesting" traffic is detected by the local endpoint
- Internet Key Exchange (IKE) phase 1 (main mode)
    - negotiate a security policy for key exchange
    - perform the key exchange (Diffie-Hellman)
    - Mutually encrypted authentication (test)
    - a single 2 way connection
- IKE phase 2 (Quick mode)
    - no re-authentication
    - generate of refresh keys (symetric keys)
    - 2 one way connection
- IPSec tunnel is established
- IPSec tunnel is terminated if no traffic is flowing anymore

Security Associations:
  - Policy-based:
    - Admin-configured rule sets define VPN-permitted traffic and security settings
    - One security association created per matched rule set

  - Route-based:
    - Traffic must target destination network to use VPN
    - Only a single security association is created for all traffic.

Site-to-Site VPN:
- The IPSec process is identical
- Tunnels are only established by traffic flowing from on-prem to AWS!
- AWS VPN tunnels can only support a single pair of IPSec security associations.
- only supports IPv4 and IPSec

## Customer Gateways
AWS Site-to-Site VPN components:
- Configure VGW or TGW
- Confirm CGD (Customer Gateway Device) meets requirements
- Configure CGW (Customer Gateway)
- VPN Connections
- Configure VPC Route tables
- Configure VPN settings on CGD (download configurations text file)

### Customer Gateway Device Requirements
- must support IKE (IKEv2) (Internet Key Exchange)
- must support IPSec
- must be accessible by a static public IPv4 address
- Must support Dead Peer Detection
- BGP support is optional
- Inbound/Outbound Firewalling:
    - UDP 500
    - IP Protocol 50
- You can use NAT-Traversal behind firewall
    - include UDP 4500

### Customer Gateway Configurations Parameters
- Name-tag value
- Dynamic or static routing (if Dynamic ASN number)
- CGD public ipaddress (or NAT-T if used)
- Optional Assign an ACM generated certificate for IKE authentication (used for phase 1)

### Configure VPN connection
- Name-tag value
- VGW ID
- CGW ID
- Dynamic or static routing
- Tunnel options as desired (or default)
    - Pre-shared keys (default 12 chars)
    - IP CIDR for both tunnels (must be /30 in the 169.254.0.0/16 CIDR block)
    - tunnel lifetime
    - key lifetime
    - Encryption mechanisms

- Routes are propagated by the VGW as soon as the VPN connection is established
- AWS creates 2 tunnel endpoints in different AZs per VPN connection (Active/Passive mode) 
  => Dead peer connection
- Both endpoints need to be configured at the CGD

### Configure VPN settings on CGD
- Download Configuration from VPN connection
- Select your Device software

## AWS VGW and VPN Limitations
1. VGW are not VPC transitive (VGW is attached to one vpc and does not know of other VPCs peered to 
this VPC) => solution: created another VGW for the other VPCs
2. VGW VPN throughput is capped to 1.25 Gbps (Attaching another won't fix this issue,
this will half the throuhput of the first and second connection)
3. VGW always used a single VPN tunnel endpoint when returning traffic to a network
4. Each AWS VPN IPSec tunnel only supports a single pair of one-way security associations
(Use a single policy matching all possible VPN traffic or use route-based VPN)
5. AWS S2S VPN only supports IPSec
6. AWS S2S VPN only supports IPv4
7. AWS S2S VPN cannot receive client-to-sit connections.

### Solutions
- All can be solved by using Software VPN on EC2
    - For issue 1. Transitive networking, the route table can direct traffic to other peered VPC
    using the route table it runs in.
- 1, 2, 3 can be solved using TGW

### Traffic Isolation within VPC
- point to point VPN connections between ec2
    - named overlay network
    - Provide encryption
    - multicasting
- By default only unicasting is supported (overlay network solves this issue)

### AWS Client VPN service
- managed openVPN service
- VPN endpoint for client to site connections
- Authenticate via Active Directory or private certificates (imported in ACM)
- Controlled access to VPC and anything connected to that VPC
- Client VPN Split tunneling can be configured so that only matching routes are send through the 
VPN (Default all traffic is send through the VPN)
- openvpn file shared with the client

## AWS VPN Monitoring and Optimization
Is it working as we expect?
- Cloudwatch metrics
    - TunnelState (1/0)
    - TunnelDataIn
    - TunnelDataOut
    - Dimensions: per VPN connection, per tunnel endpoint, or over all tunnels
    - For EC2 (standard metrics EC2), custom metric through cloudwatch agent
- Cloudwatch Logs
    - VPC flowlogs
    - EC2 - published log streams
    - AWS Client VPN authentication attempts (enable)

Can performance be improved?
- managed controlled by AWS
- EC2 software based (instance type - enhanced networking)

What can make it stop working?
- Misconfigurations:
    - Security groups
    - NACLs
    - Authentication
    - Customer Gateway Devices
    - OpenVPN client configuration
    - IAM insufficient permissions
- Hardware failure
    - Dead peer connections
    - Customer Gateway (Setup 2 Customer Gateway for high availability) => max throughput issue
    - EC2 software VPN (high availability)
- Client 2 Site VPN
    - Attach VPN endpoint to at least 2 subnets

## AWS VPN Cost Optimization
- AWS VPN Connection: 0.05/hour
- Customer Gateway: Cost? 
- Client VPN => VPN endpoint subnet association: 0.10/hour double for HA
- Client VPN connection time: 0.05/hour per client
- EC2 software VPN: EC2 cost HA
