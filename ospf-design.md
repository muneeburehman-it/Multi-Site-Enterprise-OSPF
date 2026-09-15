 OSPF Design — MultiSite Enterprise Network

 1. Overview

Open Shortest Path First (OSPF) was implemented as the dynamic routing protocol between the Headquarters (HQ) and Branch Office.

The purpose of OSPF in this project is to allow the two sites to automatically exchange their internal VLAN networks without requiring manually configured routes on the routers.

The implementation uses:

 OSPF process ID: 10
 OSPF area: Area 0
 R1HQ router ID: 1.1.1.1
 R2BRANCH router ID: 2.2.2.2
 Pointtopoint WAN network: 10.255.0.0/30



 2. OSPF Topology

The routing relationship is established over the dedicated WAN link:


              OSPF AREA 0
        10.255.0.0/30 WAN LINK

       10.255.0.1       10.255.0.2
       R1HQ  =================  R2BRANCH
       ID 1.1.1.1              ID 2.2.2.2


R1HQ advertises the HQ VLAN networks.

R2BRANCH advertises the Branch VLAN networks.



 3. WAN Addressing

The routers use a /30 point to point network.

| Device    | Interface | IP Address    | Network       |
|  |  |  |  |
| R1HQ     | G0/1      | 10.255.0.1/30 | 10.255.0.0/30 |
| R2BRANCH | G0/1      | 10.255.0.2/30 | 10.255.0.0/30 |

The /30 subnet provides exactly two usable addresses, which is appropriate for a point to point router link.



 4. Router IDs

Explicit router IDs were configured to provide stable OSPF identities.

 R1HQ

cisco
router ospf 10
 routerid 1.1.1.1


 R2BRANCH

cisco
router ospf 10
 router-id 2.2.2.2


Using explicit router IDs makes the OSPF topology easier to identify and troubleshoot.



 5. HQ OSPF Networks

R1HQ advertises the following networks:


10.10.10.0/24    HQ HR
10.10.20.0/24    HQ Finance
10.10.30.0/24    HQ IT
10.10.40.0/24    HQ Management
10.255.0.0/30    WAN


The corresponding OSPF configuration is:

cisco
router ospf 10
 router-id 1.1.1.1
 network 10.10.10.0 0.0.0.255 area 0
 network 10.10.20.0 0.0.0.255 area 0
 network 10.10.30.0 0.0.0.255 area 0
 network 10.10.40.0 0.0.0.255 area 0
 network 10.255.0.0 0.0.0.3 area 0




 6. Branch OSPF Networks

R2BRANCH advertises:


10.20.50.0/24    Branch HR
10.20.60.0/24    Branch Sales
10.20.70.0/24    Branch IT
10.255.0.0/30    WAN


Configuration:

cisco
router ospf 10
 router-id 2.2.2.2
 network 10.20.50.0 0.0.0.255 area 0
 network 10.20.60.0 0.0.0.255 area 0
 network 10.20.70.0 0.0.0.255 area 0
 network 10.255.0.0 0.0.0.3 area 0




 7. Wildcard Masks

OSPF uses wildcard masks with the network command.

For a /24 network:


Subnet mask:   255.255.255.0
Wildcard:      0.0.0.255


For the /30 WAN network:


Subnet mask:   255.255.255.252
Wildcard:      0.0.0.3


Therefore:

cisco
network 10.10.10.0 0.0.0.255 area 0


matches the entire HQ HR subnet.

And:

cisco
network 10.255.0.0 0.0.0.3 area 0


matches the WAN subnet.



 8. Passive Interfaces

The user facing VLAN interfaces were configured as passive OSPF interfaces.

 R1HQ

cisco
passive interface GigabitEthernet0/0.10
passive interface GigabitEthernet0/0.20
passive interface GigabitEthernet0/0.30
passive interface GigabitEthernet0/0.40


 R2BRANCH

cisco
passive interface GigabitEthernet0/0.50
passive interface GigabitEthernet0/0.60
passive interface GigabitEthernet0/0.70


The WAN interfaces remain active for OSPF neighbor formation.

 Why use passive interfaces?

The VLANs contain end-user devices, not OSPF routers.

Making them passive prevents OSPF Hello packets from being sent toward normal endpoint networks while still allowing their connected networks to be advertised into OSPF.

This is a simple and effective routing security practice.



 9. OSPF Neighbor Relationship

After configuration, R1HQ and R2BRANCH successfully formed an OSPF adjacency.

R1HQ identifies R2BRANCH using:


Router ID: 2.2.2.2
Address:   10.255.0.2
State:     FULL
Interface: Gi0/1


R2BRANCH identifies R1HQ using:


Router ID: 1.1.1.1
Address:   10.255.0.1
State:     FULL
Interface: Gi0/1


The FULL state confirms that the OSPF routers have successfully synchronized their link state databases.



 10. Route Advertisement

After OSPF convergence, each router learns the remote site's VLAN networks.

 R1HQ learns Branch routes

R1 receives:


10.20.50.0/24
10.20.60.0/24
10.20.70.0/24


through R2BRANCH.

The next hop is:


10.255.0.2


 R2BRANCH learns HQ routes

R2 receives:


10.10.10.0/24
10.10.20.0/24
10.10.30.0/24
10.10.40.0/24


through R1HQ.

The next hop is:


10.255.0.1


This allows endpoints at either site to communicate with remote VLANs.



 11. OSPF Route Selection

OSPF calculates the best path using its cost metric.

In this two router topology, the remote networks are reached through the single WAN connection.

For example:


HQ HR
10.10.10.0/24
       |
      R1
       |
10.255.0.0/30
       |
      R2
       |
Branch HR
10.20.50.0/24


R1 dynamically determines that the path to 10.20.50.0/24 is through R2.

No static route is required for the Branch VLAN networks.



 12. OSPF Verification Commands

The following commands were used to verify OSPF operation.

 Neighbor Verification

cisco
show ip ospf neighbor 


This verifies whether the OSPF adjacency is established.

 OSPF Routes

cisco
show ip route ospf


This displays routes learned through OSPF.

 Interface Status

cisco
show ip interface brief


This confirms that the router interfaces and sub interfaces are operational.

 Detailed OSPF Information

cisco
show ip ospf


This can be used to inspect the OSPF process, router ID, areas, and general protocol information.



 13. Failure Testing

A controlled WAN failure was performed on R1HQ.

The WAN interface was administratively disabled:

cisco
configure terminal
interface GigabitEthernet0/1
 shutdown
end


This simulated a WAN link failure between HQ and Branch.



 14. Failure Result

After the WAN interface was shut down:


R1HQ G0/1
10.255.0.1
       X
       |
R2BRANCH
10.255.0.2


The following changes were observed:

 The OSPF neighbor relationship disappeared.
 R1 no longer had an active OSPF adjacency with R2.
 Branch OSPF routes were removed from R1's routing table.
 Remote Branch networks became unreachable.

For example:


10.20.50.0/24


was no longer present in R1's routing table.

This confirms that the routing table responds dynamically to OSPF topology changes.



 15. Recovery Testing

The WAN interface was restored:

cisco
configure terminal
interface GigabitEthernet0/1
 no shutdown
end


After the link became operational, OSPF automatically began neighbor discovery again.

The adjacency progressed back to the FULL state.

The Branch routes were subsequently reinstalled into R1's routing table.

Connectivity to Branch endpoints was restored.



 16. Recovery
The following commands were used after restoring the WAN:

show ip ospf neighbor
show ip route 10.20.50.0
ping 10.20.50.21

Expected results:

OSPF neighbor → FULL
10.20.50.0/24 → OSPF route restored
Ping → Successful

The initial packet loss observed during some tests was associated with ARP/MAC learning immediately after topology changes. Repeated tests confirmed connectivity.

17. Design Limitation

The current topology contains only one WAN connection between HQ and Branch.

Therefore, the failure test demonstrates:

OSPF failure detection and route recovery/convergence.

It does not provide automatic path failover.

A production implementation could add a second WAN path and configure OSPF to provide true path redundancy.

18. Future OSPF Improvements

Potential production improvements include:

Multiple WAN links
OSPF authentication
OSPF interface cost tuning
Route summarization
Stub or totally stubby areas where appropriate
OSPF timers tuned for specific requirements
Redundant routing devices
Route filtering
Prefix control
Monitoring of OSPF neighbor state
19. Final Result

The OSPF implementation successfully provides dynamic routing between the two enterprise sites.

The final routing architecture provides:

HQ VLANs
     ↓
   R1-HQ
     ↓
 OSPF Area 0
     ↓
 R2-BRANCH
     ↓
Branch VLANs

The OSPF adjacency was successfully established, remote networks were dynamically learned, and the routing process successfully detected and recovered from a simulated WAN failure.

This demonstrates practical knowledge of OSPF configuration, route advertisement, passive interfaces, neighbor relationships, route verification, and failure recovery.
