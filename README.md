[README.md](https://github.com/user-attachments/files/32244296/README.md)
Multi-Site Enterprise Network with OSPF









Project Overview



This project demonstrates the design, configuration, verification, troubleshooting, and hardening of a multi-site enterprise network using Cisco IOS routers, Layer 2 switches, VLANs, DHCP, and OSPF.



The environment represents a small enterprise with a Headquarters (HQ) and a Branch office connected through a routed WAN link.



The project was built and tested in GNS3.



Project Objectives



The main objectives of this project were to:



Design a realistic multi-site enterprise topology.

Segment departments using VLANs.

Implement inter-VLAN routing using router-on-a-stick.

Provide DHCP services for end-user VLANs.

Establish dynamic routing using OSPF.

Provide connectivity between HQ and Branch.

Configure infrastructure management interfaces.

Apply baseline network security hardening.

Test network behavior during a WAN failure.

Verify recovery after restoring the failed link.

Document the complete implementation for portfolio use.

Network Architecture



The network consists of two sites:



Headquarters



HQ contains:



R1-HQ

SW-HQ-CORE

SW-HQ-ACCESS

HR PC

Finance PC

IT PC

Branch



The Branch contains:



R2-BRANCH

SW-BRANCH

HR PC

Sales PC

IT PC

WAN



The two routers are connected using a /30 WAN network:



R1-HQ G0/1

10.255.0.1/30

&#x20;      |

&#x20;      |

10.255.0.2/30

R2-BRANCH G0/1

Topology

&#x20;                        OSPF WAN

&#x20;                  10.255.0.0/30

&#x20;                        |

&#x20;             ┌──────────┴──────────┐

&#x20;             │                     │

&#x20;          R1-HQ                R2-BRANCH

&#x20;             │                     │

&#x20;       802.1Q Trunk           802.1Q Trunk

&#x20;             │                     │

&#x20;      SW-HQ-CORE               SW-BRANCH

&#x20;             │                  │  │  │

&#x20;     SW-HQ-ACCESS              │  │  │

&#x20;        │   │   │              │  │  │

&#x20;       HR FIN  IT              HR Sales IT



The complete visual topology is available in:



topology/topology.png

Device Inventory

Device	Role

R1-HQ	HQ router, inter-VLAN routing, DHCP, OSPF

R2-BRANCH	Branch router, inter-VLAN routing, DHCP, OSPF

SW-HQ-CORE	HQ core Layer 2 switch

SW-HQ-ACCESS	HQ access Layer 2 switch

SW-BRANCH	Branch access Layer 2 switch

HQ-HR-PC	HQ HR endpoint

HQ-FIN-PC	HQ Finance endpoint

HQ-IT-PC	HQ IT endpoint

BR-HR-PC	Branch HR endpoint

BR-SALES-PC	Branch Sales endpoint

BR-IT-PC	Branch IT endpoint

IP Addressing

HQ Networks

VLAN	Department	Network	Gateway

10	HR	10.10.10.0/24	10.10.10.1

20	Finance	10.10.20.0/24	10.10.20.1

30	IT	10.10.30.0/24	10.10.30.1

40	Management	10.10.40.0/24	10.10.40.1

Branch Networks

VLAN	Department	Network	Gateway

50	HR	10.20.50.0/24	10.20.50.1

60	Sales	10.20.60.0/24	10.20.60.1

70	IT	10.20.70.0/24	10.20.70.1

WAN

Device	Interface	Address

R1-HQ	G0/1	10.255.0.1/30

R2-BRANCH	G0/1	10.255.0.2/30

VLAN Design

HQ

VLAN 10 → HR

VLAN 20 → Finance

VLAN 30 → IT

VLAN 40 → Management

Branch

VLAN 50 → HR

VLAN 60 → Sales

VLAN 70 → IT

Unused Ports



VLAN 999 was used for unused switch interfaces:



VLAN 999 → UNUSED-PORTS



Unused interfaces were assigned to this VLAN and administratively shut down.



More information is available in:



documentation/vlan-design.md

Inter-VLAN Routing



Router-on-a-stick was used at both sites.



HQ

G0/0.10 → 10.10.10.1

G0/0.20 → 10.10.20.1

G0/0.30 → 10.10.30.1

G0/0.40 → 10.10.40.1

Branch

G0/0.50 → 10.20.50.1

G0/0.60 → 10.20.60.1

G0/0.70 → 10.20.70.1



These subinterfaces provide the default gateway for each VLAN.



DHCP



DHCP was implemented locally at each site.



R1-HQ



Provides DHCP for:



VLAN 10

VLAN 20

VLAN 30

VLAN 40

R2-BRANCH



Provides DHCP for:



VLAN 50

VLAN 60

VLAN 70



The first 20 addresses of each subnet were excluded from DHCP allocation, leaving .21 onward for dynamic clients.



DNS provided to clients:



8.8.8.8



Detailed DHCP documentation:



documentation/dhcp-design.md

OSPF Routing



OSPF was selected as the dynamic routing protocol for communication between HQ and Branch.



Configuration highlights:



OSPF Process ID: 10

Area: 0

R1 Router ID: 1.1.1.1

R2 Router ID: 2.2.2.2



The WAN interfaces form the OSPF adjacency.



Expected state:



R1-HQ ↔ R2-BRANCH

&#x20;        FULL



OSPF advertises all HQ and Branch VLAN networks between the sites.



Detailed documentation:



documentation/ospf-design.md

Management



Switch management interfaces were configured using dedicated management networks.



HQ

SW-HQ-CORE   → 10.10.40.2

SW-HQ-ACCESS → 10.10.40.3

Gateway      → 10.10.40.1

Branch

SW-BRANCH → 10.20.70.2

Gateway   → 10.20.70.1

Security Hardening



Baseline security hardening included:



Unused ports assigned to VLAN 999.

Unused ports administratively shut down.

Unused router interfaces shut down.

HTTP management services disabled on switches.

DNS lookup disabled on infrastructure devices.

End-device interfaces explicitly configured as access ports.

PortFast Edge enabled on endpoint-facing ports.

OSPF passive interfaces configured for user VLANs.

Dedicated management VLANs used for switch management.



Detailed documentation:



documentation/security-hardening.md

Verification



The network was tested at multiple layers.



Layer 2



Verified:



VLAN membership

Trunk operation

Access-port assignments

Spanning-tree operation

Layer 3



Verified:



Interface status

Default gateways

Routing tables

Inter-VLAN connectivity

OSPF



Verified:



Neighbor relationship

FULL state

Remote OSPF routes

Route restoration after failure

DHCP



Verified:



DHCP pools

Client address assignment

Default gateway assignment

DNS assignment

End-to-End Connectivity



Connectivity was tested:



HQ → Branch

Branch → HQ

PC → Local Gateway

PC → Remote VLAN

PC → Switch Management IP

Failure Testing



A controlled WAN failure was introduced to test network behavior.



The WAN interface on R1-HQ was shut down:



interface GigabitEthernet0/1

&#x20;shutdown



The expected results occurred:



WAN interface → Down

OSPF adjacency → Lost

Remote OSPF routes → Removed

Inter-site connectivity → Lost



The WAN interface was then restored:



interface GigabitEthernet0/1

&#x20;no shutdown



The expected recovery occurred:



WAN interface → Up

OSPF adjacency → FULL

Remote routes → Restored

Inter-site connectivity → Restored



This test demonstrates OSPF failure detection and routing recovery.



It is not a redundant failover test, because the topology contains only one WAN path.



Detailed troubleshooting information:



documentation/troubleshooting.md

Project Structure

Multi-Site-Enterprise-OSPF/

│

├── README.md

│

├── topology/

│   └── topology.png

│

├── documentation/

│   ├── network-design.md

│   ├── vlan-design.md

│   ├── ospf-design.md

│   ├── dhcp-design.md

│   ├── security-hardening.md

│   └── troubleshooting.md

│

├── configurations/

│   ├── R1-HQ.txt

│   ├── R2-BRANCH.txt

│   ├── SW-HQ-CORE.txt

│   ├── SW-HQ-ACCESS.txt

│   └── SW-BRANCH.txt

│

└── screenshots/

&#x20;   ├── topology.png

&#x20;   ├── R1-verification.png

&#x20;   ├── R2-verification.png

&#x20;   ├── DHCP-HQ.png

&#x20;   ├── DHCP-BRANCH.png

&#x20;   ├── SW-HQ-CORE.png

&#x20;   ├── SW-HQ-ACCESS.png

&#x20;   ├── SW-BRANCH.png

&#x20;   ├── Security-hardening.png

&#x20;   ├── HQ-to-Branch.png

&#x20;   ├── Branch-to-HQ.png

&#x20;   ├── Failure-test.png

&#x20;   └── Recovery-test.png

Technologies Used

GNS3

Cisco IOSv

Cisco IOSvL2

VLANs

802.1Q Trunking

Router-on-a-Stick

DHCP

OSPF

IPv4

Spanning Tree Protocol

Basic Network Hardening

Key Skills Demonstrated



This project demonstrates practical skills in:



Enterprise network design.

Cisco router configuration.

Cisco Layer 2 switch configuration.

VLAN segmentation.

802.1Q trunking.

Inter-VLAN routing.

DHCP configuration.

Dynamic routing with OSPF.

Network troubleshooting.

Failure testing and recovery.

Infrastructure hardening.

Network documentation.

GNS3 network simulation.

Future Improvements



A larger production deployment could be expanded with:



Redundant WAN connections.

Multiple routers and switches.

Firewall integration.

ACL-based department security.

OSPF authentication.

AAA using RADIUS or TACACS+.

DHCP snooping and Dynamic ARP Inspection.

Network monitoring using SNMP and Syslog.

Centralized DNS and DHCP servers.

High-availability gateway protocols such as HSRP or VRRP.



These features were intentionally outside the scope of this project.



Project Result



The final lab successfully provides:



✓ Multi-site connectivity

✓ VLAN segmentation

✓ Inter-VLAN routing

✓ DHCP services

✓ OSPF dynamic routing

✓ Infrastructure management

✓ Baseline security hardening

✓ End-to-end connectivity

✓ WAN failure detection

✓ OSPF route recovery

✓ Complete technical documentation



The project demonstrates how a small enterprise network can be designed, implemented, validated, hardened, and documented using Cisco technologies in GNS3.



**Author**



**Muneeb Ur Rehman**



Networking Portfolio Project

GNS3 | Cisco IOS | OSPF | VLAN | DHCP

