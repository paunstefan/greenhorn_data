# CCNA course notes

#### Abbreviations:

* ARP: Address Resolution Protocol
* NAT: Network Address Translation
* ICMP: Internet Control Message Protocol
* MAC: Media Access Control
* PDU: Protocol Data Unit
* OSPF: Open Shortest Path First
* EIGRP: Enhanced Interior Gateway Routing Protocol
* BGP: Border Gateway Protocol
* ISR: Integrated Service Router
* ASR: Aggregated Service Router
* VLSM: Variable Length Subnet Mask
* CEF: Cisco Express Forwarding
* CAM = Content Addressable Memory
* ACL = Access Control Lists
* PAT = Port Address Translation
* CDP =  Cisco Discovery Protocol 
* LLDP = Link Layer Discover Protocol
* CLM = Cisco Licence Manager
* PAK = Prodct Activation Key
* UDI = Unique Device Identifier
* STP = Spanning Tree Protocol
* ISL = Inter-Switch Link
* DTP = Dynamic Trunking Protocol
* SVI = Switch Virtual Interface
* IDS = Intrusion Detection System
* IPS = Intrusion Protection System
* DAD = Duplicate Address Detection
* VTP = VLAN Trunking Protocol
* DTP = Dynamic Trunking Protocol
* STP = Spanning Tree Protocol 
* PAgP = Port Aggregation Protocol
* LACP = Link Aggregation Control Protocol
* FHRP = First Hop Redundancy Protocols
* HSRP = Hot Standby Router Protocol
* PAP = Password Authentication Protocol
* CHAP = Challenge Handshake Authentication Protocol
* DMVPN = Dynamic Multipoint VPN
* GRE = Generic Routing Encapsulation
* SNMP = Simple Network Management Protocol
* SPAN = Switch Port Analyzer
* ACI = Cisco Application Centric Infrastructure
* APIC = Application Policy Infractructure Controllers
* IP SLA = IP Service Level Agreement
* NAM = Network Analysis Module

## OSI Model

* Application => User interface 
* Presentation => Data formatting, compression, encryption 
* Session => Separates communication between applications 
* Transport => Segments and reassembles data; Provides multiplexing for the upper layers 
* Network => Addressing & routing 
* Data Link => Determines how packets are placed on the media, identifies network layer protocols, does physical addressing 
* Physical => Sends and reveices bits 

**PDU Names:**
  * Transport: Segment
  * Network: Packet (TCP) / Datagram (UDP)
  * Data link: Frame

### Data Link layer

There are 2 sublayers:

Logical Link Control (LLC) = knows information about L3 protocols (driver software for NIC)

Media Access Control (MAC) =
* addressing 
* frame delimitation
* gets frames on and off the media
* knows how to format a frame for a media
* (like traffic rules)

Contention based access: devices wait for the media to be free before transmitting

### Connectors:
**RJ-45**

T568A: G/ G O/ B B/ O M/ M

T568B: O/ O G/ B B/ M/ M
  * Straight through = both ends are the same (used between devices of different kinds)
  * Crossover = one end is type A, the other is type B (used between devices of the same kind)
  * Rollover = one end is the other end reversed (used for console connections)

**Fiber optics**

  * Single mode (laser, 1 beam)
  * Multi-mode (LED, many beams)


### Carrier sense multiple action (CSMA)
  * /CD (collision control): detects collisions and retransmits
  * /CA (collision avoidance): waits until others finish (WiFi)

### Troubleshooting steps
  - ping localhost (TCP/IP stack)
  - ping a host on the local network (cables/NIC)
  - ping default gateway (local network)
  - ping remote server by IP address
  
## Switching

Switches break up collision domains.

Routers break up broadcast domains.

| Preamble | Dest MAC | Src MAC | Type | Payload | CRC |

| 8 | 6 | 6 | 2 | 46-1500 | 4 |

64-1518 Bytes

  * Store-and-forward = waits for the whole frame, computes CRC and forwards
  * Cut-through = can forward as soon as it receives the destination address

Autonegotiation = enables 2 devices to share info about speed and duplex

Auto Medium-Dependent Interface Crossover (Auto-MDIX) = detects the type of cable used

ARP spoofing = and attacker responds to an APR reqest for another device

MAC address first 6 digits = vendor

PPP = point-to-point protocol

CDP = Cisco Discovery Protocol (allows Cisco devices to share info over link-layer)


01:00:5E:... (multicast)

## Internet Protocol
![IP Header](/static/images/ip-header.png)

* Connectionless
* Unreliable
* Best-effort

Print routing table: 

Windows:
* route print
* netstat -r

Linux:
* route
* netstat -rn

**Route table example:**

| D |10.1.1.0/24| [90 2170] | via 200.165.200.226| 00:00:05| Serial 0/0/0|

|Route source|Dest addr| AD & Metric | Next hop | Timestamp | Exit interface |

Routing table:
* C - connected network
* L - interface (local host route)

**Routers**

Boot-up: 
* ROM: POST & bootstrap(locates and loads IOS into RAM)
* Flash: loads IOS
* NVRAM: loads config file

### Subnets

* How many subnets ? -> 2^(nr of borrowed bits)
* How many hosts per subnet ? -> 2^(nr of host bits) - 2
* Valid subnets ? -> 256 - subnet mask = block size
* Broadcast address -> number before the next subnet
* Valid hosts -> number between the subnet address and broadcast address

Private addresses: 
* 10.0.0.0/8
* 172.16.0.0/12
* 192.168.0.0/16

* Class A: 0-127.0.0.0/8 (begins with 0)
* Class B: 128-191.0.0.0/16 (begins with 10)
* Class C: 192-223.0.0.0/24 (begins with 110)

### IPv6

* Global unicast: 2000::/3
* Link local: FE80::/10
* Multicast: FF00::/8
* Unique local(like private addresses, routables to private networks): FC00::/7
* Loopback: ::1/128
* All routers: FF02::2
* All nodes: FF02::1
* All DHCP servers: FF02::1:2
* Anycast (many devices can use the same address and the packet goes to the closest)
* Used for 6-to-4 tunneling: 2002::/16

| Global routing prefix | Subnet ID | Interface ID |

| 48b | 16b| 64b |

EUI-64 -> the last 64 bits are the MAC address with FFFE added in the middle and the 7th bit is reversed

SLAAC = Stateless Address Autoconfiguration (uses ICMPv6 router advertisments)

Neighbor Discovery -> uses ICMP solicited-node address; the host sends a Neighbor solicitation to the multicast address FF02:0:0:0:0:1:FF/104 with teh last 24 bits replaced by the last 24 bits of the IP address.

Neighbor states:
* INCMP -> NS has been sent, no response yet
* REACH -> confirmation received (good)
* STALE -> no communication in an interval, change to REACH on the next packet received
* DELAY -> occurs after STALE
* PROBE -> resending NS

## Transport layer

* Tracking individual conversations
* Segmenting and reassembling data
* Identifying applications

Header size: 20 bytes TCP / 8 bytes UDP

![](/static/images/https://www.cisco.com/c/dam/en_us/about/ac123/ac147/images/ipj/ipj_9-4/94_syn_fig1_lg.jpg)
![](/static/images/http://www.bogotobogo.com/cplusplus/images/socket/TCP_DISCONNECT_TEAR_DOWN.png)

**Routers**
* Interconnect networks
* Choose best paths

**Packet forwarding mechanisms:**
* Process switching - the CPU matches every packet with an interface
* Fast switching - flow information is stored in cache and there is CPU intervention only for the first packet
* Cisco Express Forwarding (CEF) - uses a forwarding information base and adjancency table

### Routing protocols

distance vector = uses hop count, learns routes from directly connected networks

link-state = uses 3 tables (currently attached neighbors, network topology, routing table), sends keepalives, exchanges only updates

advanced distance vector = uses aspects from both

**Metrics**

* RIP = hop count
* OSPF = cumulative bandwidth
* EIGRP = bandwidth, delay, load reliability

Administrative distance = trustworthiness of a route (lower is better)

0.0.0.0 / ::/0 = default route

Floating static route = alternative route in case of faliure (greater AD)

Use fully specified route for IPv6 if the next hop is link-local.

* Level 1 route = subnet mask equal or less then the classful mask
* Level 1 parent route = a network that is subnetted
* Level 2 child route = a subnet


### Cisco Borderless Network / Switched networks

* Access layer: network access to the users
* Distribution layer: routing; differentiated services, security; layer 3 switches
* Core layer: network backbone; connects geographically separate networks

**Switches**

Switch functions:
* address learning
* forward/filter decisions
* loop avoidance (using STP)

frame filtering = send frames only on destination port
  
Rack unit = thicness

* Fixed config
* Modular config
* Stackable config (Stackwise switches; combine multiple switches into 1 with Stackwise cables, max 9)

**Switching = device making a decision based on ingress port and destination address**

CAM = Content Addressable Memory

CAM table = MAC address table

Forwarding methods:
* Store-and-forward = receive the entire frame, compute the CRC and send to destination
* Cut-through = forward the frame after only the destination address is received

Trunk port = link between switches that transport traffic from multiple VLANs

- ROM -> POST and bootloader (initializes the flash filesystem
- NVRAM -> executes startup-config

//int status up, line protocol up//  (first up is for physical layer, the second for data-link)

**! Shutdown unused ports!**

Port security = specify MAC addresses allowed on the port

* Legacy inter-VLAN routing = connect each VLAN to a different router interface
* Router-on-a-stick = uses virtual interfaces (subinterfaces) for inter-VLAN routing

Violations:
* protect = drop packets
* restrict = drop packets and send log messages
* shutdown = shuts port down (shut -> no shut to restore)
  
Port types:
* access port = port that carries traffic and belongs to only 1 VLAN (traffic is sent in native format, no tag)
* voice access port = a 2nd VLAN added to an access port
* trunk port = carry multiple VLANs
* native vlan = where untagged traffic is

* ISL = VLAN tagging method by Cisco
* IEEE 802.1q = standard tagging
  
## Access Control Lists
* Simple firewall
* ACE = Access Control Entries
* Wildcard mask = inverse subnet mask

Numbers:
* 1-99, 1300-1999 = standard ACL
* 100-199, 2000-2699 = extended ACL
  
To calculate wildcard mask, from 255.255.255.255 substract the subnet mask.

* Host = 0.0.0.0 wildcard
* Any = 255.255.255.255 wildcard

1 ACL for: each protocol, each direction, each interface

Rules:
* more specific tests at the top
* standard ACLs as close to the destination as possible ( they use only source address)
* extended ALCs as close to the source as possible

## Dynamic Host Configuration Protocol (DHCP)

DHCPv4:
- DHCP discover (broadcast)
- DHCP offer
- DHCP request (broadcast)
- DHCP ack

Troubleshooting:
* Resolve addr conflicts
* Verify physical connectivity
* Test with static address
* Verify switch ports
* Test from the same subnet

### Stateless Address Autoconfiguration (SLAAC)

* Obtain config without DHCPv6
* Uses ICMPv6
* Creates addresses using EUI-64(MAC address) or random
* M = 0, O = 0

- Router solicitation (RS)
- Router advertisment (RA)
- Verification (DAD)

Options: M, O flags

### Stateless DHCPv6 (RA + DHCP)
* M = 0, O = 1
* Use info from RA for addressing
* Get other info from DHCP server

### Stateful DHCPv6
* M = 1
* All information is from DHCP server

- Solicit
- Advertise
- Request (for stateful) or Information-request (stateless)
- Reply

## Network Address Translation
* Inside local = local address of sending device
* Outside local = destination address from inside network
* Inside global = global address of sending device
* Outside global = destination address from outside network

Mapping is added to the NAT table.

* Static NAT = one to one translation
* Dynamic NAT = many to many (uses pool)
* Port Address Translation (PAT)/ NAT overload = many to one (uses ports)


## Maintnance
Boot sequence:
- Perform POST
- Bootstrap locates and loads IOS (looks in flash, tftp, and ROM)
- IOS looks for config file in NVRAM
- Copy the file from NVRAM to RAM, if there is no file it searches for TFTP server and then starts setup mode

* Cisco Discovery Protocol (CDP)
* Link Layer Discover Protocol (LLDP)

NTP port = 123

Syslog port = 514

Log message:

 //seq. nr: timestamp: %facility-severity-mnemonic: description//

universalk9 = full cryptography capabilities
universalk9_npe = cryptography lite

1900, 2900, 3900 = ISR G2 (generation 2)

You activate packages using Cisco Software Activation licensing keys.

IOS name:

**C1900-universalk9-mz.SPA.152-4.M3.bin**

* C1900 = platform (1900 router)
* universalk9 = image designation
* mz = location and compression (RAM and compressed)
* SPA = Digital Signature indicator
* 152-4.M3 = version; major release, minor release, new feature release, extended maintnance release, maintnance rebuild
* bin = file extension

Bootstrap protocol cand send and operating system to a host and assign an IP address.

Interface problem when CRC and input errors grow, but collisions do not.

If 'no buffer' and 'ignored' grow -> broadcast storm = bad NIC or bad network design

* Collisions cause runts, frame errors
* CRC = interference
* Collision = duplex mismatch

broadcast storm = 2 switches continuosly exchange broadcasts
  
Syslog severity levels:
* 0) Emergency
* 1) Alert
* 2) Critical
* 3) Error
* 4) Warning
* 5) Notification
* 6) Informational
* 7) Debugging

Configuration register: 0x2102  (default)
  
Configuration register:
| bit | description |
| 0-3 | boot field |
| 6 | ignore NVRAM |
| 7 | OEM bit enabled |
| 8 | break disabled |
| 10 | ip broadcast with all zeros |
| 5, 11-12 | console line speed |
| 13 | boot default ROM |
| 14 | IP broadcasts do not have net number |
| 15 | enable diag message and ignore NVRAM |

Boot field
| 00 | ROM monitor |
| 01 | boot image from ROM |
| 02-F | specifies a default boot file |



## Scalability
* Use expendable, modular equipment or clustered devices
* Design a hierarchical network
* Create an IP address strategy
* Use layer 3 devices to limit broadcasts and filter traffic
* Use redundant links

#### Switch types
* Campus LAN: 2960, 3560, 3650, 3850, 4500, 6500, 6800
* Cloud managed: Meraki
* Data center: Nexus & 6500
* Virtual networking: Nexus


## VLAN Trunking Protocol (VTP)
Syncronizes VLAN info over trunk links.

VTP domain:
* 1 or more interconnected switches
* share VLAN information via VTP advertisements 
* router or L3 switch a boundary

VTP advertisements:
* each switch sends global config advertisements from each trunk port
* neighbors update their VLAN configuration

VTP modes: server, client, transparent.

#### VTP advertisments
- Summary advertisements: VTP domain name and config revision number (every 5 minutes)
- Advertisement request: sent when summary advert contain a higher revision number
- Subset advertisement: VLAN information

You can add extended VLANs only in transparent mode.


![](/static/images/vtp.png)
![](/static/images/vtp-q.png)

VTP pruning = if a switch does not have ports in a specific VLAN, it does not receive broadcasts for that VLAN (even if it is in his database) -> preserves bandwidth

Observations: 
* Reset configuration number by assigning a false domain name then reassign the old one.
* If 1 switch uses VTP v2, all switches will switch to v2.

### Dynamic Trunking Protocol (DTP)
Negotiates trunking modes

![](/static/images/dtp.png)

  
## Layer 3 switching
Switches suppoprt the following types of L3 interfaces:
* Router port (pure L3 interface)
* Switch Virtual Interface (SVI)

*Default gateway must be the L3 switch SVI

#### Issues with redundancy
* MAC database instability (from network loops)
* Broadcast storms
* Duplicate unicast frames

## Spanning Tree Protocol (STP)

* Ensures that there is only 1 logical path between all destinations on the network
* A switch is chosen as the root bridge, used as reference for all path calculations
* Switches exchange bridge protocol data units (BPDU) to determine the lowest Bridge ID (BID) (it becomes root bridge)
* After that they calculate the shortest path to the root bridge
* On every link, there can only be 1 designated port. So if a port is DP the other end is either root or blocked.

The BID contains:
* Priority value
* MAC address
* optional extended system ID

Port roles: 
* Root port: port closest to the root bridge in terms of overall cost
* Designated port: all non-root ports that are allowed to forward traffic
* Alternate (backup) port: ports in blocking state
* Disabled port: shutdown port

When the root bridge has been elected, STA(Spanning Tree Algorithm) starts calculating the best paths to the root by using port costs (speed). 

When selecting which ports to block, lowest path cost to root has priority, then lowest BID, then port priority (+port number). 

Types of STP:
* Legacy STP: 802.1D-1998
* PVST+ (Cisco protocol- Per VLAN STP): 1 instance for each VLAN
* 802.1D-2004
* RSTP (faster convergence): 802.1w
* Rapid-PVST+
* Multiple STP (Maps multiple VLANs into the same ST instance)

Port states:
* Blocking: alternate port, receives BPDU frames to determine the STP topology
* Listening: listens for the path to the root, is prepared to participate in forwarding
* Learning: learns MAC addresses, prepares to participate in forwarding
* Forwarding: part of the active topology
* Disabled

RSTP states:
* Discarding
* Learning
* Forwarding

Portfast:
* Transitions from blocking directly to forwarding
* Use of access ports

BPDU guard:
* Puts port in error disabled mode if it receives BPDU

Maximum recommended STP diameter: 7

## Link aggregation
* Creating 1 logical link using multiple physical links
* Resulting Etherchannel interface = port channel

Etherchannel:
* Up to 8 ports/channel
* Up to 6 channels/switch

### Port Aggregation Protocol (PAgP)

* Cisco proprietary
* Creates/manages Etherchannel

Modes:
* On: forces channel creation without PAgP
* PAgP desirable: actively asking to create channel
* PAgP auto: passively waits for channel creation

### Link Aggregation Control Protocol (LACP)

* IEEE standard (802.3ad)

Modes:
* On
* Active
* Passive

## First Hop Redundancy Protocols (FHRP)
**Virtual routers**: multiple routers configured to look as a single router on the network

**Proxy ARP**: router responds to ARP for remote hosts

3 redundancy protocols:
* HSRP
* Virtual Router Redundancy Protocol (VRRP)
* Gateway Load Balancing Protocol (GLBP)

### Hot Standby Router Protocol (HSRP) ====
* Priority: router with the highest priority number becomes active (default = 100)
* Preemption: when the router with highest priority becomes active again it regains the active state

HSRP MAC address:
* first 24 bits = OUI
* next 16 = HSRP MAC = 07AC (v2: 9FF)
* next 8 = HSRP group number

HSRP timers:
* Hello
* Hold (time to wait for hellos)
* Active
* Standby

Interface tracking = if the outside interface fails, the the standby router will take over and priority of the active one lowers

States:
* Initial
* Learn
* Listen
* Speak
* **Standby**
* **Active**

![](/static/images/hsrp.png)
![](/static/images/hsrp_states.png)

Load balancing -> set each router as active for specific VLANs

Addressing:
* Version 1: 224.0.0.2:1985 UDP
* Version 2: 224.0.0.102:1985 UDP


## Routing Protocols

* Distance Vector: routes are advertised by providing 2 characteristics, distance and direction (algorithms: Bellman-Ford (RIP), DUAL (EIGRP))
* Link-state: creates a complete view of the network (topological map); routers exchange link-state packets which contain address of the link, type, cost, neighbors (algorithm: Dijkstra's shortest path algorithm)

Other things:
  * split horizon = do not send information on the interface it was received from
  * convergence time = time it takes a router to share info, calculate paths and update routing tables

### EIGRP
* Sends updates only on topology changes
* Sends keepalives
* Max hops: 255
* Mantains a topology table (for rapid convergence)

* Uses Diffusing Update Algorithm (DUAL)
* Establishes neighbor adjacencies
* Reliable Transport Protocol ( Transport layer protocol, for delivery of update packets)
* Partial and bounded updates (only on change and only to some routers)
* Equal and unequal load balancing (default on 4 links)
* Uses Protocol Dependent Modules (PDM) so it works for both IPv4 and IPv6 (+ others)
* Topology table contains all destinations advertised by the neighbors

**Packet types:**
* Hello packets (every 5 seconds)
* Update (propagate routing information)
* Acknowledgement
* Query
* Reply

**Exchanged at neighbor discovery:**
* Hello / Ack
* AS numbers (must match)
* Metrics (must match)

**EIGRP packet:**
* Header:
  * Optcode (type)
  * AS number (which routing process)
* TLV (Type, Length, Value)
  * Parameters (metric weights & hold time)
  * IP internal routes (advertised routes)
  * IP external routes (advertised routes from other protocols)

**EIGRP composite metric formula:**
-  metric = [K1 * bandwidth + K3 * delay] * 256  - 
-  metric = [K1 * bandwidth + (K2 * bandwidth)/(256 - load) + K3 * delay] * [K5/(reliability + K4)]

**Default values for weights:**
* K1 = 1 (bandwidth, slowest outgoing interface on path)
* K2 = 0 (load)
* K3 = 1 (delay, sum of delays on interfaces on path)
* K4 = 0 (reliability)
* K5 = 0 (MTU)

* Bandwidth = (10000000 / bandwidth)
* Delay = (delay / 10)

**DUAL:**
* Computes loop free routes
* Reported distance/advertised distance (RD / AD) = metric of a remote network as reported by a neighbor
* Feasible Distance (FD) = best metric to a remote network; RD of a neighbor + metric to neighbor
* Successor = neighboring router/next hop (best cost)
* Feasible Successors = backup path (must respect FC)
* Feasibility condition (FC) = the reported distance (RD) is lower than the current router's distance to the destination

* Neighbor table = keeps state information about neighbors
* Topology table = contains all destinations advertised by neighbors, the best path is put in the routing table

**Topology table:**

| //State// | //address// | //nr// successors, | FD is //nr// |

| ......... | //next hop//| ( //FD// / //RD// ), | //exit interface// |

**Finite State Machine (FSM):**
* Defines a set of possible states something can go through
* Like a flowchart

RTP multicast = send multicast, then unicasts to neighbours that did not acknowledge, will declare dead after 16 attempts

**Multicast addresses:**
* 224.0.0.10
* FF02::A

**EIGRP tuning:**
* Summarized updates are sent only out interfaces on different major classful networks
* EIGRP uses Null0 to avoid loops (router doing summarization adds the summary on a route to Null0, so if it receives a packet for a network it is not connected to, but the summary includes, it discard the packet)
* Hello time: interval between Hellos
* Hold time: maximum time for the neighbor to wait for Hellos
* For unequal cost load balancing use 'variance'. It shares load only if //metric of the route < metric of the best route * variance//

**Causes for no adjacency:**
* interfaces are down
* mismatching AS numbers
* interfaces not enabled for EIGRP
* interface is passive
* mismatching K values
* EIGRP authentication is misconfigured


### OSPF
* OSPFv2 -> IPv4
* OSPFv3 -> IPv6
* Uses Dijkstra's algorithm
* Link = interface on a router assigned to a network (has up/down states)

**Data structures:**
* Adjacency database: neighbor table
* Link-state database (LSDB): topology table
* Forwarding database: routing table
* Interface Table

**Operation:**
- Establish neighbor adjacency
- Exchange Link-state Advertisements (LSA) (describes a router and the network connected to it)
- Build topology table
- Execute SPF algorithm

An **area** is a group of routers that share the same LSDB.

**Packet types:**
- Hello packet (establish and maintain adjacency)
- Database Description packet(DBD) (abbreviated list of the sender's LSDB)
- Link-state request packet (LSR) (more info about an entry in the DBD)
- Link-state update (LSU) (reply to LSR)
- Link-state acknowledgement (LSAck) (confirm receipt of LSU)

Dead interval = time to wait for hellos

**Hello protocol contains:**
* Router ID (RID) 
* Hello/Dead interval*
* Neighbours
* Area ID*
* Priority
* DR IP address
* BDR IP address
* Authentication*
* Subnet mask*

* -> must match for neighbor adjacency 

**Neighbor states:**
* Down -> no Hellos received
* Attempt -> configure manually (on NBMA)
* Init -> Hello received, yet no bidirectional communication
* 2Way -> bidirectional communication established (own RID is in neighbor Hello)
* ExStart -> DR is elected
* Exchange -> routing info is exchanged using DBD (database descriptions)
* Loading -> Link-state requests (LSR) are sent for networks missed in the exchange
* Full -> LSA info is synchronized, OSPF routing can begin

Designated router = highest priority or highest router ID

Ethernet links are **multiaccess** !!

**Multicast addresses:**
* 224.0.0.5 (normal)
* 224.0.0.6 (to DR)
* FF02::5 (normal)
* FF02::6 (to DR)

**OSPF cost:**
* cost = reference bandwidth / interface bandwidth (in bps)
* cost (default) = 100000000 / interface bandwidth

* route cost = accumulated value from one router to destination
* reference bandwidth should be adjusted every time there are links faster than 100Mbps

**Adjacency requirements:**
* Two-way communication (hello protocol)
* Database synchronization
  * Database description
  * Link-state request
  * Link-state update

### Multiarea OSPF

* All areas are connected to the backbone (area 0)

**Types of routers:**
* Internal : all interfaces in the same area
* Backbone : at least 1 interface situated in the backbone
* Area Border Router (ABR) : has interfaces in multiple area
* Autonomous System Boundary Router (ASBR) : at least 1 interface attached to an external network

**LSA types:**
* Type 1: router link advertisement (RLA) -> sent by every router to routers in its area; contains RID, interfaces, IP info, current interface states
* Type 2: network link advertisement (NLA) -> sent by DR to inform about the state of non-DR routers; contains DR and BDR IP info
* Type 3: summary link advertisement (SLA) -> generated by ABR to advertise networks between areas; contains RID of ABR and IP info
* Type 4: sent by ABR, contains info about how to get to the ASBR
* Type 5: sent by ASBR, advertise routes that are external to the OSPF AS

**OSPF problems in multiaccess networks:**
* Creation of multiple adjancencies ( n(n-1)/2 )
* Extensive flooding of LSAs

On multiaccess networks elects designated router (DR) to collect and distribute LSAs. Routers only form full adjacencies with the DR and BDR (backup DR) and send their LSAs.

Election:
- Router with highest priority is DR
- If priorities are equal the router with highest ID is DR

To reelect DR restart interfaces or the OSPF process

Highest priority on a link is also the master (master-slave relationship), the router which sends the DBD first.

External routes types:
* Type 1: cost = addition of external and internal
* Type 2: external cost only

To load balance, set the cost to the same value on all links (on both ends)

## WAN Concepts
* WANs are owned by service providers
* Topologies:
  * Point-to-point
  * Hub-and-spoke
  * Full mesh
  * Dual-Homed
* Used for linking distant LANs


Terminology:
* Customer Premises Equipment (CPE): devices on the customer edge of the link
* Data Communications Equipment (DCE): devices that put data on the local loop; interface to subscribers connection (usually CSU/DSU)
* Data Terminal Equipment (DTE): devices that pass data from the customer network to the WAN provider (usually router interfaces)
* Demarcation point (the cable junction point): separates the user side from the provider side
* Local loop: the actual cable that connects the CPE to the CO
* Central Office (CO): service provider building that connects CPE to the provider network
* Toll network: the WAN provider network


* Access server = controls and coordinates dial-up modems
* CSU/DSU = for digital leased lines; CSU provides termination for the digital signal(+ error correction); DSU converts line frames into LAN frames; connects a DTE to a digital circuit; provides clocking


* Circuit switched network: establishes a dedicated circuit from start to end
* Packet switching: data is split into packets that are routed over a shared network

Fiber technologies:
* Synchronous Optical Networking (SONET) - US
* Synchronous Digital Hierarchy (SDH) - EU
* Dense Wavelength Division Multiplexing (DWDM)

### Bandwidth
* Digital Signal 0 (DS0) = 1 channel = 64Kbps
* DS1 = T1 = 24 DS0 circuits = 1.544 Mbps
* E1 = European equivalent = 30 DS0 = 2.048Mbps
* T3 = DS3 = 28 DS1 = 678 DS0 = 44.736 Mbps
* Optical carrier 3 (OC3) = 3 DS3 = 155.52Mbps
* OC-12 = 4 OC3 = 622.08 Mbps
* OC-48 = 4 OC-12 = 2488.32 Mbps
* OC-192 = 4 OC-48 = 9953.28 Mbps

### Connection types
* Dedicated (leased) lines = pre-established WAN path
* Circuit switching (ISDN) = creates circuit on demand
* Packet switching 

### Private WAN infrastructures
* Leased lines: T1/E1, T3/E3, permanent dedicated line
* Dial-up
* Integrated Services Digital Network (ISDN): uses telephone local loop to carry digital data
* Frame relay: non-broadcast multiaccess; uses data-link connection identifier (DLCI)
* Asynchronous Transfer Mode (ATM): uses fixed size cells (53B)
* Ethernet WAN
* MPLS (Multiprotocal Label Switching): uses labels to send frames to distant routers; can carry any payload
* VSAT: uses satellite communication

### Public WAN infrastructures
* DSL: uses telephone lines for high-bandwidth data
* Cable
* Wireless
* 3G/4G
* VPN: private connections over a public network

### Point-to-Point connections
WAN Protocols:
* HDLC: encapsulation type used on point-to-point connections and circuit switched when using Cisco devices
* PPP: router-to-router & host-to-network connection based on HDLC + security
* SLIP: protocol for point-to-point serial connections using TCP/IP (replaced by PPP)
* X.25/LAPB: standard for DTE and DCE connections, predecesor to Frame relay
* Frame relay: data-link layer protocol that handles multiple virtual circuits
* ATM: cell relay

![](/static/images/hdlc_frame.png)

### PPP
Used when connection non-Cisco routers (and not only).

Components:
* HDLC-like framing for transporting multiprotocol packets
* Extensible Link Control Protocols (LCP) for establishing, configuring connection
* Network Control Protocols (NCP) for establishing and configuring network layer protocols (allows multiple at the same time)
* Link Quality Management: monitors quality of the link

PPP LCP:
* data-link layer
* establishes, configures, tests the point-to-point data-link connection
* handles automatic configuration of interfaces
* authentication, compression and others

![](/static/images/lcp_operation.png)

PPP NCP:
  * every L3 protocol uses a different NCP (IPv4 -> IPCP, IPv6->IPv6CP)
  * method of configuring L3 protocols; L3 protocols establish services with PPP

![](/static/images/ncp_operation.png)

PPP frame:

| Flag(beginning of data) | Address (all 1s) | Control | Protocol | Data | FCS |

Establishing connection:
* Phase 1: Link establishment & configuration negotiation
* Phase 2: Link quality determination (optional)
* Phase 3: Network layer protocol configuration negotiation

**LCP operation**

LCP frame types:
* Link-establishment frames
* Link-maintnance frames
* Link-termination frames

* Code-reject frame -> when receiving an invalid frame

**NCP operation**

IPCP example negotiations:
* Compression: negotiating an algorithm to compress TCP/IP headers
* IPv4 address

**PPP configurations**

Options:
* Authentication (PAP & CHAP)
* Compression (Stacker & Predictor)
* Error detection
* PPP callback
* Multilink (provides load balaning, fragments packets and sends them over multiple links)

**PPP authentication**

PAP (Password Authentication Protocol)
* sends user and password in plain text
* remote router sends credentials unrequested

CHAP (Challenge Handshake Authentication Protocol)
* after link establishment the local router sends a challenge to the remote one
* remote router responds with a hash and username
* if the password is correct it accepts the authentication
* the challenges repeat with variable values over time

PPP putes the neighbor's IP address in the routing table as a connected interface.

### Broadband connections
**Cable**
* Data over Cable Service Interface Specification (DOCSIS) - international standard for cable
* 2 types of equipment:
  * Cable modem termination system (CMTS): exchanges digital signal with modems (at the headend)
  * Cable modem (CM) (at the client
* 500-2000 active subscribers for a cable segment

**Digital Subscriber Line (DSL)**
* Provides high-speed internet over installed copper wires
* DSL Access Multiplier (DSLAM): concentrates connections from multiple subscribers

**Wireless**
* Municipal WiFi (mesh network)
* Cellular
* Satellite
* WiMax


### Point-to-Point Protocol over Ethernet (PPPoE)

* appeared because of PPP's authentication, accounting and link-management features and the ease of use of Ethernet
* encapsulates PPP frames in Ethernet frames

In a TCP 3-way handshake, the TCP maximum segment size (MSS) is negotiated. It is 'Ethernet MTU - TCP/IP headers), by default 1460. On PPPoE, the frame MTU is only 1492 (opposed to 1500) because of the size of the PPPoE header, so the MSS must be 1452.

![](/static/images/pppoe-mtu.png)

## VPN
  * A private network created via tunneling over a public network

Adaptive Security Appliance (ASA) = combines firewall, VPN concentrator and intrusion prevention functionality

Types of VPN:
* Site-to-site: connects entire networks, traffic goes through VPN gateways
* Remote access: a single device gains access to a private network; client-server
* Dynamic Multipoint VPN (DMVPN): creates spoke-to-spoke tunnels automatically;  uses NHRP, mGRE, IPSec.
* Cisco Virtual Tunnel Interface (VTI): simplifies VPN configuration

Provider managed VPN:
* Layer 2 MPLS VPN:
  * Uses MPLS labels to transport data
  * Virtual Private Wire Service (VPWS) - Ethernet over MPLS
  * Virutal Private LAN Switching Service (VPLS)
* Layer 3 MPLS VPN
  * Exchange routes with service provider routers

## IPSec
Framework that allows secure data transmission.

IPSec transforms = specifies a single security protocol

* Authentication Header (AH)
  * Authentication for data and IP header
  * Sender generates a hash and the receiver generates the same hash, they must match
* Encapsulating Security Payload (ESP)
  * Encryption: symmetric encryption of the packets
  * Data integrity: checksums
  * Authentication: ensures that the connection is made with the correct partner
  * Anti-replay service: uses sequence numbers to stop replay attacks (transmitting a packet)
  * Traffic flow: offers confidentiality

Encryption:
* Symmetric: DES, 3DES, AES
* Asymmetric: public key is used to encrypt a symmetric key; private key encrypts a hash to create a digital signature

### Generic Routing Encapsulation (GRE)
* Encapsulates a wide variety of packet types in IP tunnels
* It is non-secure

## Border Gateway Protocol (BGP)
* Exchange routing information between ASs
* Every AS has a 16 or 32 bit AS number
* Path vector routing protocol (exchanges the list of ASs to cross to a destination)
* Uses port 179
* Advertised routes include: the network, a list of attributes, next-hop address, list of ASs through which the update has passed
* Attributes influence best path selection

* Use BGP when the AS has connections to multiple ASs (multi-homed)

Ways to implement BGP:
- Default route only: ISP only advertises a default route to the organization
- Default route and IP routes: ISP advertises default route and their network
- All Internet Routes: ISP advertises all routes, results for best path for any network (over 550000)

## ACLs
* Standard ACL: filters based on source address
* Extended ACL: filters based on protocol type, source/destination address, source/destination port, optional protocol type information

Place:
* Standard ACLs as close to the destination
* Extended ACLs as close to the source

* Extended ACL numbers: 100-199, 2000-2699

Form:  access-list 101 [permit|deny] [protocol] [source] [destination] [port]
![](/static/images/extended_acl.png)

You can use logical operations for ports: eq, neq, gt, lt.

**IPv6 ACLs**
* Only named ACLs
* Similar to extended ACLs
* 2 implicit permit statements and the deny statement at the end:
  * permit icmp any any nd-na
  * permit icmp any any nd-ns
![](/static/images/ipv6_acl.png)

## Security
Common LAN attacks:
  * CDP reconnaisance attack: get information about devices
  * Telnet attack: brute force or DoS
  * MAC Address Table flooding attack: When the MAC address table is full, the switch enters fail-open mode and broadcasts all frames
  * VLAN attacks: spoofing a trunk link with the switch (using DTP)
  * DHCP attacks: server spoofing and server request flooding (starvation)

### Best practices
* Always use secure protocols (SSH, SCP, SSL, SNMPv3, SFTP)
* Use strong passwords
* Enable CDP only on selected ports
* Secure Telnet access
* Use a dedicated management VLAN
* Use ACLs to filter unwanted access

* IP Source Guard (IPS): prevents MAC and IP address spoofing
* Dynamic ARP Inspection (DAI): prevents ARP spoofing
* DHCP Snooping: prevents DHCP attacks
* Port Security: prevents CAM table overflow and attacks
* Identity-based networking: when you connect to a switch and authenticate, you will be put in your needed VLAN (802.1X)

DHCP Snooping:
* enabled on switch interfaces
* denies packets coming from untrusted ports(that do not lead to a DHCP server) or client messages not adhering to the Binding Database or rate limits

### Authentication, Authorization and Accounting (AAA)
* router accesses a central AAA server that holds all usernames and passwords
* uses TACACS+ or RADIUS

**802.1X**
* Port based access control and authentication protocol
* Restricts workstations from connecting to LAN
* Roles:
 * Supplicant (client): requests access to the LAN
 * Authenticator (Switch): controls physical access, acts as proxy between client and authentication server, requests information from client
 * Authentication server: validates identity
* Uses RADIUS

**RADIUS**
* After user sends name and password, the server replies with:
  * Accept
  * Reject
  * Challenge
  * Change password

**TACACS+**
* Cisco developed
* more complex

## Simple Network Management Protocol (SNMP)

3 elements:
* SNMP manager
* SNMP agent
* Management Information Base (MIB): collection of information organized hierarchically

Operation:
* Manager requests:
  * get (query info)
  * set (change config)
  * walk (list information from successive MIB objects)
* Agent traps:
  * trap (unsolicited messages alerting the manager to an event)
  * inform (like a trap, but with acknowledgement)

SNMPv3 uses authentication and encryption.

**MIB Object ID**
* hierarchical design (like DNS)
* each variable has an OID

ex: //.iso.org.dod.internet.private.enterprise.cisco// (1.3.6.1.4.1.9)

## Switch Port Analyzer (SPAN)
Port mirroring = copy and send a packet throuch more than 1 port

Local SPAN -> traffic on a switch is mirrored on another port on that switch

Remote SPAN -> source and destination ports are on different switches

Source port = ports whose traffic is mirrored

Destination port = destination port for copied packets

SPAN session = association of source and destination ports

RSPAN VLAN = unique VLAN to transport SPAN traffic

## Quality of Service
Congestion points:
* Aggregation
* Speed mismatch
* LAN to WAN

delay/latency = time it takes for a packet to travel to the destination

Playout delay buffer = helps eliminate jitter (variable delay); buffers packets and plays them at a steady pace

Trust boundary = where packets are classified and marked; lies between the ingress interface of the device trusting the marking and the egress interface of the device marking the traffic

## Queueing Algorithms
- FIFO -> no QoS
- Priority Queuing (PQ): low priority queues are serviced only when high priority are empty
- Custom Queuing (CQ): round robin
- Weighted Fair Queuing (WFQ)
  * Applies priority or weights to traffic
  * Determines how much bandwidth each flow needs
  * Uses packet header addressing
  * Not supported with encryption
- Class-Based WFQ (CBWFQ)
  * Matches traffic to classes based on protocols, ACLs and interfaces
  * You assign bandwidth, weight and packet limit to classes
- Low Latency Queuing (LLQ)
  * Adds strict priority queueing to CBWFQ
  * Allows delay sensitive data to be sent before other packets

## QoS models
* Best-effort -> no QoS
* Integrated services:
  * uses end-to-end signaling and resource reservation
  * connection oriented
  * applications request a service from the network
  * uses Resource Reservation Protocol (RSVP)
* Differentiated services
  * routers identify flows and provide appropriate QoS for each class
  * configured on each router

Dropped TCP segments couse TCP to reduce window size.

**QoS tools**
* Classification and marking
  * packets are marked based on class
  * add a value to the header
  * Class of service: Layer 2, uses 3 bits
  * IP precedence: Layer 3, uses 3 bits
  * At layer 3: 8 bits (Type of Service/Traffic Class)
    * DSCP (6b): best effort (no QoS), Expedited forwarding (for voice), Assured forwarding (uses class and drop preference), backwards compatible with IP precedence
    * ECN (2b): used to inform routers of congestion
  * Traffic Identifier (TID)- Layer 2, used by WiFi

![](/static/images/qos_marking.png)
![](/static/images/dscp.png)

* Congestion avoidance
  * Queueing and scheduling
  * Weighted Random Early Detection (WRED) -> manages congestion
* Shaping and Policing
  * Shaping = retains packets in a queue for later transmission for a smooth output rate
  * Policing = excess traffic is dropped

## Internet of Things
  * Machine-to-machine communication (M2M)

IoT Pillars:
* Network connectivity: identifies devices that can be used to connect IoT devices
* Fog computing: distributed computing closer to the network edge (locally) for IOT
* Security:
  * Operational Technology (OT) specific technology
  * IOT Network security
  * IOT physical security
* Data Analytics
* Management & Automation
* Application Enablement Platform:
  * Infrastructure for application hosting and mobility between fog and cloud
  * IOx = IOS + Linux

## Cloud services
* Software-as-a-service (SaaS): applications delivered over the network
* Platform-as-a-service (Paas): developmentt tools delivered over the network
* Infrastructure-as-a-service (IaaS): access to virtualized hardware
* IT-as-a-Service (ITaaS)

Cloud models:
* Public cloud: services offered to the general population
* Private cloud: cloud specific to an organization
* Hybrid cloud
* Community cloud: public clouds for a specific community (ex: medical)

## Virtualization
* Hypervisor = program that adds an abstraction layer on top of real hardware

* Type 1 hypervisor = virtualization server; runs directly on hardware
* Type 2 hypervisor = runs on top of an OS

* East-West traffic = traffic flow between the virtual servers

## Software Defined Networking (SDN)
* Control plane = makes forwarding decisions (protocols, tables)
* Data plane = the switch fabric connecting the ports; decisions made by special processors (DSP); physically responsible for forwarding frames
* SDN = control plane is removed from the device and is done by a centralized server
* Application plane = added by SDN; contains applications that communicate network requirements to the controller using APIs
* SDN controller = logical entity (virtualized) that manages how the data plane handles traffic (uses APIs)

* Cisco Application Centric Infrastructure (ACI) = hardware solution for integrating cloud computing
* OpenFlow = manages traffic between devices and a controller
* OpenStack = virtualization and orchestration platform (automates provisioning of devices)

## APIs
* Southbound APIs:
  * used for communication between controllers and network devices
  * OpenFlow, NETCONF, OnePK, OpFlex
* Northbound APIs:
  * communication between SDN controller and applications

## Controllers
* define the data flows (packets with common header info) in the data plane
* computes routes for flows (adds entries to switching flow tables)
* on the switches, flow tables in hardware or firmware defines the action to be taken on the flow

## Cisco Application Centric Infrastructure ====
**Components:**
* Application Network Profile (ANP): end point groups (EPG) (VLANs, web servers, apps) and their connections and policies
* Application Policy Infractructure Controllers (APIC): centralized software controller; translates app policies into network programming
* Nexus 9000 switches

## Types of SDN
* Device based SDN: devices are programmed by apps running on themselves or on a server (ex: Cisco OnePK)
* Controller based SDN: centralized controller that has knowledge of all devices (OpenDaylight)
* Policy-based SDN: includes a policy layer; uses user friendly GUI (APIC-EM)

## APIC-EM
* Is a SDN controller
* Provides automation of policy based application profiles
* southbound interfaces using service abstraction layer (SAL) -> uses SNMP and CLI (soon NETCONF)
* Collects info about all devices
* Creates topology maps
* Monitors traffic flows
* Manages ACL policies

## Network Troubleshooting
### Documentation
* configuration files(network and end-system)(tables with: device name, OS, IP address, subnet mask, gateway, DNS, important apps)
* physical and logical topology
* baseline performance levels

**Baseline:**
- Determine what types of data to collect (utilization, CPU ...)
- Identify devices and ports of interest
- Determine baseline duration

### Troubleshooting procedures
- Gather symptoms
- Isolate the problem
- Implement corective action (implement, test and document solutions)

If an OSI layer is working, the layers under are working.

## IP Service Level Agreement (IP SLA)
* Used for monitoring
  
![](/static/images/sla_sched_command.png)

## Troubleshooting tools
Software:
* Network Management System Tools
* Knowledge bases
* Baselining Tools
* Protocol Analyzers

Hardware:
* Multimeters
* Cable testers
* Cable analyzers
* Network Analysis Module (NAM)
