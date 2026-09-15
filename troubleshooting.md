 Troubleshooting Guide

 1. Purpose

This document records the troubleshooting process used during the deployment and verification of the MultiSite Enterprise OSPF network.

The objective was not only to make the network work, but also to identify common enterprise networking failures, isolate their causes, and verify recovery.



 2. Troubleshooting Methodology

A structured troubleshooting approach was used:


1. Identify the failure
2. Check physical/interface status
3. Check VLAN and Layer 2 configuration
4. Check Layer 3 addressing
5. Check routing
6. Check DHCP
7. Test connectivity
8. Make the required correction
9. Retest
10. Document the result


This prevents random configuration changes and makes the troubleshooting process easier to reproduce.



 3. Common Verification Commands

 3.1 Interface Status

On Cisco routers and switches:

cisco
show ip interface brief


This quickly identifies interfaces that are:


up/up
administratively down
down/down


For switch ports:

cisco
show interfaces status




 3.2 Routing Table

On routers:

cisco
show ip route


To check a specific remote network:

cisco
show ip route 10.20.50.0


An OSPFlearned route should contain an O code.

Example:


O    10.20.50.0/24 [110/2] via 10.255.0.2




 3.3 OSPF Neighbor Status

Check the OSPF adjacency with:

cisco
show ip ospf neighbor


The expected relationship is:


R1HQ ↔ R2BRANCH


The neighbor should reach:


FULL


A missing neighbor usually indicates a problem with the WAN interface, IP addressing, OSPF configuration, or Layer 1/Layer 2 connectivity.



 3.4 VLAN Verification

On switches:

cisco
show vlan brief


This verifies VLAN membership and accessport assignments.

To inspect trunks:

cisco
show interfaces trunk


Expected HQ trunk VLANs:


10,20,30,40


Expected Branch trunk VLANs:


50,60,70




 3.5 DHCP Verification

On the DHCP routers:

cisco
show ip dhcp pool


and:

cisco
show ip dhcp binding


A client can request an address with:


ip dhcp


and verify the result with:


show ip




 4. Issue 1 — Initial Ping Timeouts

 Symptom

During initial connectivity testing, some first ping attempts timed out even though the network configuration was correct.

 Investigation

The interfaces, gateways, OSPF routes, and addressing were checked.

The network showed:

 Correct IP addressing.
 Correct default gateways.
 OSPF adjacency in FULL state.
 Correct remote OSPF routes.
 Successful subsequent ping tests.

 Cause

The initial packet loss was associated with ARP and MACaddress learning.

When a device communicates with a new destination for the first time, it may need to resolve the required Layer 2 information before the ICMP exchange completes.

 Resolution

The ping test was repeated.

Subsequent tests succeeded with full reachability.

Example:


5 packets transmitted, 5 packets received, 0.0% packet loss


 Lesson

A single initial timeout does not automatically indicate a routing failure. Always repeat the test and inspect ARP, interface, and routing status before changing the configuration.



 5. Issue 2 — Switch Management Interface Initially Unreachable

 Symptom

The HQ switch management interface on:


10.10.40.3


was initially unreachable from the test PC.

The other management addresses were reachable.

 Investigation

The following were checked:

cisco
show ip interface brief
show runningconfig


The management SVI was configured correctly.

The switch had:


10.10.40.3/24


and the management gateway was:


10.10.40.1


 Cause

The switch required a usable Layer 3 path for management traffic leaving its local subnet.

 Resolution

The switch was given the required static default route:

cisco
ip route 0.0.0.0 0.0.0.0 10.10.40.1


The default gateway was also configured:

cisco
ip defaultgateway 10.10.40.1


 Verification

The management IP was tested again:


ping 10.10.40.3


The result became:


5/5 successful


 Lesson

Layer 2 switches still require a valid management gateway/path when management traffic must reach a different subnet.



 6. Issue 3 — DHCP Bindings Not Always Present

 Symptom

At one point:

cisco
show ip dhcp binding


did not show active entries.

 Investigation

The DHCP pools themselves remained configured.

The client devices were then checked.

Because DHCP leases are dynamic, an empty binding table does not necessarily mean DHCP configuration has been deleted.

 Resolution

The VPCS clients were renewed with:


ip dhcp


The clients successfully received addresses again.

 Lesson

DHCP configuration and active DHCP leases are different things.

Use:

cisco
show ip dhcp pool


to inspect pool configuration and:

cisco
show ip dhcp binding


to inspect currently active leases.



 7. Issue 4 — Branch Reachability Failure After WAN Shutdown

 Purpose

A controlled failure was introduced to verify that the network could detect and recover from a WANlink outage.

This was an intentional fault injection rather than an accidental failure.

 Step 1 — Baseline

Before the failure, R1HQ had an OSPF route to:


10.20.50.0/24


through:


10.255.0.2


The OSPF neighbor relationship was:


FULL


 Step 2 — Introduce Failure

R1HQ WAN interface was shut down:

cisco
configure terminal
interface GigabitEthernet0/1
 shutdown
end


 Step 3 — Verify Interface Failure

The interface became:


administratively down/down


 Step 4 — Verify OSPF

The OSPF neighbor relationship disappeared.

cisco
show ip ospf neighbor


No active neighbor was present.

 Step 5 — Verify Routing

The remote branch route was removed.

cisco
show ip route 10.20.50.0


Result:


% Subnet not in table


 Step 6 — Verify Connectivity

A ping to the branch network failed because the route was no longer available.

This confirmed that the routing protocol was responding to the WAN failure.



 8. WAN Recovery

After completing the failure test, the WAN interface was restored.

cisco
configure terminal
interface GigabitEthernet0/1
 no shutdown
end


 Recovery Verification

The OSPF adjacency was checked again:

cisco
show ip ospf neighbor


The neighbor returned to:


FULL


The branch route was restored:

cisco
show ip route 10.20.50.0


The remote network again appeared through:


10.255.0.2


Connectivity testing was then repeated.

The first packet after recovery could experience a timeout while ARP information was rebuilt, but repeated tests succeeded.



 9. FailureRecovery Summary

| Test                   | Expected Result           | Actual Result |
|  |  |  |
| Normal OSPF operation  | Neighbor FULL             | Passed        |
| Remote route available | OSPF route installed      | Passed        |
| WAN shutdown           | Neighbor lost             | Passed        |
| WAN shutdown           | Remote route removed      | Passed        |
| WAN shutdown           | Remote connectivity fails | Passed        |
| WAN restored           | Neighbor returns FULL     | Passed        |
| WAN restored           | Remote route returns      | Passed        |
| WAN restored           | Connectivity returns      | Passed        |



 10. Troubleshooting Decision Tree

A practical troubleshooting sequence for this topology is:


                 Problem
                    |
                    v
          Is the interface UP?
             /            \
           NO              YES
           |                |
     Check cable,      Check IP address
     shutdown,          and subnet mask
     interface config        |
                             v
                    Can local gateway
                         be pinged?
                      /          \
                    NO            YES
                    |              |
               Check VLAN,      Check routing
               trunk, access      |
               and gateway        v
                              Is OSPF FULL?
                              /          \
                            NO            YES
                            |              |
                       Check WAN,      Check remote
                       OSPF, IPs,     subnet and ACLs
                       network area


In this project there were no ACLs or firewall policies configured, so routing and Layer 2/Layer 3 connectivity were the primary areas of investigation.



 11. Useful Troubleshooting Commands

 Router commands

cisco
show ip interface brief
show ip route
show ip route <network>
show ip ospf neighbor
show ip ospf interface brief
show ip protocols
show ip dhcp pool
show ip dhcp binding
show runningconfig


 Switch commands

cisco
show vlan brief
show interfaces trunk
show interfaces status
show ip interface brief
show runningconfig
show spanningtree


 VPCS commands


show ip
ip dhcp
ping <destination>




 12. Troubleshooting Best Practices

 Start with the lowest layer

Always check physical/interface status before assuming the problem is routing.

 Verify both sides

For a link between two devices, inspect both interfaces rather than only one.

 Check the routing table

A working OSPF neighbor does not automatically mean every expected route is present.

 Separate local and remote problems

First verify:


PC → Default Gateway


Then verify:


HQ → Branch


This makes fault isolation much easier.

 Repeat tests

An isolated packet loss event can be caused by ARP or MAC learning. Repeated testing provides better evidence.

 Change one thing at a time

Avoid making several unrelated configuration changes simultaneously. Otherwise, it becomes difficult to determine which change actually solved the problem.



 13. Project Troubleshooting Result

The project was successfully tested under both normal and failure conditions.

The troubleshooting process demonstrated that:

 VLAN connectivity could be verified locally.
 DHCP clients could obtain addressing dynamically.
 OSPF established a FULL adjacency between sites.
 Remote networks were learned through OSPF.
 A WAN failure caused the expected loss of the OSPF adjacency and remote routes.
 Restoring the WAN interface reestablished routing and endtoend connectivity.
 Switch management connectivity was corrected through proper Layer 3 management routing.



 14. Important Design Limitation

The failure test demonstrates failure detection and recovery, not redundant failover.

There is only one WAN link between:


R1HQ ↔ R2BRANCH


Therefore, when that link fails, connectivity between the sites is temporarily lost.

A production network would normally use redundant WAN paths, dual routers, or another resilient design to maintain connectivity during a singlelink failure.



 Final Result

The troubleshooting and recovery process confirmed that the network behaves predictably during normal operation and controlled failure scenarios.

Troubleshooting and recovery testing: Completed and verified.
