# CCNA IOS Commands

## Initial configuration
```
configure terminal
service password-encryption
enable secret [pass] # Privileged access password

interface [type] [slot]/[port]
line console 0   # Console connection
line vty 0 15   # All 16 virtual remote terminals
password [pass]
logging syncronous   # Optional
login

banner motd " [message] "
no ip domain-lookup

# Switch remote config IP
interface vlan 1
ip address [ip] [mask]
no shutdown
exit
ip default-gateway [ip]
```

## Show commands

```
show running-config
show ip interface brief
show arp
show switch     # switch info
show mac address-table
show ip arp # On routers
show ip route
show version
show sessions
show interfaces
show history
show terminal
show protocols
show cdp entry * protocol
show cdp neighbors
show ip protocols
show boot
show debug
show controller ethernet-controller [int] physical
show ssh
show ip ssh
show vlan brief
show vlan
show interfaces trunk
show interface [int] switchport
show port-security interface [int]
show port-security address
show controllers [int]   #L1 info
show access-lists
show ip dhcp conflict
show ip dhcp binding
show ip dhcp server statistics
show ipv6 dhcp pool|binding
show ip nat translations|statistics  (verbose)
show cdp
show cdp neighbors (detail)
show lldp
show lldp neighbors (detail)
show clock (detail)
show logging
show file systems
show license feature
show license udi
show license
show users
show hosts
show ipv6 neighbors

show vtp status | password
show spanning-tree (summary)
show etherchannel summary|port-channel
show interfaces [int] etherchannel
show standby (brief)
show ip eigrp neighbors
show ip protocols
show ip eigrp topology
show ip eigrp interfaces (detail|brief)
show ip ospf
show ip ospf neighbor
show ip ospf interface (brief| [int])
show ip ospf database
```

## Router config

```
(in conf interface)
ip address [ip] [mask] (secondary)
no shutdown   # Turns on interface

# For serial interfaces
clock rate [nr]
bandwidth [nr]

# For IPv6
ipv6 enable   # Optional
ipv6 unicast-routing   # Activate
ipv6 address [ip]/[prefix]
ipv6 address [address]/[prefix] link-local

# For VLANs
interface [int_id].[subinterface_id]
encapsulation dot1q [vlan_id] (native)
```

## Switch config

```
boot system flash:/[path]/[name].bin

(in interface)
duplex full/half/auto
speed 10/100/1000/auto
mdix auto

## VLANs
vlan [id]
name [vlan name]
interface [int_id]
switchport mode access
switchport access vlan [id]

switchport mode trunk
switchport trunk native vlan [id]
switchport trunk allowed vlan [ids] | all | remove [nr]-[nr]
switchport trunk encapsulation dot1q | isl

switchport mode dynamic auto   # converts to trunk if the other end is trunk or desirable
switchport mode dynamic desirable   # converts to trunk if the other end is trunk, auto, desirable
switchport nonegotiate

mls qos trust [cos|device cisco-phone|dscp|ip-precedence]
switchport voice vlan [id]

# Layer 3 switch VLAN config
ip routing
interface vlan [nr]
ip address [addr] [mask]
```

## Security

```
auto secure
security password min-length [nr]
login block-for [seconds] attempts [nr] within [mins]
exec-timeout [mins] [seconds]   # Logs you off of console

interface loopback [nr]

# SSH config
hostname [name]
ip domain-name [domain]
crypto key generate rsa   # Delete with 'crypto key zeroize'
ip ssh version 2
username [user] secret [pass]
line vty 0 15
transport input ssh
login local

## Switch security
switchport port-security
switchport mode access
switchport port-security mac-address [addr]   # Only allows this address on that interface
switchport port-security mac-address sticky   # Remembers the learned address on the interface

switchport port-security violation protect|restrict|shutdown
switchport port-security maximum [nr]
```

## Routing protocols

```
# Static routing
ip route [network] [mask] [exit int] [next hop] ([Administrative dist])
ipv6 route [network]/[prefix] [exit int] [next hop] ([Administrative dist])

# RIP
router rip
version 2
network [addr]   # Connected network

no auto-summary
passive-interface (default) [int] (don't send updates on that interface)
default-information originate (advertise default static route)
```

## Access lists

```
# Standard
access-list [nr] deny|permit|remark [source] [wildcard] (log)

# Extended
acces-list [nr] permit|deny [protocol(tcp, udp, ip)] [source_addr] [dest_addr] eq [port]

ip access-list standard|extended [name]
permit|deny|remark [source] [wildcard] (log)

# On interface
ip access-group [nr|name] in|out

# On vty
access-class [nr] in|out

# On access list, for changing statements
no [nr]
[nr] statement

clear acces-list counters [nr]

## IPv6 ACL
ipv6 access-list [name]
permit|deny [protocol] [source/prefix] [dest/prefix] eq [port]    # see image for more info

#on int
ipv6 traffic-filter [name] in|out
#on vty
ipv6 access-class [name] in|out
```

## DHCP

```
ip dhcp excluded-address [low_addr] [high_addr]
ip dhcp pool [name]
network [addr] [mask]|[prefix]
default-router [addr]

dns-server [addr]
domain-name [name]
lease [days] ([h][m] | infinite)
option 66 ascii [domain]  (tftp server)

no service dhcp  # disable DHCP

# On interface
ip helper-address [srv_addr]  # Relays broadcasts to the addr

# As client
ip address dhcp

debug ip packet [acl_nr]
debug ip dhcp server events

## For DHCPv6
(no) ipv6 nd managed-config-flag  # M flag
(no) ipv6 nd other-config-flag    # O flag

# Stateless DHCPv6
ipv6 dhcp pool [name]
dns-server [addr]
domain-name [addr]
interface [int]
ipv6 dhcp server [name]
ipv6 nd other-config-flag

# Set router as client
ipv6 enable
ipv6 address autoconfig | dhcp

# Stateful DHCPv6
ipv6 unicast routing
ipv6 dhcp pool [name]
address prefix [addr]/[prefix] lifetime [secs]|infinite
dns-server [addr]

# On int
ipv6 dhcp server [name]
ipv6 nd managed-config-flag

ipv6 dhcp relay destination [srv_addr]
```

## NAT

```
#Static NAT
ip nat inside|outside source static [local] [global]

#On interface  (for all types of NAT)
ip nat inside
ip nat outside

# Dynamic NAT
ip nat pool [name] [start] [end] netmask [mask]
access list [nr] permit [source] [wildcard]     # The inside addresses allowed to be translated
ip nat inside source list [acl_nr] pool [name]

# PAT
ip nat pool [name] [start] [end] netmask [mask]
access-list [nr] permit [source] [wildcard]
ip nat inside source list [acl_nr] pool [name] overload

# For a single outside address, no pool nedeed
ip nat inside source list [acl_nr] interface [int] overload

# Port forwarding
ip nat inside source static tcp|udp [local_ip] [local_port] [global_ip] [global_port] (extendable)
ip nat inside
ip nat outside

ip nat translation max-entries
ip nat translation timeout

clear ip nat statistics|translations
clear ip nat translations *
clear ip nat translations inside [global] [local]
clear ip nat translations [protocol] inside [global_ip] [global_port] [local_ip] [local_port]
debug ip nat (detailed) ( * means that it's fast switched)
```

## CDP/LLDP

```
show cdp
cdp run  # Enable CDP
no cdp enable   # Disable CDP on an interface
show cdp neighbors (detail)

cdp holdtime [sec]
cdp timer [sec]

# On interface
lldp transmit
lldp receive
show lldp
show lldp neighbors (detail)
show cdp entry * protocol
```

## NTP

```
ntp server [address]

show clock (detail)
show ntp associations
show ntp status

clock timezone [ABC] [UTC offset]
clock summer-time [ABC] reccuring
```

## Logging

```
service timestamps log datetime

logging console
logging buffered
show logging

logging [ip_addr]   # For syslog server
logging trap [max_level]
logging source-interface [int]
service sequence-numbers
```

## IOS

```
show file systems
dir
cd [filesystem]
copy [file] tftp:   # You can replace tftp with any filesystem
copy tftp: [local_file]

## To reset password
# In ROMMON
confreg 0x2142
reset
enable
copy startup-config running-config
enable secret [password]
config-register 0x2102
copy running-config startup-config
reset

boot system flash0://[file_name]

show license udi
show license feature

license install [file_location]

# Right-to-Use license
license accept end user agreement
license boot module [name] technology-package [name]

license save [location]

# Remove a license
license boot module [name] technology package [name] disable
license clear [feature_name]
no license boot module [name] technology package [name] disable
```

## VTP & DTP

```
show vtp status | password

#configure VTP server
vtp mode server
vtp domain [name]
vtp password [pass]

#configure VTP clients
vtp mode client
vtp domain [name]
vtp password [pass]

#vtp pruning
switchport trunk pruning vlan [IDs]

#add VLANs
vlan [nr]
name [name]

no vlan [nr]  # delete


show dtp interface [int]
#see photos

```

## Layer 3 switching

```
no switchport   # activate router port, after this assign IP address like on a router

interface vlan [nr]
ip address [addr] [mask]

ip routing     # activate routing
```

## STP

```
(on int)
spanning-tree cost [nr] 

spanning-tree portfast
spanning-tree portfast default    # enables automatically on non-trunk ports

spanning-tree bpdguard enable   # on interface
spanning-tree portfast bpdguard default   #in global config

spanning-tree link-type point-to-point|shared    # p-t-p for full duplex, shared for half duplex

#configure BID
spanning-tree vlan [id] root primary|secondary
spanning-tree vlan [id] priority [nr]    # increments of 4096

spanning-tree mode [mode]

show spanning-tree (summary)


# Rapid PVST
spanning-tree mode rapid-pvst
int [id]
spanning-tree link-type point-to-point (|shared)

clear spanning-tree detected-protocols
```

## Etherchannel

```
#configuration
interface range [type][nr]-[nr]
channel-group [nr] mode active
interface port-channel [nr]
switchport mode trunk
...

show etherchannel summary|port-channel
show interfaces [int] etherchannel
```

## HSRP

```
show standby (brief)  # verify HSRP state


# HSRP config (on int, after IP address configured)
standby version 2
standby ([group]) ip [addr]            # Configures virtual IP address; if no group it becomes 0
standby ([group]) priority [0-255]     # If no priority, router with highest IP elected
standby ([group]) preempt              # Trigger HSRP reelection
standby ([group]) timers msec [hello] msec [hold]   # on int

debug standby
```

## EIGRP

```
#config
router eigrp [as_nr]
eigrp router-id [ipv4_addr]
network [addr] ([wildcard_mask])

# alternative to AS number
router eigrp [word]
address-family ipv4 autonomous-system [nr]

passive-interface [int] (|default)

(ipv6 for IPv6)
show ip eigrp neighbors
show ip protocols
show ip eigrp topology (all-links)
show ip eigrp interfaces (detail|brief)
show ip eigrp traffic
show ip eigrp events

metric weights [tos] [k1] [k2] [k3] [k4] [k5]

# For IPv6
ipv6 router eigrp [as_nr]
router-id [32 bit nr]
no shutdown
(on int)
ipv6 eigrp [AS_nr]

auto-summary    # not recommended
ip summary-address eigrp [as_nr] [address] [mask]  # summary; on interface

redistribute static

ip bandwidth-percent eigrp [AS] [percent]    # on int
ip hello-interval eigrp [AS] [seconds]
ip hold-time eigrp [AS] [seconds]

# add neighbours manually
neighbor [address]

distribute-list [acl] in|out    # add ACL

maximum-paths [nr]
variance [nr]

debug eigrp fsm
eigrp log-neighbor-changes
```

## OSPF

```
# configuration
router ospf [proc_id]
router-id [id]   # optional
network [address] [wildcard] area [area_id]    #wildcard can be 0.0.0.0 if address if exact interface address, not network

#alternative for network statements
(on int) ip ospf [proc_id] area [nr]

passive-interface [int]

clear ip ospf process

auto-cost reference-bandwidth [Mb/s]

# on int
ip ospf cost [cost]
ip ospf priority [value]

# ipv6 for OSPFv3
show ip ospf
show ip ospf neighbor
show ip ospf interface (brief| [int])
show ip ospf database

area [source_area] range [summary addr] [mask]    # for manual summarization
default-information originate (always)

# on int
ip ospf hello-interval [secs]
ip ospf dead-interval [secs]
# no [something] to restore default
#intervals must match for adjacency

# OSPFv3 config
ipv6 unicast-routing
ipv6 router ospf [proc_id]
router-id [id]
# on int
ipv6 ospf [proc_id] area [area_id]
```

## HDLC

```
interface [int]
encapsulation hdlc

show controllers   #shows cable types
```

## PPP

```
encapsulation ppp    #on int
compress predictor | stac
ppp quality [percentage]   # if error > % link is taken down

#multilink - spread traffic across multiple links
interface multilink [nr]
// configure ip address
ppp multilink
ppp multilink group [nr]
// on interfaces 
no ip address
ppp multilink
ppp multilink group [nr]

show ppp multilink

#authentication
#pap
username [remote_hostname] password [pass]     #on global
ppp authentication pap
ppp pap sent-username [local-name] password [pass]

#chap
username [remote_hostname] password [pass]    # password must be the same on both sides
ppp authentication chap (callin)    #callin doesn't challenge the router you connect to

#hostname is sent as username
#you can enter multiple username/password pairs

debug ppp packet|negotiation|error|authentication|compression|cbcp
```

## PPPoE

```
interface dialer [nr]
encapsulation ppp
ip address [addr][mask] | negotiated
ppp chap hostname [name]
ppp chap password [pass]
ip mtu 1492
dialer pool [nr_pool]
no shutdown

interface [physical_int]
no ip address
pppoe enable
pppoe-client dialer-pool-number [nr_pool]
no shutdown

show pppoe session
ip tcp adjust-mss 1452
```

## GRE

```
interface tunnel[nr]
tunnel mode gre ip
ip address [tunnel_addr][mask]
tunnel source [real_addr]
tunnel destination [real_addr]
#you must configure a routing protocol or static routing
```

## BGP

```
router bgp [as_nr]
neighbor [ip] remote-as [as_nr]
network [addr] mask [mask]

show ip bgp (summary | neighbors)
```

## SNMP
```
#SNMPv2
snmp-server community [string] ro|rw ([ACL])    #the string is the password
#optionals
snmp-server location [text]
snmp-server contact [text]
snmp-server host [id] version 1|2c|{3 auth|noauth|priv} [community_string]
snmp enable traps ([trap])

#SNMPv3
// configure standard ACL to permit managers
snmp-server view [name] [oid-tree]
snmp-server group [group_name] v3 priv read [view_name] access [acl]
snmp-server user [username] [group_name] v3 auth md5|sha [password] priv des|3des|{aes 128|192|256} [priv_pass]

#oid tree example: 'iso included'

show snmp (community)
```

## RADIUS & TACACS+

```
#RADIUS config
aaa new-model
username [name] password [pass]
radius server [s_name]
address ipv4 [ip]
key [password]

aaa group server radius [g_name]
 server name [s_name]

aaa authentication login default group [g_name] local   # the local at the end is the backup authentication

#TACACS+ config
tacacs-server [host]
// incomplete
```


## SPAN

```
monitor session [nr] source {interface [int] | vlan [nr]}
monitor session [nr] destination {interface [int] | vlan [nr]}

show monitor
```

## IP SLA

```
show ip sla application | configuration | statistics
ip sla [operation_nr]
icmp-echo [dest] (source-ip [addr] | source-interface [int])
ip sla schedule [op_nr] start-time now life forever
```

## Misc

```
enable
disable
logout
reload

(Ctrl + A to negate a command)

telnet [addr]
show sessions
(type connection number to return to it)
show users
disconnect [nr]

# Create host translation
ip host [name] ([port]) [addr]
show hosts

ip domain lookup
ip name-server [addr]
ip domain-name [domain]

copy running-config startup-config
erase startup-config
delete flash:[file]
clear counters

privilege level 15   # on line con or vty to enter directly to privileged mode
vlan database        # configure VLANs on etherswitch or old switches

interface [type number]
interface range [type][first] - [last]
description [text]

no ip routing   #disable routing on L3 switches/Etherswitches

ping
traceroute

clear contents
debug
terminal monitor   # Show logs on vty
show processes (cpu)
show memory

dir/copy/more/show file/delete/erase/format/cd/pwd/mkdir

# | pipe used after show with parameters: section, include, exclude, begin
show ip interface brief | include up

# use 'no' keyword to delete VLANs, ip routes, IP addresses on interfaces, etc.
```
