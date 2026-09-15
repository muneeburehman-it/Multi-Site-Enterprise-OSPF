 Network Design — MultiSite Enterprise OSPF

 1. Project Overview

This project demonstrates the design and implementation of a multisite enterprise network using Cisco IOSv routers, Cisco IOSvL2 switches, VLAN segmentation, RouteronaStick interVLAN routing, DHCP, and OSPF dynamic routing.

The network consists of two locations:

 Headquarters (HQ)
 Branch Office

The design separates departments into dedicated VLANs and IP networks while using OSPF to provide dynamic routing between the two sites.

The lab was built and tested in GNS3 using Cisco IOSv/IOSvL2 devices and VPCS endpoints.



 2. Network Objectives

The primary objectives were to:

 Build a realistic multisite enterprise network.
 Separate departments using VLANs.
 Provide interVLAN routing.
 Provide automatic IP addressing using DHCP.
 Establish dynamic routing between HQ and Branch.
 Use OSPF Area 0 for intersite routing.
 Create dedicated management networks.
 Harden unused switch interfaces.
 Verify endtoend connectivity.
 Perform a controlled WAN failure and recovery test.
 Document configurations and verification results for a professional networking portfolio.



 3. Network Architecture

The network is divided into two sites.

 Headquarters

HQ contains:

 1 edge/router
 1 core switch
 1 access switch
 4 departmental VLANs
 3 VPCS endpoints

 Branch Office

The Branch contains:

 1 edge/router
 1 access switch
 3 departmental VLANs
 3 VPCS endpoints

The two routers are connected through a dedicated pointtopoint WAN network.

 Logical Architecture


                         ENTERPRISE WAN
                   10.255.0.0/30  OSPF Area 0
                              |
                 +++
                 |                         |
             R1HQ                      R2BRANCH
          10.255.0.1                  10.255.0.2
                 |                         |
          802.1Q Trunk                802.1Q Trunk
                 |                         |
          SWHQCORE                  SWBRANCH
                 |                         |
          802.1Q Trunk              VLAN 50/60/70
                 |
          SWHQACCESS
                 |
        +++
        |        |        |
       HR     Finance     IT
     VLAN 10  VLAN 20  VLAN 30

        Management VLAN 40




 4. Physical Topology

 Headquarters


R1HQ
  |
  | G0/0
  |
SWHQCORE
  |
  | G0/1
  |
SWHQACCESS
  |       |       |
 HR PC  FIN PC   IT PC


 Branch


R2BRANCH
    |
    | G0/0
    |
SWBRANCH
    |       |       |
 HR PC   SALES PC   IT PC


 InterSite WAN

R1HQ G0/1
10.255.0.1/30
      |
      |
R2BRANCH G0/1
10.255.0.2/30




 5. Device Inventory

| Device       | Type         | Location | Role                           |
|  |  |  |  |
| R1HQ        | Cisco IOSv   | HQ       | InterVLAN routing, DHCP, OSPF |
| R2BRANCH    | Cisco IOSv   | Branch   | InterVLAN routing, DHCP, OSPF |
| SWHQCORE   | Cisco IOSvL2 | HQ       | Core switching                 |
| SWHQACCESS | Cisco IOSvL2 | HQ       | Department access switching    |
| SWBRANCH    | Cisco IOSvL2 | Branch   | Branch access switching        |
| HQHRPC     | VPCS         | HQ       | HR endpoint                    |
| HQFINPC    | VPCS         | HQ       | Finance endpoint               |
| HQITPC     | VPCS         | HQ       | IT endpoint                    |
| BRHRPC     | VPCS         | Branch   | HR endpoint                    |
| BRSALESPC  | VPCS         | Branch   | Sales endpoint                 |
| BRITPC     | VPCS         | Branch   | IT endpoint                    |



 6. IP Addressing Strategy

Private IPv4 addressing was used throughout the enterprise network.

 Headquarters

| VLAN | Department | Network       | Default Gateway |
| : |  |  |  |
|   10 | HR         | 10.10.10.0/24 | 10.10.10.1      |
|   20 | Finance    | 10.10.20.0/24 | 10.10.20.1      |
|   30 | IT         | 10.10.30.0/24 | 10.10.30.1      |
|   40 | Management | 10.10.40.0/24 | 10.10.40.1      |

 Branch

| VLAN | Department | Network       | Default Gateway |
| : |  |  |  |
|   50 | HR         | 10.20.50.0/24 | 10.20.50.1      |
|   60 | Sales      | 10.20.60.0/24 | 10.20.60.1      |
|   70 | IT         | 10.20.70.0/24 | 10.20.70.1      |

 WAN

| Link              | Network       | Address        |
|  |  |  |
| R1HQ ↔ R2BRANCH | 10.255.0.0/30 | R1: 10.255.0.1 |
|                   |               | R2: 10.255.0.2 |

The /30 WAN network provides two usable addresses, making it appropriate for a pointtopoint router connection.



 7. VLAN Design

VLANs were used to logically separate departments.

 HQ VLANs

 VLAN 10 — HR
 VLAN 20 — Finance
 VLAN 30 — IT
 VLAN 40 — Management

 Branch VLANs

 VLAN 50 — HR
 VLAN 60 — Sales
 VLAN 70 — IT

A separate VLAN 999 was also created for unused switch interfaces.


VLAN 999 — UNUSEDPORTS


Unused physical interfaces were assigned to VLAN 999 and administratively shut down.

This prevents unused switch ports from remaining available for accidental or unauthorized network connections.



 8. InterVLAN Routing

InterVLAN routing is provided using RouteronaStick.

The router's physical LAN interface operates as an 802.1Q trunk.

Each VLAN has a router subinterface.

 HQ Example


G0/0.10 → VLAN 10 → 10.10.10.1/24
G0/0.20 → VLAN 20 → 10.10.20.1/24
G0/0.30 → VLAN 30 → 10.10.30.1/24
G0/0.40 → VLAN 40 → 10.10.40.1/24


 Branch Example


G0/0.50 → VLAN 50 → 10.20.50.1/24
G0/0.60 → VLAN 60 → 10.20.60.1/24
G0/0.70 → VLAN 70 → 10.20.70.1/24


This allows hosts in different VLANs to communicate through their respective default gateways.



 9. Switching Design

The HQ uses a twolayer switching structure:


R1HQ
  |
SWHQCORE
  |
SWHQACCESS


The connection between the core and access switches is an 802.1Q trunk carrying VLANs 10, 20, 30, and 40.

The connection between R1HQ and SWHQCORE is also an 802.1Q trunk.

At the Branch, the connection between R2BRANCH and SWBRANCH is an 802.1Q trunk carrying VLANs 50, 60, and 70.

Enduser interfaces are configured as access ports in their appropriate VLAN.



 10. Management Network

VLAN 40 is used as the HQ management network.

Management addresses:


SWHQCORE   10.10.40.2/24
SWHQACCESS 10.10.40.3/24
Gateway      10.10.40.1


The Branch switch uses VLAN 70 for management:


SWBRANCH    10.20.70.2/24
Gateway      10.20.70.1


Static default routes were configured on the Layer 2 switches to allow management traffic to reach remote networks.



 11. Dynamic Routing

OSPF was selected as the dynamic routing protocol.

 OSPF Configuration


OSPF Process ID: 10
Area: 0
R1 Router ID: 1.1.1.1
R2 Router ID: 2.2.2.2


The WAN link participates in OSPF and forms the adjacency between the two routers.

The VLAN interfaces are advertised into OSPF as passive interfaces.

This allows the networks to be advertised without attempting to form OSPF neighbor relationships with enduser VLANs.



 12. DHCP Design

DHCP services are provided directly by the routers.

 HQ DHCP

R1HQ provides DHCP for:

 VLAN 10 — HR
 VLAN 20 — Finance
 VLAN 30 — IT
 VLAN 40 — Management

 Branch DHCP

R2BRANCH provides DHCP for:

 VLAN 50 — HR
 VLAN 60 — Sales
 VLAN 70 — IT

The first 20 addresses of each subnet are excluded from DHCP to reserve them for gateways, infrastructure, servers, or future static assignments.

Example:


10.10.10.1 – 10.10.10.20


are excluded from the HQ HR DHCP pool.

Google DNS 8.8.8.8 is supplied to DHCP clients in this lab environment.



 13. Security Hardening

Several basic securityhardening measures were implemented.

 Unused Ports

Unused switch ports were:

1. Placed into VLAN 999.
2. Configured as access ports.
3. Administratively shut down.

Example:

cisco
switchport mode access
switchport access vlan 999
shutdown


 Management Services

HTTP and HTTPS management services were disabled on the switches because they were not required for this implementation.

 OSPF Passive Interfaces

User VLAN interfaces were configured as passive OSPF interfaces.

This prevents unnecessary OSPF neighbor formation on enduser networks.

 Unused Router Interfaces

Unused router interfaces were administratively shut down.



 14. Verification

The network was verified using Cisco IOS and VPCS commands.

Important verification commands included:

cisco
show ip interface brief
show ip ospf neighbor
show ip route ospf
show ip dhcp binding
show ip dhcp pool
show vlan brief
show interfaces trunk
show interfaces status


Endtoend connectivity was tested between HQ and Branch hosts.

Successful communication was verified between:


HQ HR → Branch HR
HQ HR → Branch Sales
HQ HR → Branch IT


and:


Branch HR → HQ HR
Branch HR → HQ Finance
Branch HR → HQ IT


Management connectivity was also tested between the HQ and Branch networks.



 15. Failure and Recovery Testing

A controlled WAN failure was performed by shutting down the R1HQ WAN interface:

cisco
interface GigabitEthernet0/1
 shutdown


The expected results occurred:

 OSPF adjacency was lost.
 The remote Branch routes were removed from the routing table.
 The Branch network became unreachable from R1.

The interface was then restored:

cisco
interface GigabitEthernet0/1
 no shutdown


OSPF successfully reestablished the adjacency.

The Branch routes returned to the routing table and connectivity was restored.

This test demonstrates OSPF failure detection, route withdrawal, adjacency recovery, and route reinstallation.

Because the lab has only one WAN path, this is a recovery/convergence test rather than redundant failover.



 16. Design Benefits

The implemented design provides:

 Departmental network segmentation
 Controlled broadcast domains
 Centralized interVLAN routing
 Automatic IP address assignment
 Dynamic intersite routing
 Dedicated management addressing
 Basic switchport hardening
 OSPFbased route convergence
 Structured troubleshooting and verification

The design can be expanded in the future with redundant WAN links, firewall integration, ACLs, sitetosite VPN, centralized DHCP/DNS, network monitoring, and authentication services.



 17. Future Improvements

For a production environment, the following improvements could be considered:

 Firewall between the enterprise and WAN/Internet
 Redundant core switches
 Redundant WAN links
 OSPF authentication
 Extended ACLs between departments
 DHCP server redundancy
 DNS infrastructure
 Centralized AAA using TACACS+ or RADIUS
 SNMP monitoring
 Syslog server
 Network time synchronization using NTP
 Sitetosite IPsec VPN
 Backup and configuration management
 Port security and BPDU Guard on access ports



 18. Project Result

The completed implementation successfully demonstrates a functional multisite enterprise network with:

7 VLANs + RouteronaStick + DHCP + OSPF + Management + Security Hardening + Failure/Recovery Testing.

The project was implemented, verified, and documented in GNS3 using Cisco IOSv and IOSvL2 devices.
