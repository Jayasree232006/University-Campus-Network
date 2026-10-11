University Campus Network
Cisco Packet Tracer | Campus Network Simulation
A university campus network simulation connecting academic departments, administration, laboratories, and centralized services. The project demonstrates VLAN-based segmentation, multilayer switching, LACP EtherChannel, HSRP gateway redundancy, OSPF adjacency, Cisco ASA firewall policies, NAT/PAT, static routing, and DNS services.
Network architecture
The simulated campus uses a Cisco 2911 edge router (R-EDGE), a central ASA firewall (FW-CENTRAL), and two Cisco 3560 multilayer switches (MLS-1 and MLS-2). Individual department networks connect through their own firewall and access switches, while the server LAN sits behind FW-SERVER.
Departments: ADMIN, CSE, EEE, MECH, CIVIL, BIOMED, and LAB. The server segment provides DHCP, DNS, and RADIUS services.
Add a screenshot of the actual Packet Tracer topology here when available.

VLANs and IP addressing
VLAN	Network	Purpose
10	10.10.10.0/24	ADMIN
20	10.10.20.0/24	CSE
30	10.10.30.0/24	EEE
40	10.10.40.0/24	MECH
50	10.10.50.0/24	CIVIL
60	10.10.60.0/24	BIOMED
70	10.10.70.0/24	LAB
80	10.10.80.0/24	SERVERS
99	10.10.99.0/24	MANAGEMENT


Key hosts: LAB PC0 10.10.70.10, FW-LAB inside gateway 10.10.70.1, SRV-DHCP 10.10.80.10, SRV-DNS 10.10.80.11, SRV-RADIUS 10.10.80.12.
Implemented technologies
- VLANs: logical segmentation of departmental networks and management traffic.
- EtherChannel (LACP): Po1 bundles Fa0/22 and Fa0/23 between MLS-1 and MLS-2.
- HSRP: management VLAN virtual IP 10.10.99.1; MLS-1 is Active (priority 110) and MLS-2 Standby (priority 100).
- OSPF: adjacency between MLS-1 (1.1.1.1) and MLS-2 (2.2.2.2) over VLAN 99.
- Static routes: department and server paths through the campus core and ASA firewalls.
- Firewall ACLs and NAT: Cisco ASA access policies, FW-LAB dynamic PAT, and FW-SERVER identity static NAT for selected servers.
- Centralized services: DHCP pools, DNS records and RADIUS AAA service configured; not all application-level features have been tested end-to-end.
Troubleshooting case study: LAB-to-server communication
Initially, LAB PC0 could access its local gateway but could not reach 10.10.80.10 or 10.10.80.11, and DNS lookups failed. Investigation used routing tables, ASA configurations, ACL counters, NAT counters, and Packet Tracer Simulation Mode.
The evidence identified several contributing issues:
1. FW-SERVER ACL binding: an existing outside ACL was defined but not applied. Applying it restored MLS-2-to-DNS ICMP connectivity.
2. Translated LAB source: FW-LAB's PAT changed the PC0 source from 10.10.70.10 to 10.255.70.2. FW-SERVER needed narrow permits for the translated address.
3. Missing return route: FW-CENTRAL required a route for 10.255.70.0/30 through MLS-1 at 10.255.2.1 so responses could return toward FW-LAB.
4. DNS permissions: an inbound UDP/53 permit at FW-SERVER allowed the DNS query; Simulation Mode then identified an ACL drop of the DNS response at FW-LAB. A narrowly scoped response permit restored DNS resolution.
5. DHCP server ICMP permission: a specific permit at FW-SERVER restored ICMP reachability to 10.10.80.10 from LAB.
Changes were made without redesigning the topology or replacing working EtherChannel, HSRP, OSPF, or VLAN configurations.
Verification
Test	Observed result
show etherchannel summary	Po1(SU); both members (P)
show standby brief	MLS-1 Active; MLS-2 Standby
show ip ospf neighbor	MLS-2 neighbor in FULL/DR
show vlan brief	Required campus VLANs present
PC0 → LAB gateway	Successful
PC0 → 10.10.80.11	3/4 ICMP replies during verification
PC0 → 10.10.80.10	3/4 ICMP replies during verification
nslookup dns.campus.local on PC0	Successful
nslookup dhcp.campus.local on PC0	Successful


Scope note: ICMP reachability to the DHCP server does not establish successful DHCP lease assignment. End-to-end DHCP leasing for every department and RADIUS client authentication remain unverified. HSRP failover itself was not demonstrated.
Run the project
1. Install a compatible version of Cisco Packet Tracer.
2. Download and open [`University_Campus_Network.pkt`](./University_Campus_Network.pkt).
3. Allow time for ARP, LACP, STP, HSRP, and OSPF to converge.
4. On MLS-1, inspect show vlan brief, show etherchannel summary, show standby brief, and show ip ospf neighbor.
5. On LAB PC0, test ping 10.10.80.11 and nslookup dns.campus.local.
Project files
- [`University_Campus_Network.pkt`](./University_Campus_Network.pkt) — Packet Tracer project
- Documentation, presentation, and annotated screenshots — to be added
Security note
This is an educational simulation. Check saved device configurations for passwords, RADIUS shared secrets, and other sensitive data before publicly distributing the Packet Tracer file.
