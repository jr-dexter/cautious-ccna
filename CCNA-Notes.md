Transport : segments
Network : packets
Data Link : frames
Physical : bits

## IOS Monitoring Commands
- Display IP Address and Interface status
	# show ip interface brief
- Display Physical-layer status of interfaces
	# show interface <type/number>
- Display switchport operating mode and trunk info
	# show interface <type/number> switchport
- Display sumary of VLAN trunks
	# show interface trunk

- Show routes
	# show ip route
	# show ipv6 route
- Display summary of all active routing protocols
	# show ip protocols
- Display routing protocol neighbor relationships
	# show ip <protocol> neighbor
- Show _directly_ connected Cisco Devices
	# show cdp neighbor

- Version
	# show version
- Display directory files
	# dir flash:
	# dir system/dram

## Configuration Management
- Save current running config
	# copy running-config startup-config
	_ or _ 
	# wr / write memory
- Setting router to factory defaults
	- delete startup-config
		# erase startup-config
		_or_
		# write erase
	- Reload
		# reload

## Secure Access
- Configure enable passwod
	# enable password <password>
	# enable secret <password>
- Configure console password
	# line console 0
	# password <password>

- Configure Telnet password
	# line vty 0 4 (up to 5 telnet settions)
	# password <password>
	# login
- Uniqure username/password
	# username <username> privilege 15 password
	# login local

## Configuration Register

| Bit No. | Hex Value 				| Meaning/function | 
|---------|-------------------------|------------------|
| 00 - 03 | 0x0000 - 0x000F  | Defines the source of the default Cisco IOS Software image required to run the router 00 - Stays at the system bootstrap prompt 01 - Boots the first system image in onboard Flash 02-0F - specifies a default netboot filename. Enables boot system commands that override the default netboot filename |
| 06 | 0x0040 | Causes system software to ignore NVRAM |
| 07 | 0x0080 | Enables the OEM bit |
| 08 | 0x0100 | Disables the Break function | 
| 09 | 0x0200 | Uses secondary bootstrap | 
| 10 | 0x0400 | Broadcasts IP with all zeros |
| 5, 11, 12 | 0x0800 - 0x1000 | Defines the console baud rate (default 9600) | 
| 13 | 0x2000 | Boots default Flash software if network boot fails | 
| 14 | 0x4000 | Causes IP Broadcast to leave out network numbers |
| 15 | 0x8000 | Enables diagnostic messages and ignores contents of NVRAM | 

### Defaults
 0x2142 - ignore nvram/startup-config (Password recovery scenarios)
 0x2102 - Default load IOS from flash
 - Verify configuration register at the bottom of show version
 - Displays current config and next reboot setting
 
## Password recovery
### Router Procedure
1. Get connected to the Serial port of device
2. Reboot device
3. Send break signal during boot cycle
4. After entering ROMMON, Change config register
	- confreg 0x2142
	- reset
5. Replace startupconfig 
	# copy nvram:startup-config running-config

### Switch Procedure
1. Get connected to the Serial port of device
2. Power cycle device
3. Hold MODE button on front of device
4. Terminal drops into switch/boot mode
	# switch:
5. Initialize Flash
	# flash_init
6. Move out startup config
	# rename flash:config.text flash:config.text.old
7. Boot device	
	# boot
8. Move old config back 
	# rename flash:config.text.old flash:config.text
9. Move config back to running config
	# copy flash:config.text running-config
	
# Troubleshooting Layer 1

| type | descriptions | Causes | 
|------|--------------|------------|
| runts			|  < 64bytes 			| Caused by collision, duplex mismatch ? | 
| giants 		| packets > 1518-bytes 	| Failed NIC, Mismatched encapsulation types ( 802.1q on one side and ISL on the other ) for full-MTU payloads |
| input error 	| any possible reason that prevents the router/switch from fully receiving ir oricessubg a frame | includes all types of packet errors |
| CRC Error 	| Bad FCS | Noise on lan/bad cables, collisions, Bad NIC | 
| Frame Error 	| Non-integer number of octects | bad NIC/ cables | 
| collisions	| tracked by transmitting interface | retransmissions due to collisions |

# Ipv6 Addresses
| Type 				| Example | Uses |
|------				|---------|------|
| Link-Local 		| fe80::/10 - 1111 1110 1000 0000 | Assigned automatically; Last 64 bits is the 48-bit MAC address with FFFE inserted in the middle | 
| Global Unicast 	| 2000::/3 - first 3 bits set to 001 | Global routing prefix is 48 bit or less |
| Unique local 		| FC00::/7 	| Alway begins the same way; equivelent to private IPv4 space |
| Multicast 		| FF00::/8 - As log as the first 8 bits take the form of -> 1111 1111 <- is multicast | Ipv6 nodes listen to several IPv6 Multicat groups by default | 


# CDP - Cisco Discovery Protocol
 - Propriety
 - Layer 2 protocol
 - Provides os, versions

## Default setting
	- Hello time = 60s 
	- holdtime value of 180 seconds
# LLDP - Link Layer Discovery Protocol
	- open standard
	- IEEE 802.1ab
	- Media endpoint Discovery (MED) is an LLDP enchangement for VOIP
	- Ethernet but not WAN interfaces 
	- CDP and LLDP can be operational on the same interface
### LLDP configuration
	# lldp run
	# lldp holdtime 150
	# lldp timer 15
	# interface ethernet 0/0
	# lldp transmit

# VLAN
- Usage range is 1-4094

| Range 	| Usage | 
|--------	|----------|
| 1-1001 	| Normal usage range | 
| 1002-1005 | Reserved for Token Ring | 
| 1006-4094 | Extended-Range |

_Router ports are default DOWN_
_Switches ports are default UP_


## Voice VLAN
	- voice VLAN implies two operating on a switchport	
		- data (access) VLAN
		- Voice VLAN
	- Both must be explicityly configured
	- IP Phones lear of the Voice VLAN 
		- CDP (Cisco Phones)
		- DHCP Option-15 (non-cisco phones)
### Configuration
 -	# switchport voice vlan <vlan-id | dot1p | untagged | none>

## VLAN Trunk Port
	- Can have two or more VLAN configured
	- Can carry Multiple VLAN information
	- By Default, all VLAN traffic is allowed from a trunk port

### 802.1Q
	- Open Standard
	- All traffic exept native VLAN is inserted with a 802.1q tag
	- support concept of native VLAN
	- Must match on both ends of the TRUNK

#### Configuration
	```
	Switch(config)# interface <interface>
	Switch(config-if)# switchport trunk encapsulation dot1q
	Switch(config-if)# switchport mode trunk 
	Switch(config-if)# end
	```
#### Verification
	```
	Switch# sh vlan <brief>
	Switch# sh interface trunk
	Switch# show interface status
	Switch# show interface <interface> switchport
	```
## VLAN LAB
Task: VLANs & Trunking

1. Configure VLAN 100 and VLAN 200 on all switches.
2. Configure any names for the VLANs.
 Associate VLANs on the ports as follows:
	Sw1's Gi0/1 in VLAN 100
	Sw2's Gi0/1 in VLAN 100
	Sw2's Gi0/0 in VLAN 200
	Sw3's Gi0/3 in VLAN 200
3. Configure trunk ports as specified below:
	Gi1/0 connecting Sw1 to Sw2
	Gi3/0 connecting Sw1 to Sw3
	Configure these trunks to use 802.1q tagging

4. Configure IP addresses on the hosts as follows:
	VLAN 100: 100.1.1.0/24
	VLAN 200: 200.1.1.0/24

## VLAN LAB Solution
Add/Name vlan
```
    Sw1,Sw2 & Sw3:
    vlan 100
    name IT
    exit
    !
    vlan 200
    name Sales
    exit
```
Config Access Ports and assign to vlan
```
	Sw1:
    interface Gi0/1
     switchport mode access
     switchport access vlan 100

    Sw2:
    interface Gi0/1
     switchport mode access
     switchport access vlan 100
    !
    inter Gi0/0
     switchport mode access
     switchport access vlan 200

    Sw3:
    interface Gi0/3
     switchport mode access
     switchport access vlan 200
```
Config Trunk between switches
```
	Sw1:
    interface Gi1/0
     switchport trunk encapsulation dot1q
     switchport mode trunk
    !
    interface Gi3/0
     switchport trunk encapsulation dot1q
     switchport mode trunk

    Sw2:
    interface Gi1/0
     switchport trunk encapsulation dot1q
     switchport mode trunk

    Sw3:
    interface Gi3/0
     switchport trunk encapsulation dot1q
     switchport mode trunk
```

Config IP address on the host routers that correspond to the particular VLAN.
```
   R1:
    interface Gi0/0
     ip address 100.1.1.1 255.255.255.0
     no shutdown

    R2:
    interface Gi0/1
     ip address 100.1.1.2 255.255.255.0
     no shutdown

    R3:
    interface Gi0/1
     ip address 200.1.1.3 255.255.255.0
     no shutdown

    R4:
    interface Gi0/1
     ip address 200.1.1.4 255.255.255.0
     no shutdown
```
### Verification
R1/R2 assigned to VLAN 100
R3/R4 assigned to VLAN 200

```
# sh vlan
# sh interfaces trunk
```

R1 should reach R2 via ping
```
R1# ping 100.1.1.2
```

R3 should reach R4 via ping
```
R3# ping 200.1.1.4
```

# Spanning Tree
 1. Negative effects of a bridging loop
	- Excessive network bandwitch utilization
	- Host devices which are cut off from the network
	- High CPU utilization
 2. Original IEEE specification for rapid spanning-tree
	- 802.1w
## Root bridge election
- switch with lowest bridge ID in the network becomes the root bridge
- bridge ID contains (16 bit number)
	- bridge priority 
		- default 32768
		- increments in multiples of 4096
		- lowest number is highest priority
	- system ID extension (0 - 4095)
	- MAC Address
	
| RSTP Port Role | RSTP Port State | 
|----------------|-----------------|
| Designated port (Designated to delivers BPDU)	| Forwarding |
| Root Port (receives) 							| forwarding | 
| Edge port (host) 								| forwarding |  
| Alternate Port  								| Discarding / Blocking |
| Backup Port 									| Discarding / Blocking|

### STP Port States Interface Modes
| 802.1w (RSTP)	| 802.1d (STP)	| RX BPDUs | TX BPDUs | Learns MAC Addrs | FWD Data Packets | 
|---------------|-----------|----------|----------|------------------|------------------|
| Discarding | Disabled 	| No 		| NO 		| NO 			| No				|  
| Discarding | Blocking 	| Yes 		| NO 		| NO 			| No				|  
| Discarding | Listening	| Yes 		| Yes 		| NO 			| No				|  
| Learning	 | Learning 	| Yes 		| Yes 		| Yes 			| No				|  
| Forwarding | Forwarding	| Yes 		| Yes 		| Yes 			| Yes				|  

_ What are the components of the RSTP Bridge-ID ?_
	- Bridge-Priority + MAC Address
	
### Enable RSTP
- automatically backwards compatible with legacy STP
```
(config) # spanning-tree mode rapid-pvst
```
- Enable PortFast for RSTP Edge Port recognition
```
(config) # spanning-tree mode portfast [trunk]
_OR_
(config) # spanning-tree mode portfast default
```

### Deterministically set RSTP root bridge per VLAN
```
(config) # spanning-tree vlan <vlan-id> root primary
_OR_
(config) # spanning-tree vlan <vlan-id> priority <value>
```
- Priority values attempts to set the lowest rBridge ID just enough lower than the existing root bridge. 
	- if there is a matching case of priority , ie both set to 4096, the lowest mac address becomes the root bridge
	
```
 show spanning-tree summary [vlan <vlan-id>]
```

### Check spanning tree
```
#sh spanning-tree summary
Switch is in rapid-pvst mode
Root bridge for: none
Extended system ID                      is enabled
Portfast Default                        is disabled
PortFast BPDU Guard Default            is disabled
Portfast BPDU Filter Default           is disabled
Loopguard Default                      is disabled
EtherChannel misconfig guard            is enabled
UplinkFast                              is disabled
BackboneFast                            is disabled
Configured Pathcost method used is long

Name                   Blocking Listening Learning Forwarding STP Active
---------------------- -------- --------- -------- ---------- ----------
VLAN0003                     0         0        0          1          1
VLAN0060                     0         0        0         27         27
VLAN2060                     0         0        0         27         27
---------------------- -------- --------- -------- ---------- ----------
3 vlans                      0         0        0         55         55

```

```
#sh spanning-tree vlan 3

VLAN0003
  Spanning tree enabled protocol rstp
  Root ID    Priority    4099
             Address     7872.5d28.df00
             Cost        1551
             Port        2285 (Port-channel5)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32771  (priority 32768 sys-id-ext 3)
             Address     d009.c8f0.7100
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Po5                 Root FWD 1000      128.2285 P2p
```

#### STP Timers
| Type 			| Time 		| Description | 
|---------------|-----------|-------------|
| Hello  		| 2 seconds | How often the root bridge originates BPDUs |
| Forward Delay | 15 sec (learning) + 15sec (fowarding) | length of the listening and learning port states | 
| Max Age 		| 20 seconds | How long a switch will retain the superior BPDUs contents before discarding | 

### Root Port Election
- RP is upstream facing towards Root Bridge
- Elected based on lowest Root Path Cost
	- cumulative cost of all links to get to the root
- Cost based on inverse bandwidth
	- higher bandwidth , lower cost
	- not linear
- if tie in cost...
	- lowest upstream bridge ID
	_OR_
	- lowest upstream (SENDING - where BPDU came from ) port ID

#### RSTP Cost Values
- every switch that transmits/forwards a BPDU incluses its own localt cost to reach STP Root
- *_802.1d_* specififded some default port costs but does not specify any formula for ports outside of theses values
|Parameter | link speed | recommended value | recommended range | range | 
|--------	|---------------|---------------|-------------------|--------|
| path cost | 10Mb/s 		| 100			 | 50-600 			| 1-65535 |
| path cost | 100Mb/s 		| 19			 | 10-60 			| 1-65535 | 
| path cost | 1Gb/s 		| 4			 	 | 3-10 			| 1-65535 | 
| path cost | 10Gb/s 		| 2				 | 1-5	 			| 1-65535 | 

_ Under which conditions would a switchport be elected as a Designated Port? _
	- If the port in question leads to the a host device.
_Which of the answers below accurately identifies a benefit of configuring a Cisco switch with Rapid-PVST as compared to PVST+? _
	- Rapid-PVST allows any Bridge to send a Topology Change BPDU whereas PVST+ restricts this action solely to the Root Bridge.
_An RSTP Topology Change BPDU is received by Switch-A on Port-1 and forwarded on Port-2. Based on this event_
	- MAC addresses that were learned on Port-2 will be instantly deleted from the MAC table.

# EtherChannel

| Type | Modes | State to activate | 
|-------|---------|----------------|	
| PAgP | On / Desirable / Auto | On/On - On/Desirable  |
| LACP | (On) / Active / Passive | Active/Active - Active/Passive |

When bundling several links on a single switch into an Etherchannel, you need to use the "channel-group mode" command followed by a keyword. 
The keyword determines the usage of PAgP, LACP, or creation of a static Etherchannel. 
Which of the following statements are true? 
	- The mode of "active" utilizes LACP.
	- The mode of "auto" utilizes PAgP.

When placing the connections between two switches into an Etherchannel, which LACP modes can be used to dynamically create the Etherchannel? 
	- Switch-A: Active --- Switch-B: Active
	- Switch-A: Active --- Switch-B: Passive

When placing the connections between two switches into an Etherchannel, which PAgP modes can be used to dynamically create the Etherchannel? 
	- Switch-A: Auto --- Switch-B: Desirable
	- Switch-A: Desirable --- Switch-B: Desirable

## Lab: 
Configure EtherChannel on Sw1, Sw2, and Sw3 as follows:
Configure PAgP on all the links between Sw1 and Sw2.
Configure LACP on all the links between Sw1 and Sw3.
In both configurations, only Sw1 should be able to initiate the channel.

1. Verify all interfaces share a similar configuration
2. Modify interface ranges to create etherchannel
3. Sw1 -> Sw2: PAgP ( auto /desirable )
```
    Sw1:
    interface range Gig1/0-2
     channel-group 1 mode desirable

	Sw2:
    interface range Gig1/0-2
     channel-group 1 mode auto
```

4. Sw1 -> Sw3: LACP ( active / passive )
```
	Sw1:
	interface range Gig3/0-2
     channel-group 2 mode active
	
	Sw3:
    interface range Gig3/0-2
     channel-group 2 mode passive
```
5. Verify
```
Sw1#show etherchannel summary
    Flags:  D - down        P - bundled in port-channel
            I - stand-alone s - suspended
            H - Hot-standby (LACP only)
            R - Layer3      S - Layer2
            U - in use      N - not in use, no aggregation
            f - failed to allocate aggregator

            M - not in use, minimum links not met
            m - not in use, port not aggregated due to minimum links not met
            u - unsuitable for bundling
            w - waiting to be aggregated
            d - default port

            A - formed by Auto LAG


    Number of channel-groups in use: 2
    Number of aggregators:           2

    Group  Port-channel  Protocol    Ports
    ------+-------------+-----------+-----------------------------------------------
    1      Po1(SU)         PAgP      Gi1/0(P)    Gi1/1(P)    Gi1/2(P)    
    2      Po2(SU)         LACP      Gi3/0(P)    Gi3/1(P)    Gi3/2(P)  
```

# Wireless

|Acronym | Definition | Description |
|------|-----|----|
| BSS | Basic Service Set | Single access point and its coverage area |
| BSSID | ID (Mac) | ID of Access point |
| SSID | Service Set Identifier | Configurable name of the WLAN | 
| DS | Distribution System | Wired network | 
| ESS | Extended Service Set | collection of APs connected to the same DS and offering the same WLAN (SSID) |
| ESSID | Extented Service Set | Same as ESS | 


## Radio Frequency
 - a measurement of how often something changes over a given time interval
 - Hertz: measurement of how frequently something changes over 1-second interval 
	- 1 Hz = 1 changes /cycle per seconds
 - Electro Magnetic Radition: EMR
	- radition that has both electrical and magnetic properties 
	- detectable waveform (ossilates)

## Radio Propagation Problems

| Type | Result | Example |
|-------|-----------|------|
| Absorption | Soaks up RF energy | Concrete (6-15dB), Window (3dB), drywall (3-5dB) |
| Reflection | redirects RF energy into other directions | Metal / Body of water |
| Refraction | bending of RF energy as it passes through objects of different density | Air to window |
| Diffraction | bending and spreading of RF; creates shadow |  hills or buildings |
| Scattering | similar to Refraction on larger scale | smog, dust, humidity | 

### Free space path Loss - FSPL
 - amount of energy lost as a signal travels through the air
 - Loss if relative to frequency and distance
 - high frequency > FSPL
### Multipath
 - RF Signal wave is transmitted away from transmitter, will expand and encounter multiple objects on its way to the receiver
 Effects:
	- data correuption
	- signal nulling
	- increase/decrease signal amplitude

## Wi-Fi bandwidth
 - Each channel is ~20Mhz wide

- What is the main value of using a standard of Wi-Fi that leverages increased channel bandwidth as compared to an older standard?
	- Greater channel BW = Higher Throughput
- If Wi-Fi channel "Z" is said to have more bandwidth than Wi-Fi channel "Y", then this means
	- There are more frequencies available on Channel-Z than on Channel-Y
- The earliest forms of Wi-Fi (802.11a/b/g) used channels that were limited to ____ in bandwidth
	- 20MHz

- What was the first IEEE Wi-FI standard to provide for the possibility of utilizing 40MHz channels? 
	- 802.11n
- Which U-NII band(s) is frequently avoided in the United States due to the requirements of DFS? 
	- U-NII-2A
	- U-NII-2C
- In the United States, what is the maximum quantity of available 40 MHz non-overlapping channels in the 2.4GHz RF space? 
	- 1

## Split-MAC Architecture
 - implementing an on premise controller
 - utilizing lightwghit access point (LAP) to split functions into two sections

 - Real time function handled locally by LAP

 ### LAP
	- RF tx/rx  ( mgmt, control, data)
	- MAC Mgmt (DCF)
	- encryption/decryption of user data
 ### Mgmt functions offloaded to WLAN controller
	- Association and roaming mgmt
	- Client authentication
	- securit policy Management
	- QoS

# WiFi Access Point Modes

| Mode | Description | 
|------|----------------|
| Local 	| default mode; creates CAPWAP tunnel to WLAN controller; All control and data flows across CAPWAP tunnel; Should CAPWAL tunnel fail, AP disconnects all WLAN clients | 
| Bridged 	| allows autonomous AP act as a WLAN client and associate to a LAP; wired clients can connect to ETH port on back of AP |
| Flex | Hybrid-remote-AP; AP utilizeds services of controller, like controller; this mode must be supported/config'd on WLAN and AP; primarily designed for AP placed at remote site; allows AP to locally bridge WiFi traffic onto wired LAN for clients assoicated to specific SSID/VLAN |

- Bridge: wireless Printer <-ETH-> autonomous AP <- wifi client -> LAP > controller > Switch 

When an access point in Local Mode loses its CAPWAP tunnels, which of the following statements is true?
	-All existing Wi-Fi clients are disassociated from the LAP and no new client associations are allowed

You have placed several outdoor access points throughout a local park to provide free Wi-Fi connectivity. 
Most of these APs have no connection to the Distribution System but rather connect back to the WLAN Controller via a wireless connection to another access point. These access points are called _____. 
	- MAPs

You want to utilize one of your access points to perform some onboarding, network and application tests on a particular SSID. To do this you should configure your AP to be in _______
	- Sensor mode

You want to utilize one of your access points to scan all available channels in the 2.4GHz range to detect Wi-Fi attacks such as MAC spoofing and rogue AP detection. To do this, you should place your AP into _______.
	- Monitor Mode


# WPA3
Which of the following answers identifies the advanced form of authentication that was introduced with WPA3?
	- SAE
Which mandatory feature of WPA3 was designed to prevent a Wi-Fi user from being disassociated from the WLAN due to a spoofing attack? 
	- PMF
Which of the following is an advantage of using WPA3 Personal instead of WPA2 Personal?
	- WPA3 Personal requires the use of Simultaneous Authentication of Equals
Which of the following is a feature of WPA3 that allows for an encrypted Wi-Fi session that does not require usage of a PSK or 802.1x?
	- Wi-Fi Enhanced Open

# IP routing
 ## General rules of routing
 - routers will only accept routes that match its own, active protocols
 - routers will only use routes with a reachable "next hop"
 - next-hop must be paired with a usable L2 address
 - routers will only use the best routes

 As a network administrator, you want to enable IOS debugging commands but are concerned those commands could overwhelm the console. 
 Instead you wish to allow all SYSLOG messages to be displayed on the console but to have debugging output prevented from being displayed on the console and stored in a memory buffer instead.
 Which of the configurations below will accomplish this? 
 - 	Device#config t 
 	Device(config)#no logging console 
	Device(config)#logging console 6 
	Device(config)#logging buffer debug

You have configured a static route on your router, but it is not showing up in the IP Routing Table. Which of the following reasons could account for this?
- 	The egress/outbound interface that is paired with this route is down
	The static route is pointing to a next-hop that the router can't reach

## How are routes stored
 - Process-based switching
	From ingress to egress traffic directions

### Quiz
- An Ethernet interface, running in Half-Duplex mode will utilize the CSMA/CD protocol. If this interface continually detects collisions when retransmitting the same frame repeatedly, it will attempt to resend the same frame up to _____ before giving up and discarding that frame
	- 16 times
- Which of the following statements are true about the differences between 802.3 and EthernetV2 frames? 
	- EthernetV2 frames utilize an Ethertype feild
	- 802.3 frames utilize an SSAP feild

# Ethernet Frame Structure
  - MAC Address = 48 bit 

## Runt
 - Frames less than 64-bytes
 - caused by collions, malfunctions NICs
## Giants
 - Frame larger than 1518 Bytes (without 802.1q tag)

# Cables
 - Straight - through

 - Crossover 
 	- Devices of same type / same OSI Layer
	- TX 1 - RX 3 
	- TX 2 - RX 6

### Quiz
 -  When using the nomenclature of 100BaseT to describe Ethernet cabling, what does the "Base" represent?
 	- That, when transmitting, this technology has access to the full spectrum of electrical energy on the cable

 
## Router on a stick
- For a router to route frames between VLANs the following must be
	- *TRUE* if:
	- Router must have a unique IP addr appropriate for each VLAN subnet
	- Router may be required to support 802.1q VLAN tagging
	- Host must become aware of the router and its IP Address
### Router
- Create sub interfaces on the router
- Apply vlan to the sub interface
### Switch
- make link to router a trunk port

```
R1# int fast 0/0
	no shutdown
	
	int fast 0/0.2
	encapsulation dot1q 2
	ip address 2.2.2.3 255.255.255.0
	
	int fast 0/0.3
	encapsulation dot1q 2 
	ip address 3.3.3.3 255.255.255.0

S1# int fast 0/1
	switchport mode trunk

```


## LAB: Router on a stick 
Task: Inter-VLAN Routing

Your objective in this task is to convert routers R2, R3, and R4 into hosts by disabling their routing capability. After that, you will configure router R1 as a "router-on-a-stick" so that it can route traffic between these hosts.

Configure Sw1's Gig/1 interface as a trunk, using encapsulation 802.1Q.

Configure inter-VLAN routing using R1 as a "router-on-a-stick".

R1 must be configured to route between VLANs-100 and 200.
R1 must be provided two IPv4 addresses for this task:
100.1.1.1 /24
200.1.1.1 /24
Routers R2, R3 and R4 should act like hosts for this task. This will involve disabling IP Routing on these devices and adding "ip default-gateway" statements to each of them pointing to R1
Upon completing this task, R3 should reach R2 and R4.


Solutions:

Because we have used different VLANs to connect the routers and the task is asking us to make them reachable, inter-VLAN routing should be in place to make them reachable. As we know, there should be at least one L3 device; that is, a router or L3-capable switch. So we are using R1 to perform inter-VLAN routing for VLAN 100 and VLAN 200. Make sure that the switchport connected to R1 is configured as static trunk because a router does not support DTP. Additionally, we must configure sub-interfaces on R1 on the basis of which VLANs we are trying to route to each other.

First, configure Sw1's Gig0/1 port as a trunk.

```

    Sw1>en
    Sw1#config t
    Enter configuration commands, one per line.  End with CNTL/Z.
    Sw1(config)#interface gig0/1
    Sw1(config-if)#switchport trunk encapsulation dot1q
    Sw1(config-if)#switchport mode trunk
    Sw1(config-if)#end
    Sw1#
    *Jul 27 17:07:50.275: %SYS-5-CONFIG_I: Configured from     console by console
    Sw1#
    Sw1#show interface gig0/1 switchport
    Name: Gi0/1
    Switchport: Enabled
    Administrative Mode: trunk
    Operational Mode: trunk
    Administrative Trunking Encapsulation: dot1q
    Operational Trunking Encapsulation: dot1q
    Negotiation of Trunking: On
```
- Now configure R1 with sub-interfaces and IP addressing for Vlann 100 / 200
```

R1:
interface Gig0/0
 no ip address
 no shutdown
!
interface Gig0/0.100
 encapsulation dot1Q 100
 ip address 100.1.1.1 255.255.255.0
!
interface Gig0/0.200
 encapsulation dot1Q 200
 ip address 200.1.1.1 255.255.255.0
```

Set the default-gateway on R2, R3, and R4. Because we are using routers as the hosts, we must disable "ip routing" first and set the default gateway accordingly.
```

    R2:
    no ip routing
    !
    ip default-gateway 100.1.1.1

    R3:
    no ip routing
    !
    ip default-gateway 200.1.1.1

    R4:
    no ip routing
    !
    ip default-gateway 200.1.1.1
```

- Verification
```

    R2#sh ip route
    Default gateway is 100.1.1.1

    Host               Gateway           Last Use    Total     Uses  Interface
    ICMP redirect cache is empty

    R2#ping 200.1.1.4
    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 200.1.1.4, timeout is 2     seconds:
    .!!!!
    Success rate is 80 percent (4/5), round-trip min/avg/max =     36/36/36 ms

    R2#ping 200.1.1.3
    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 200.1.1.3, timeout is 2     seconds:
    .!!!!
    Success rate is 80 percent (4/5), round-trip min/avg/max =     28/29/32 ms

    #########################

    R3#show ip route
    Default gateway is 200.1.1.1

    Host               Gateway           Last Use    Total     Uses  Interface
    ICMP redirect cache is empty

    R3#ping 100.1.1.2
    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 100.1.1.2, timeout is 2     seconds:
    !!!!!
    Success rate is 100 percent (5/5), round-trip min/avg/max =     12/15/16 ms

    #########################

    R4#show ip route
    Default gateway is 200.1.1.1

    Host               Gateway           Last Use    Total     Uses  Interface
    ICMP redirect cache is empty

    R4#ping 100.1.1.2   
    Type escape sequence to abort.
    Sending 5, 100-byte ICMP Echos to 100.1.1.2, timeout is 2     seconds:
    !!!!!
    Success rate is 100 percent (5/5), round-trip min/avg/max =     28/29/32 ms
```



## OSPF
 - Mutlicast Hello messages on 224.0.0.5 
 - LSU packets from the DR use 224.0.0.6
### OSPF Neighborship
- Hello packets- heloow and dead interval 
- authentication 
- area-id
- prefix length
- Stub area flag 

### Neighbor States
R1				|||		R2
Init 			|>|
				|<| 	2-Way State
2-Way State 	|>|	
				|<| 	DR Election
ExStart State   |><| 	ExStart State (LSA Headers)
				|<| 	Exchange State
				|<| 	Loading State (LSR / LSU / LSAck / Full LSA) 
 Full State 	|><| 	Full State


Which OSPF packet type is only exchanged between two routers during the initial stages of forming an adjacency?
- Database Descriptor

Which type of OSPF packet is used to transport LSAs?
- LS Update

While building an OSPF adjacency, two routers go into the EXSTART state. Up to this point, what OSPF packet types have been exchanged? 
- Hellow Packets
- Database Description Packets


### OSPF Network types
- Dynamically Determined
	- Broadcast
		- Ethernet, token ring, FDDI
	- Point-to-Point
		- Loopback, PPP, HDLC
	- Non-broadcast Multi-access
		- Frame-relay
- Point-to-Multipoint
	- Serial

In OSPF terminology, a router that has learned of a route from some non-OSPF protocol, and then converts it into OSPF and advertises it within an LSA is called 
- ASBR

How does OSPF categorize a network which can potentially connect to three-or-more routers at Layer-2, but requires packets to be addressed to each router individually?
- NBMA - Non broadcast multi access

What is the name of the OSPF Data Structure which stores all LSAs
- Link State Database

How would OSPF categorize a link that is using HDLC encapsulation?
 - Point-to-point

An OSPF router has interfaces in Area-1, Area-2 and Area-3. Because of this, we call this router a(n) ___?
- Nothing special

How does OSPF categorize a network in which a single packet can be placed on that network and potentially be received by three-or-more routers?
- broadcast



### Configuring OSPF
 
```
# init 
R1(config)# router ospf <process-id>
R1(config-router)# network <network-id> <wildcard-mask> area <area id>
R!(config-router)# router-id <router-id>

# priority
R1(config-interface)# ip ospf priority <priority>
# Priority 0 = never become DR
# priority 1 = default/lowest 

# Verification
show ip ospf neighbor
show ip ospf interface
show ip ospf database
show ip router ospf 
```

# EiGRP
 - Open standard
 - Hybrid iGP
 - Characteristics of both link state and distance vector
 - Metrics based from link bandwidth & Delay
 - Manual and auto summarization
 - Supports unequal load-balancing

## EiGRP Packets
*Multicast Address is 224.0.0.10*
 - Neighbor hello 
 - Routing Updates
	- Update (unicast initially, then multicast)
	- Acknowledgement (unicast)
	- Query
	- Reply

### Category of Routes
 - Administrative Distance is Locally defined
 0. directly connected
 1. Static
 X. the rest

 ### EiGRP Internal
 - Admin Distance = 90
 ### EiGRP External
 - route that was previously learned via some non-eigrp methoth and inhjected int oEiGRP with "redistrute command"
 - Admin Distance = 170

 ### DUAL technology
 - Successor and Feasible Succesor Route
 1. Evaluate all non-sucsessor routes
 ```
  if eval-route-RD < FD
	then neighbor = Feasible success path
  if eval-rout-rd >= FD
	then neighbor can not be a feasible success for the path (possible loop)
 ```

 ### Commands
 ```
  # show ip route eigrp
  # show ip eigrp topology <all-links>
  
 ```

# Port Security
```
# show port-security
# show port-security interface gi0/1
# show port-security address
```
- _The output of the command, "show port-security" displays (among other things) an interface that has zero (0) Security Violations and a Security Action of "Protect". Based on this observation which of the following answers are absolutely true?_
 - the security violations counter will never increment beyond zero this port

 _What information will the command, "show port-security interface" provide that you would NOT see from the output of "show port-security"?_
 - Current port status
 - most recent MAC learned on this interface
 - quanitity of sticket MAC that have been learned

 ### Configuration 
 ```
 interface gi 0/1
 switchport port-security
 switchport port-security ?
  aging        Port-security aging commands
  mac-address  Secure mac address
  maximum      Max secure addresses
  violation    Security violation mode

#switchport port-security violation ?
  protect   Security violation protect mode
  report    Security violation report only mode
  restrict  Security violation restrict mode
  shutdown  Security violation shutdown mode


#switchport port-security mac-address ?
  H.H.H   48 bit mac address
  sticky  Configure dynamic secure addresses as sticky

 ```

 # DHCP Snooping
 - Discover and Request are broadcast
 - Untrusted port traffic is not broadcast to another untrusted port
 - DHCP Snooping Binding Database contains:
	- Client MAC
	- Client Port
	- Client IP
	- Lease Time
	
 ### Configuration
 ```
 ip dhcp snooping
 ip dhcp snooping vlan 2
 ip dhcp snooping limit rate <1-2048> (limit INCOMING PACKETS)
 ip dhcp snooping trust
 [no]
 ip dhcp snooping information option
  ```

# Identifing Classes & Types of IPv4 Address
 ### Classes of Addresses:
 | Class | Low | High | Description |
 |------|--------------|-----------|----|
 | A    | 0.0.0.0 	| 127.255.255.255 | First bit is 0 |
 | B 	| 128.0.0.0 	| 191.255.255.255 | Starts with bits 10 |
 | C 	| 192.0.0.0 	| 223.255.255.255 |	Starts with bits 110 (Multicasts) |
 | D	| 224.0.0.0		| 239.255.255.255 | Starts with bits 1110 |
 | E	| 240.0.0.0 	| 255.255.255.255 | Starts with bits 11110 |



# IP Routing Basics
## Static Routes
- Static routers on Point-To-Point Links can use the destination interface. There is expect to me only two hosts on a P2P links, so if the packet is not for me, it must be for you.
- Static route for Multiple Access Links, ie THE REST. There is the possiblility that many host can recieve the packet. Result is to specify the testing IP Address instead of the interface. 


```RTR(config)# ip route <network destination> <subnet mask of destination> {destination IP | destination interface}```

### Verify
```bash
sh ip route
sh ip route static
```

## Floating Static Route
- Floating refers to the manual configuration of the routes Admin distance. 
- Route Table numbers in the brackets. first number is the Administrative distance. lowest is best.

## Routable addresses
 - IPv4 / IPv6 are routable protocols
 - Reaching address with the same network are within the same broadcast domain
 - Reaching addresses outside of the network where the IP is assigned, route to the default gateway
 - Reaching addresses outside of the default gateway router through the default route

## What happens to a routed packet
 - Compares its local address and determined the destination is not on-link -it routes to the default gateway
 - Routing: process of forwarding packets between networks
	- routing between networks
	- _route_ table is different than a *_routing_* table
- Requirements 
	- routable packet (ipv4/v6)
	- network address
	- subnet mask
	- next hop
	- egress interface
 - Orginal Ethernet frame is re-written to once it reaches a router
	- The Layer 2 Src/Dest MAC will reflect each hop in the flow
	- Layer 2 re-written at the outbound interface

### Question
In order for an address, assigned to a device, to be considered routable
 - the address must have embedded within it a network idenfitifier
 - address must be paired with some kind of mask

## Where are the routes stored
 ### Types of Routes
  - Connected
  - Static 
	- Manually configured 
  - Dynamic 
	- learning from another router
- Recursive route
	- route that finds a match but the next hop address where to find the interface a route high in the routing table need to be used. Example, a route that leads to another route

### Rules of Routing
	- only match their active protocols
		- if ipv6 route is received but ipv6 is not running, this packet is dropped
	- router will only use routes with reachable "next hops"
		- either direct or recursive
	- next hop of paired with usable Layer 2 address
		- if an ARP reply is unfulfilled, this layer 2 address is unknown
		- _encasulation failed_
 			- results from an unusable layer 2 address
	- routers will only use the "best" routes
		- static routes linger in the routing table after its interface is admin-down
	- Routes must be "beleivable" - is it still valid.

## Redirect Debug logging to buffer
 ```
	no logging console debug
 	logging buffer debug
	# Agressive debug to turn on
	debug ip packet
	# Clear the buffer log
	clear log
	# Remove debug
	undebug all
	# View collected logs
	show log
```

### Switching within Routers
	- Process-based switches
		- original CPU-based switch architecture. each packet interupts the CPU, triggers a ARP, prcoess reply and passes to routing table
	- Fast Switching
		- cache created to contain next hops/ interface information. reduces CPU interupts, but unknown info is still sent to CPU
	- Cisco Express Forwarding
		- copy of ip routing
		- creates a table of next hop/interface matching as well as mac address/interface table
		- CEF for forwarding
		- FIB (forward information base) - interface to map table
``` sh ip cef ```

#### CEF Components
	- Populated with Layer 2 Adjacency Information
	- Populated by Layer 2 Tables
		- ARP Table
		- Frame-Relay
		- P2P Header Formats
``` bash 
show adjacency <intf type/number > [summary|detail]
show adjacency detail
```

### Adjacency Types
- Glean
	- CEF has learned of a subnet and a host in that network. if other hosts are made available, this glean adjacency is use to ask the CEF to ARP for the address
- Null
	- packets are dropped. can be used as access filtering
- Drop
	- dropped packets
- Discard
	- dropped based off policy
- Punt
	- _Which type of CEF adjacency would, if matched by a packet, force that packet to be processed by the device's CPU?_
	- Inconnect Layer 2 information, passes work off to higher level switch to determine

## How are Routes Selected
	- Admin distance is used to compare routes learned via different protocols
	- Metric is used to compare routes learned via same protocol
#### Example
```
R1 learns the following routes via dynamic routing protocols:

192.168.0.0/25 via RIP, metric 4
192.168.0.0/30 via EIGRP, metric 1234
192.168.0.0/28 via OSPF, metric 10
192.168.0.0/16 via eBGP, metric 1
An admin also configured a static route to 192.0.0.0/8.

Which route(s) will R1 add to its routing table?
```
 - group/compare by protocol

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

## Contrasting ROuting Protocols:: IGP/EGP
 - Interior Gateway Protocol
	- Within an Autonomous System
	- Fast Convergence
 - Exterior Gateway Protocol
	- BGP
	- Secure 
	- Large quantity of routes

### Protocol Charactistics
 - Operational 
	- Distance Vector (IGP)
	- Link-State (IGP)
	- Advanced Distance Vector (IGP)
	- Path Vector (EGP)
 - 	Dat Points
	- Neighbor Requirements
	- Route Maintenance
	- Visibility into Topology
	- Data Structures ( Tables, DBs)
 - Updates
	- Frequency
		- Periodic / Triggered
	-  Size
		- Incremental / Full	

### Distance Vector
|  Protocol  |    Metric   | Neighbor Reqs      | Route Maintenance           | Visibility to topology   | route mgmt data structure   |
|----|----|----|----|----|---|
|  RIP / IGRP |  Distance  | None               | Resend routes periodically  | Directly connected routers  | db of learned routes |
| OSPF / IS-IS     | Link State  | Required     |  periodic keep-alives / regenerate LSAs | complete topology | db of link state, neighbor table , SPF (shortest path first) Tree |
| EIGRP       | Advanced Distance Vector | Required |periodic keep-alives | directly connected routers | topology of learned routes / neighbor table | 
| BGP      | Path Vector | Required/Exterior | Period keep-alives | no knowledge of topology- only next hop | neightbor table / bgp table | 

