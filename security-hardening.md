 Security Hardening

 1. Purpose

Security hardening was applied to the network infrastructure to reduce unnecessary exposure, prevent accidental network loops, and protect unused interfaces and services.

The goal was to implement practical baseline security measures suitable for an enterprise network lab while keeping the configuration simple and maintainable.



 2. Security Measures Implemented

The following hardening measures were implemented:

 Unused switch interfaces were placed into a dedicated unused VLAN.
 Unused switch interfaces were administratively shut down.
 Unused router interfaces were administratively shut down.
 HTTP and HTTPS management services were disabled on the switches.
 Router DNS lookup was disabled to prevent delays caused by mistyped commands.
 Switch DNS lookup was disabled for the same reason.
 Access ports were explicitly configured as access ports.
 Enddevice ports were configured with PortFast Edge.
 OSPF was made passive on user VLAN interfaces.
 Infrastructure management interfaces were placed in dedicated management VLANs.



 3. UnusedPort Security

A dedicated VLAN was used for unused switch interfaces:


VLAN 999
Name: UNUSEDPORTS


Unused interfaces were assigned to VLAN 999 and shut down.

Example:

cisco
interface GigabitEthernet0/2
 switchport access vlan 999
 switchport mode access
 shutdown


This configuration was applied to unused switch ports.

 Why VLAN 999?

Using a dedicated unused VLAN creates a clear separation between active production VLANs and interfaces that should not be used.

Even though the interfaces are also administratively disabled, the unused VLAN provides an additional layer of organizational control and makes the configuration easier to audit.



 4. Administrative Shutdown of Unused Router Interfaces

Unused router interfaces were also disabled.

Example:

cisco
interface GigabitEthernet0/2
 shutdown

interface GigabitEthernet0/3
 shutdown


The same approach was used on both routers.

This prevents an unused physical interface from accidentally becoming an active network connection.



 5. Disabling Unnecessary Web Services

The builtin HTTP management services were disabled on the switches.

cisco
no ip http server
no ip http secureserver


The purpose is to reduce unnecessary management services and minimize the infrastructure's attack surface.

These services were not required for the operation of the lab.



 6. Disabling DNS Lookup

DNS lookup was disabled on the routers and switches:

cisco
no ip domain lookup


Without this command, an incorrectly entered IOS command may be interpreted as a hostname and IOS can attempt a DNS lookup.

For example, a typo such as:

show vlan brief

could result in an unnecessary lookup attempt.

Disabling DNS lookup makes configuration and troubleshooting faster.

7. Access-Port Hardening

End-device interfaces were explicitly configured as access ports.

Example:

interface GigabitEthernet0/1
 switchport access vlan 10
 switchport mode access

This prevents the interface from being configured as a trunk through normal switch negotiation behavior and ensures that the endpoint remains associated with its intended VLAN.

The same approach was used for the other end-device interfaces.

8. PortFast Edge

PortFast Edge was enabled on end-device access ports.

Example:

interface GigabitEthernet0/1
 spanning-tree portfast edge

This allows an endpoint-facing interface to transition to forwarding more quickly instead of waiting through normal spanning-tree transition states.

It is appropriate for ports connected to devices such as PCs.

PortFast should not be used indiscriminately on switch-to-switch links.

9. OSPF Passive Interfaces

OSPF was configured so that user VLAN interfaces do not attempt to form OSPF neighbor relationships.

HQ:

router ospf 10
 passive-interface GigabitEthernet0/0.10
 passive-interface GigabitEthernet0/0.20
 passive-interface GigabitEthernet0/0.30
 passive-interface GigabitEthernet0/0.40

Branch:

router ospf 10
 passive-interface GigabitEthernet0/0.50
 passive-interface GigabitEthernet0/0.60
 passive-interface GigabitEthernet0/0.70

The WAN interface remains active for the OSPF adjacency.

This follows a practical principle:

User-facing interfaces → Passive
Router-to-router links → Active

The result is that OSPF routing information is still advertised for the VLAN networks while unnecessary neighbor discovery is prevented on end-user networks.

10. Management Network

Management interfaces were placed in dedicated management VLANs.

HQ
Management VLAN: 40
Network: 10.10.40.0/24
Gateway: 10.10.40.1

SW-HQ-CORE   → 10.10.40.2
SW-HQ-ACCESS → 10.10.40.3
Branch
Management VLAN: 70
Network: 10.20.70.0/24
Gateway: 10.20.70.1

SW-BRANCH → 10.20.70.2

This separates infrastructure management addressing from normal end-user VLANs.

11. Switch Default Gateway

Because the switches operate primarily at Layer 2 in this design, management traffic requires a default gateway.

HQ core:

ip default-gateway 10.10.40.1

HQ access:

ip default-gateway 10.10.40.1

Branch:

ip default-gateway 10.20.70.1

Static default routes were also configured on the switches:

ip route 0.0.0.0 0.0.0.0 10.10.40.1

and:

ip route 0.0.0.0 0.0.0.0 10.20.70.1

These allow management traffic to reach networks outside the local management subnet.

12. OSPF Security Considerations

OSPF was restricted to the required router-to-router WAN connection.

The only expected OSPF adjacency is:

R1-HQ ↔ R2-BRANCH

The adjacency was verified in the FULL state.

The design does not implement OSPF authentication. In a production environment, OSPF authentication should be considered to help protect the routing domain from unauthorized routing peers.

13. Current Security Scope

This project focuses on infrastructure hardening rather than complete enterprise security.

The implemented controls protect against common configuration and operational risks such as:

Unused interfaces
Unnecessary management services
Accidental OSPF neighbor formation
Improper access/trunk configuration
Uncontrolled infrastructure access
Unnecessary DNS lookup delays
14. Security Features Not Implemented

The following controls were intentionally left for future projects or future expansion:

Extended ACLs between departments.
Firewall policies.
NAT security policies.
OSPF authentication.
AAA with TACACS+ or RADIUS.
Port security with static/sticky MAC addresses.
DHCP snooping.
Dynamic ARP Inspection.
IP Source Guard.
802.1X authentication.
Centralized logging and SIEM integration.
SNMP-based security monitoring.

These features were outside the primary scope of this OSPF routing project.

15. Verification

The following commands can be used to verify the hardening configuration.

Check unused ports
show interfaces status
Check VLAN assignment
show vlan brief
Check management interface
show ip interface brief
Check default gateway
show running-config | include default-gateway
Check HTTP services
show running-config | include ip http

Expected result:

no ip http server
no ip http secure-server
Check OSPF passive interfaces
show ip protocols
Check OSPF neighbors
show ip ospf neighbor
16. Security Validation

The completed configuration was checked to confirm that:

Unused switch ports were assigned to VLAN 999 and shut down.
Unused router interfaces were shut down.
HTTP management services were disabled on the switches.
DNS lookup was disabled on infrastructure devices.
End-device interfaces were configured as access ports.
PortFast Edge was applied to endpoint-facing interfaces.
OSPF user VLAN interfaces were passive.
Switch management addresses were reachable through their management gateways.
OSPF adjacency remained operational between the two routers.
17. Future Enterprise Security Model

A production version of this topology could evolve toward:

                    Internet
                       |
                    Firewall
                       |
                ┌──────┴──────┐
                │             │
             HQ Core       Branch Core
                │             │
          Access Switches  Access Switch
                │             │
             Users         Users

Additional controls could then be introduced at the firewall, routing, switching, authentication, monitoring, and endpoint layers.

18. Final Result

The network now includes a practical baseline security configuration covering infrastructure management, unused interfaces, VLAN separation, OSPF behavior, and unnecessary services.

The hardening measures reduce the attack surface while keeping the topology operational and easy to troubleshoot.

Security hardening: Completed and verified.
