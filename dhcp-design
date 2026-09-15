 DHCP Design

 1. Purpose

Dynamic Host Configuration Protocol (DHCP) is used in this project to automatically provide IP addressing information to enduser devices at both the Headquarters (HQ) and Branch office.

The routers act as the DHCP servers for their respective sites. Each VLAN has a separate DHCP pool so that clients receive an address from the correct subnet.



 2. DHCP Architecture

The network uses distributed DHCP:

 R1HQ provides DHCP services for all HQ VLANs.
 R2BRANCH provides DHCP services for all Branch VLANs.
 Each VLAN has its own DHCP pool.
 Default gateways are the router subinterfaces.
 DNS is provided as 8.8.8.8.
 The first 20 addresses in each subnet are excluded from DHCP allocation.

This design keeps DHCP services local to each site and avoids unnecessary dependency on the WAN link.



 3. HQ DHCP Design

R1HQ provides DHCP for four VLANs.

| VLAN | Department | Network       | Gateway    | DHCP Pool     |
|  |  |  |  |  |
| 10   | HR         | 10.10.10.0/24 | 10.10.10.1 | HQHr         |
| 20   | Finance    | 10.10.20.0/24 | 10.10.20.1 | HQFINANCE    |
| 30   | IT         | 10.10.30.0/24 | 10.10.30.1 | HQIT         |
| 40   | Management | 10.10.40.0/24 | 10.10.40.1 | HQMANAGEMENT |

 Excluded addresses

The following addresses are reserved in every HQ subnet:


.1  .20


For example:


10.10.10.1  10.10.10.20
10.10.20.1  10.10.20.20
10.10.30.1  10.10.30.20
10.10.40.1  10.10.40.20


This leaves DHCP clients to receive addresses beginning from .21.

 Example HQ DHCP configuration

cisco
ip dhcp excludedaddress 10.10.10.1 10.10.10.20
ip dhcp excludedaddress 10.10.20.1 10.10.20.20
ip dhcp excludedaddress 10.10.30.1 10.10.30.20
ip dhcp excludedaddress 10.10.40.1 10.10.40.20

ip dhcp pool HQHr
 network 10.10.10.0 255.255.255.0
 defaultrouter 10.10.10.1
 dnsserver 8.8.8.8

ip dhcp pool HQFINANCE
 network 10.10.20.0 255.255.255.0
 defaultrouter 10.10.20.1
 dnsserver 8.8.8.8

ip dhcp pool HQIT
 network 10.10.30.0 255.255.255.0
 defaultrouter 10.10.30.1
 dnsserver 8.8.8.8

ip dhcp pool HQMANAGEMENT
 network 10.10.40.0 255.255.255.0
 defaultrouter 10.10.40.1
 dnsserver 8.8.8.8




 4. Branch DHCP Design

R2BRANCH provides DHCP for three VLANs.

| VLAN | Department | Network       | Gateway    | DHCP Pool    |
|  |  |  |  |  |
| 50   | HR         | 10.20.50.0/24 | 10.20.50.1 | BRANCHHR    |
| 60   | Sales      | 10.20.60.0/24 | 10.20.60.1 | BRANCHSALES |
| 70   | IT         | 10.20.70.0/24 | 10.20.70.1 | BRANCHIT    |

 Excluded addresses

The first 20 addresses in every Branch subnet are reserved:


10.20.50.1  10.20.50.20
10.20.60.1  10.20.60.20
10.20.70.1  10.20.70.20


DHCP clients can therefore begin receiving addresses from .21.

 Example Branch DHCP configuration

cisco
ip dhcp excludedaddress 10.20.50.1 10.20.50.20
ip dhcp excludedaddress 10.20.60.1 10.20.60.20
ip dhcp excludedaddress 10.20.70.1 10.20.70.20

ip dhcp pool BRANCHHR
 network 10.20.50.0 255.255.255.0
 defaultrouter 10.20.50.1
 dnsserver 8.8.8.8

ip dhcp pool BRANCHSALES
 network 10.20.60.0 255.255.255.0
 defaultrouter 10.20.60.1
 dnsserver 8.8.8.8

ip dhcp pool BRANCHIT
 network 10.20.70.0 255.255.255.0
 defaultrouter 10.20.70.1
 dnsserver 8.8.8.8




 5. DHCP Address Allocation

The expected allocation pattern is:


Gateway       Reserved          DHCP Clients
   .1         .1  .20          .21  .254


Example:


HQHRPC      → 10.10.10.21
HQFINPC     → 10.10.20.21
HQITPC      → 10.10.30.21

BRHRPC      → 10.20.50.21
BRSALESPC   → 10.20.60.21
BRITPC      → 10.20.70.21


Actual DHCP leases can change when leases expire or clients renew their addresses, so the binding table should be treated as dynamic rather than permanently fixed.



 6. DHCP and RouteronaStick

DHCP works together with the routeronastick configuration.

At HQ, R1HQ uses subinterfaces:


G0/0.10 → 10.10.10.1
G0/0.20 → 10.10.20.1
G0/0.30 → 10.10.30.1
G0/0.40 → 10.10.40.1


At Branch, R2BRANCH uses:


G0/0.50 → 10.20.50.1
G0/0.60 → 10.20.60.1
G0/0.70 → 10.20.70.1


Each subinterface acts as the default gateway for its VLAN and provides the correct Layer 3 boundary for DHCP clients.



 7. DHCP Process

When a VPCS starts with DHCP enabled, it follows the normal DHCP process:


Client
  |
  | DHCP Discover
  v
Router DHCP Server
  |
  | DHCP Offer
  v
Client
  |
  | DHCP Request
  v
Router DHCP Server
  |
  | DHCP ACK
  v
Client receives:
IP Address
Subnet Mask
Default Gateway
DNS Server


Because the DHCP server is the local router for each VLAN, no DHCP relay configuration is required in this design.



 8. DHCP Verification

DHCP configuration can be verified on the routers using:

cisco
show ip dhcp pool


This displays DHCP pool information and utilization.

To view active DHCP bindings:

cisco
show ip dhcp binding


To review DHCPrelated configuration:

cisco
show runningconfig | section dhcp


To troubleshoot DHCP activity:

cisco
debug ip dhcp server events


Debugging should only be enabled temporarily because it can generate a large amount of console output.



 9. Client Verification

On each VPCS, DHCP can be requested using:


ip dhcp


The client can then verify its address with:


show ip


A successful result should contain:


IP address
Subnet mask
Default gateway


For example, an HQ HR client should receive an address from:


10.10.10.0/24


and use:


10.10.10.1


as its default gateway.



 10. DHCP Testing Performed

DHCP was tested from both sites.

 HQ

The following clients successfully obtained DHCP configuration:


HQHRPC
HQFINPC
HQITPC


 Branch

The following clients successfully obtained DHCP configuration:


BRHRPC
BRSALESPC
BRITPC


The clients were then able to communicate with their local gateways and remote networks across the OSPF WAN connection.



 11. Design Advantages

This DHCP design provides several practical benefits:

 Centralized control per site

Each router manages the address pools for its own site.

 VLAN separation

Every VLAN has its own independent subnet and DHCP pool.

 Reduced manual configuration

End devices do not require manually assigned IP addresses.

 Reserved infrastructure addresses

The first 20 addresses are kept available for gateways, management interfaces, servers, printers, or future static devices.

 WAN independence

Local clients can obtain DHCP addresses without depending on the remote site.



 12. Future Improvements

A production deployment could improve this design further by adding:

 DHCP redundancy using dedicated DHCP servers.
 DHCP reservations for important endpoints.
 Centralized DNS services.
 IP address management (IPAM).
 DHCP snooping on switches.
 DHCP failover between redundant servers.
 Monitoring and alerting for DHCP pool utilization.

These features were not required for this lab but would be appropriate in a larger enterprise environment.



 13. Final Result

The completed project successfully implements DHCP across both enterprise sites.


                    ENTERPRISE DHCP
                          |
            ┌─────────────┴─────────────┐
            │                           │
         R1HQ                       R2BRANCH
            │                           │
     ┌──────┼──────┬──────┐       ┌────┼────┐
     │      │      │      │       │    │    │
    HR   Finance   IT   Mgmt      HR  Sales  IT
   VLAN10 VLAN20 VLAN30 VLAN40  VLAN50 VLAN60 VLAN70


Each VLAN receives addressing from the correct local DHCP pool, uses its router subinterface as the default gateway, and can communicate with remote networks through the OSPFbased WAN.

DHCP implementation: Completed and verified.
