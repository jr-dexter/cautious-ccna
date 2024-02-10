## Base Switch Configuration
```
hostname ${hostname}

#Convenience Commands
no ip domain-lookup
logging synchronous

```

### Configure Management Address
```
interface vlan 10
ip address ${ip-address} ${subnet-mask}
no shutdown
exit
ip default-gateway ${default-gateway}
```

### Switch to Host Config
```
interfce gigabitEthernet1/0/1
description ${int-description}
switchport mode access
no shutdown
```

### Switch to Switch Config
```
# Sw1
interfce gigabitEthernet1/0/2
description ${int-description}
switchport mode dynamic desirable
no shutdown

#Sw2
interfce gigabitEthernet1/0/2
description ${int-description}
switchport mode dynamic desirable
no shutdown
```
### Configure a range of interfaces
```
interface range gigabitEthernet0/0/1-10
switchport mode access
no shutdown
```
### Verification
```
show ip interface brief
show running-configuration
show version
show mac address-table
show mac address-table address ${MAC-address}
show mac address-table dynamic
show mac address-table interface
```

## VLAN
```
vlan ${vlan-id}
name ${vlan-desciption-name}
```

### VLAN interface
```
#Sw2
interfce gigabitEthernet1/0/2
description ${int-description}
switchport mode access
switchport access vlan 12
no shutdown
```

## VLAN Trunk
```
# Change default VLAN
switchport trunk native vlan ${vlan-id}

# 802.1q
interface gi1/0/1
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan {add | all |except | remove } ${vlan-list}

```

### Verification
```
show vlan brief
show interface gi0/1 switchport
show interface trunk
show interface ${interface} switchport
```

## DTP
 - desirable (initiates the trunk)
 - auto (passive listening)
 
### Configure
```
switchport mode dynamic [dynamic|auto]
```
### Verification


## CDP
### Configure
```
cdp run
cdp time ${interval} # at which HELLO messages are sent
```
### Verification
```
show cdp neighbor
show cdp neighbor ${interface}
show cdp neighbor ${interface} detail
```

## LLDP
### Configure
```
lldp run
lldp holdtime 150
lldp timer 15
interface ethernet 0/0
lldp transmit
```

### Verification
```
show lldp traffic 
show lldp neighbors
```

# LAB: IOS CLI Basics

# EtherChannel
## PAgP (Cisco Propiety)
- On : forces the channel
- Desirable: Sends PAgP init messages
- Auto: Passively listens 

## LACP
- IEEE 802.3ad
- On: forces the channel
- Active: Sends LACP init messages
- Passive: Passively listends for LACP Requests

# Routes
As a network administrator, you want to enable IOS debugging commands but are concerned those commands could overwhelm the console. Instead you wish to allow all SYSLOG messages to be displayed on the console _but_ 
to have debugging output prevented from being displayed on the console and stored in a memory buffer instead. Which of the configurations below will accomplish this? 
```
Device#config t 
Device(config)#no logging console 
Device(config)#logging console 6 
Device(config)#logging buffer debug
```

As a network administrator, you become irritated when you are frequently in the middle of typing IOS configuration commands and are interrupted by SYSLOG messages. You wish to disable these SYSLOGS and have them just stored in memory for future viewing. Which of the configurations below will accomplish this?
```
Device#config t 
Device(config-line)#no logging console 
Device(config-line)#logging buffer
```

## Learning Routes

| Protocol | Admin Distance Value | 
|----------| -------------------- |
| Connected     |   0                 |
| Static        |   1                 |
| EIGRP (Int)   |   90                |
| OSPF          |   110               |   
| IS-IS         |   115                 |
| RIP           |   120                 |
| EIGRP (Ext)   |    170         |
| iBGP/eBGP     |   200/20              |
| Unreachable   | 255                 |

### Metrics
 1. Administrative distance is used between differenct routing protocols
 2. Metrics are used between same protocol

 - Used for best path selection process
 - IGPs use metrics for shortest path calculation
 - lower values is preferred
 - Depends on Routing Protocol Architecture
    - EIGRP = (DISTANCE) link bankdwidth + delay
    - RIP = hop count
    - OSPF = (COST) link bandwidth

## Contasting Routing Protocols

### Distance Vector
|  Protocol  |    Metric   | Neighbor Reqs      | Route Maintenance           | Visibility to topology   | route mgmt data structure   |
|  RIP / IGRP |  Distance  | None               | Resend routes periodically  | Directly connected routers  | db of learned routes |
| OSPF / IS-IS     | Link State  | Required     |  periodic keep-alives / regenerate LSAs | complete topology | db of link state, neighbor table , SPF (shortest path first) Tree |
| EIGRP       | Advanced Distance Vector | Required |periodic keep-alives | directly connected routers | topology of learned routes / neighbor table | 
| BGP      | Path Vector | Required/Exterior | Period keep-alives | no knowledge of topology- only next hop | neightbor table / bgp table | 


# OSPF
- Open Standard
- Shorted path first algorithm
- Hello used for neighbor relationship
    - Hello = 10 seconds
    - Hold = 40 seconds
- Works based on area heirarchy
- Minimized LSA flooding
    - Type 1 Router LSA
### ABR (Area Border Router) Router that splits areas 
 - Metric = cost
 - 100 Million / Interface Bandwidth in bps
### OSPF Packet Types
 - Hello
 - Database Descriptor (Sync check)
 - Link State Request (missing LSA)
 - Link State Update ( added to db)
 - Link State Acknowledgement (Syncnonized)
- PACKETS ARE MULTICAST
- OSPFv2: 224.0.0.5 / 224.0.0.6(DR/BDR UPDATE Address)
- OSPFv3: ff02::5 / ff02::6 
### Network Types 
- classifies links based on Layer 2 encap
- Broadcast (ethernet)
- Point-to-Point (loopback / HDLC)
- Non broadcast Multi-access
- Point to Multipoint
- Loopback

### NBMA
- a network which can potentially connect 3 or more routers at LAYER-2 but requires the pacckets be address indifividually

### OSPF Neigborship
 - Hello / dead interval 
 - Authentication
 - Area ID
 - Prefix-length
 - Stub area Flag

 ### Tables
 - Neighbor table
 ``` show ip ospf neighbor```
 - Link State Database
 ```sh ip ospf database```
 - Routing table
 ``` sh ip route ospf```
- interface
```sh ip ospf interface ```

 ## OSPF DR / BDR Election
 - Creates a LSA, elects a Router-ID
 _router DR is sticky_
 _router id is not changed on the fly_
 - to change router id, 
 ```clear ip ospf process```
  - Priority determines the DR
    - then highest IP becomes the DR
    - next highest becomes the BDR

| Router | Role | State|
|--------|------|------|
| R1    | DR    | Full |
| R2    | BDR   | Full |
| R3    | DROTHER | 2-way |

### Path Selection
``` auto-cost reference-bandwidth ?```
```ip ospf cost ?```

### Verification
- Router ID

```
show ip ospf
show ip protocols
```
- IP Routing Table
```O IA == Route resides in an area that your loacal router is NOT CONNECTED to```
- Command _router ospf 1_
```it is a locally significant process ID that does NOT NEED TO MATCH between router```

# OSPF LAB
```
#R3
conf t 
!
router ospf  1
 router-id 0.0.0.3
 exit
!
interface loopback 0 
 ip addres 77.77.3.1 255.255.255.0
 ip ospf network point-to-point
 exit
!
router ospf 1 
 network 20.1.3.0 0.0.0.255
 network 20.1.34.0 0.0.0.255 area 0 
 network 180.50.0.128 0.0.0.31 area 0
 network 180.50.0.160 0.0.0.31 area 0
 network 77.77.3.0 0.0.0.255 area 0

#R4
conf t
!
router ospf 1
 router-id 0.0.0.4
 exit
!
interface loopback 0 
 ip address 77.77.4.1 255.255.255.0
 ip ospf network point-to-point
exit
!
router ospf 1
 network 20.1.45.0 0.0.0.255 area 0
 network 20.1.34.0 0.0.0.255 area 0
network 77.77.4.0 0.0.0.255 area 0


#R5
conf t 
!
router ospf 1
 router-id 0.0.0.5
 exit
!
interface loopback0 
 ip address 77.77.5.1 255.255.255.0
 ip ospf network point-to-point
exit
!
int g0/0
 ip ospf 1 area 0
 exit
 !
 int 0/1
  ip ospf 1 area 51
  exit
!
int gi 0/2
 ip ospf 1 area 215
 exit
 !
 end

```



## CLEAN ATTEMPT

router ospf 1 
 router-id 0.0.0.3

interface g 0/0
 ip ospf 1 area 0