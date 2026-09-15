 VLAN Design — MultiSite Enterprise OSPF

 1. Purpose

VLANs are used in this network to logically separate departments and reduce unnecessary Layer 2 broadcast traffic.

Each department receives its own VLAN and IP subnet. InterVLAN communication is handled by the site router using RouteronaStick.

The design contains 7 production VLANs and 1 security VLAN for unused ports.



 2. VLAN Overview

 Headquarters

| VLAN ID | Name       | Department             | Network       | Gateway    |
| : |  |  |  |  |
|      10 | HR         | Human Resources        | 10.10.10.0/24 | 10.10.10.1 |
|      20 | FINANCE    | Finance                | 10.10.20.0/24 | 10.10.20.1 |
|      30 | IT         | Information Technology | 10.10.30.0/24 | 10.10.30.1 |
|      40 | MANAGEMENT | Network Management     | 10.10.40.0/24 | 10.10.40.1 |

 Branch

| VLAN ID | Name  | Department             | Network       | Gateway    |
| : |  |  |  |  |
|      50 | HR    | Human Resources        | 10.20.50.0/24 | 10.20.50.1 |
|      60 | SALES | Sales                  | 10.20.60.0/24 | 10.20.60.1 |
|      70 | IT    | Information Technology | 10.20.70.0/24 | 10.20.70.1 |

 Security VLAN

| VLAN ID | Name         | Purpose                          |
| : |  |  |
|     999 | UNUSEDPORTS | Isolate unused switch interfaces |

VLAN 999 is not used for normal endpoint connectivity.



 3. Headquarters VLAN Architecture

The HQ network uses four VLANs.

 VLAN 10 — HR


Network:       10.10.10.0/24
Gateway:       10.10.10.1
DHCP:          R1HQ
Router:        G0/0.10


The HR endpoint is connected to an access port assigned to VLAN 10.

 VLAN 20 — Finance


Network:       10.10.20.0/24
Gateway:       10.10.20.1
DHCP:          R1HQ
Router:        G0/0.20


 VLAN 30 — IT


Network:       10.10.30.0/24
Gateway:       10.10.30.1
DHCP:          R1HQ
Router:        G0/0.30


 VLAN 40 — Management


Network:       10.10.40.0/24
Gateway:       10.10.40.1
Management SVI:
    SWHQCORE   10.10.40.2
    SWHQACCESS 10.10.40.3


VLAN 40 provides management connectivity for the HQ switches.



 4. Branch VLAN Architecture

The Branch network uses three VLANs.

 VLAN 50 — HR


Network:       10.20.50.0/24
Gateway:       10.20.50.1
DHCP:          R2BRANCH
Router:        G0/0.50


 VLAN 60 — Sales


Network:       10.20.60.0/24
Gateway:       10.20.60.1
DHCP:          R2BRANCH
Router:        G0/0.60


 VLAN 70 — IT


Network:       10.20.70.0/24
Gateway:       10.20.70.1
DHCP:          R2BRANCH
Router:        G0/0.70


The Branch switch is managed through VLAN 70:


SWBRANCH
Management IP: 10.20.70.2/24
Default Gateway: 10.20.70.1




 5. HQ Switch Port Assignments

 SWHQCORE

The core switch provides the main Layer 2 aggregation point for the HQ.

| Interface   | Connection        | Mode            | VLANs       |
|  |  |  |  |
| Gi0/0       | R1HQ G0/0        | Trunk           | 10,20,30,40 |
| Gi0/1       | SWHQACCESS G0/0 | Trunk           | 10,20,30,40 |
| Gi0/2–Gi3/3 | Unused            | Access/Shutdown | 999         |

The core switch therefore carries all HQ production VLANs between the router and access switch.



 6. HQ Access Switch Ports

 SWHQACCESS

| Interface   | Device/Purpose | Mode            |        VLAN |
|  |  |  | : |
| Gi0/0       | SWHQCORE     | Trunk           | 10,20,30,40 |
| Gi0/1       | HQHRPC       | Access          |          10 |
| Gi0/2       | HQFINPC      | Access          |          20 |
| Gi0/3       | HQITPC       | Access          |          30 |
| Gi1/0–Gi3/3 | Unused         | Access/Shutdown |         999 |

Enduser ports are configured as access ports so that endpoints cannot directly negotiate trunking.

PortFast is enabled on the active endpoint interfaces.



 7. Branch Switch Port Assignments

 SWBRANCH

| Interface   | Device/Purpose | Mode            |     VLAN |
|  |  |  | : |
| Gi0/0       | R2BRANCH G0/0 | Trunk           | 50,60,70 |
| Gi0/1       | BRHRPC       | Access          |       50 |
| Gi0/2       | BRSALESPC    | Access          |       60 |
| Gi0/3       | BRITPC       | Access          |       70 |
| Gi1/0–Gi3/3 | Unused         | Access/Shutdown |      999 |



 8. Trunk Configuration

Trunk links use 802.1Q VLAN tagging.

 HQ CoretoRouter


SWHQCORE Gi0/0
        |
        | 802.1Q
        |
R1HQ G0/0


Allowed VLANs:


10,20,30,40


 HQ CoretoAccess


SWHQCORE Gi0/1
        |
        | 802.1Q
        |
SWHQACCESS Gi0/0


Allowed VLANs:


10,20,30,40


 Branch RoutertoSwitch


R2BRANCH G0/0
        |
        | 802.1Q
        |
SWBRANCH Gi0/0


Allowed VLANs:


50,60,70


Restricting the allowed VLAN list prevents unrelated VLAN traffic from being carried across a trunk.



 9. Router Subinterfaces

RouteronaStick is used to provide a Layer 3 gateway for every VLAN.

 R1HQ


G0/0.10
    VLAN 10
    10.10.10.1/24

G0/0.20
    VLAN 20
    10.10.20.1/24

G0/0.30
    VLAN 30
    10.10.30.1/24

G0/0.40
    VLAN 40
    10.10.40.1/24


 R2BRANCH


G0/0.50
    VLAN 50
    10.20.50.1/24

G0/0.60
    VLAN 60
    10.20.60.1/24

G0/0.70
    VLAN 70
    10.20.70.1/24


Each router subinterface uses an IEEE 802.1Q VLAN tag.



 10. DHCP Address Reservation

The first 20 addresses of each production subnet are excluded from DHCP.

For example:


VLAN 10
10.10.10.1 – 10.10.10.20


are reserved.

DHCP clients therefore begin receiving addresses from:


10.10.10.21


The same reservation strategy is used across the other VLANs.

This provides space for:

 Default gateways
 Switch management addresses
 Infrastructure devices
 Servers
 Printers
 Future static assignments



 11. Management Addressing

Management IP addresses are assigned to switch SVIs.

 HQ


SWHQCORE
VLAN 40
10.10.40.2/24

SWHQACCESS
VLAN 40
10.10.40.3/24


 Branch


SWBRANCH
VLAN 70
10.20.70.2/24


The switches use their respective router gateway for Layer 3 management traffic.



 12. VLAN 999 — Unused Ports

VLAN 999 is a dedicated unusedport VLAN.

Unused physical interfaces are configured as:

cisco
switchport mode access
switchport access vlan 999
shutdown


This provides two layers of protection:

1. The port is placed into a VLAN that is not used for normal network traffic.
2. The interface is administratively disabled.

If an unused port is required in the future, it must be deliberately reconfigured before it can provide network access.



 13. VLAN Design Rationale

The VLAN structure provides clear separation between business functions.


HQ
├── VLAN 10  HR
├── VLAN 20  Finance
├── VLAN 30  IT
└── VLAN 40  Management

Branch
├── VLAN 50  HR
├── VLAN 60  Sales
└── VLAN 70  IT

Security
└── VLAN 999 Unused Ports


The separation makes the network easier to:

 Troubleshoot
 Monitor
 Manage
 Secure with ACLs/firewall policies
 Expand in the future



 14. Verification Commands

The following commands were used to verify VLAN and switching configuration:

cisco
show vlan brief
show interfaces trunk
show interfaces status


Router subinterfaces were verified with:

cisco
show ip interface brief


Connectivity was tested using VPCS ping commands across VLANs and between sites.



 15. Result

The VLAN implementation successfully provides logical departmental segmentation across the enterprise.

Each production VLAN has:

 A dedicated VLAN ID
 A dedicated IPv4 subnet
 A dedicated default gateway
 DHCP addressing
 OSPF advertisement through the site router

The implementation provides a structured Layer 2 foundation for the enterprise routing and security components of the project.
