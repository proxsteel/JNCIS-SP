#vLab SSH passwd: jcluster/Juniper!1

- Module 05: ISIS Concepts
- Module 06: ISIS Configuring and Monitoring
----
# ISIS by ISO: [ISIS Useful Info](https://www.rhyshaden.com/isis.htm)
### *//developed by International Organization of Standardization(ISO) for OSI model
#### [Interesting Facts:](https://www.youtube.com/watch?v=EiXvyFA1Rrs)
- [ISIS vs OSPF](https://momcanfixanything.com/ospf-vs-isis/)
- An IGP desinged to route CLNS -- vs OSPF that was designed for TCP/IP stack. Nowadays, Integrated IS-IS is used primarily to route IP rather than CLNS. 
>> - IP family was added later to ISIS -- `Junos OS is runing Integrated ISIS that has spport for IP protocol, both v4&v6`
- Link state SPF(Dijkstra) Algorithm -- likewise OSPF
- ISIS LSP can be fragmented(only on the originating node in multiple PDUs using seq ID but must stay intact across the network). MTU in OSPF and ISIS must be the same `MTU is a prerequesit for neighborship forming in OSPF and ISIS` 
- Routers -- Represented by Netowork Entity Title - `"NET" a 10 bytes hex value (range 8 to 20 bytes) *//NET 49.0001.1921.6825.5010.00 {49.-AFI(49. Private use); 0001.-AID(can be 16/32/48 bits leght); 1921.6825.5010.-SysID(must be a 6bytes leght); 00-NSEL`
- More tunable than OSPF
- Rarely run full SPF recalculations and prefers using partial route calculation(PRC) 
- More Efficient than OSPF -- Better resouce utilization of device
- More Felxible than OSPF -- Less restrictions on deployment
- Much More Difficult to Understand than OSPF   
- Default metric on all interface is 10, except lo0 which is 0 `[range min 0 - max 63 || a 2^24 value using wide metric, range  0 - 16 mil.]`, max path cost on a path is 1023.(`and wide 2^32???`). Metric is not based on Bandwith but rather on the value(user defined/Default).[METRICs](https://momcanfixanything.com/isis-narrow-and-wide-metrics/)
- Default Preference/AD: `Internal L1-15/L2-18 && External routes L1=160/L2=165`
- DESTINATION ADDRESS/PROTOCOL: * Encapsulated within Layer 2 frames using:
  - DSAP/SSAP= 0xFE
  - Multicast DA = 0180.c200.0014 (LEVEL 1)
  - Multicast DA = 0180.c200.0015 (LEVEL 2) 
- LSP Lifetime = 1200 sec (20 min) VS OSPF 3600 sec(60 min)
- Default LSP Refresh Interval= 883 sec (lifetime minus 317) VS OSPF 3000 sec(50 min)
```
user@vMX-1> show isis overview
...........Omited Output.........
  Level 1
    Internal route preference: 15
    External route preference: 160
    Prefix export count: 31
    Wide metrics are enabled
  Level 2
    Internal route preference: 18
    External route preference: 165
    Prefix export limit: 666, Prefix export count: 24
    Wide metrics are enabled
```
## Terminology: `CLNP <-> NSAP + NSEL = NET{AFI.AID.SysID,NSEL}`
- ES -- End System == PC/Server
- IS -- Intermediate System == Router/MLS -- One ISIS router belongs to only one area.
- DIS -- Designated IS (no Backup DIS),`Priority 0 - 127(default 64, tie breakers: MAC/SNPA)`, All IS can reach each other assuming full conectivity. DIS election is deterministic and premption enabled by default.[DIS Adjs](./ISIS-DIS.png)
- NET -- Netowork Entity Title. Most of ES and IS have one NET. NET is identical to NSAP that has NSEL 00. `NET = NSAP + NSEL`
- CLNS -- Connectionless Network Service.
- CLNP -- Connectionless Network Protocol address - (aka OSI Addr == IP Addr) and when it is assigned to a IS Router it is called NSAP. Only one NSAP address needed per node, not per Interface. 
- NSAP -- Network Service Access Point, IS-IS supports multiple NSAP addresses on the loopback lo0 interface.`Oposite to OSPF, in ISIS a single node can belong to multiple areas(Probably an internal router since ABRs belong or have interfaces to diffent areas -???`
- NSEL -- NSAP Selector -- Always set to `00` for an ISIS Router. Represent the roll of device in the network.(IS or ES) 
- NLPID -- TLV carries the `Network Layer Protocol ID` of the supported protocols  -- `IP == 0x81`. 
- TLV -- Type-Leght-Value fields are used in order to facilitate communication using the same packet format but carring different content/purpose.
- SNPA -- The subnetwork point of attachment (SNPA) indicates the data link address of the next hop.

- Hierarchy: [ISIS ](./ISIS-Hierarchy.png)
>> - areas (Borders between IS)
>> - L1 IS -- within an area
>> - L2 IS -- between areas

- TLV field{variable}
>> - Object Type       -- a predefined code in table below... `2+128 and for extended metric 22+135, to surpress legasy metric needs to enable wide-metric-only == 22+135 this because ISs prefer legasy metric since both are inculded in LS-PDUs(by default)`
>> - Object Length     -- the length of the information
>> - Object value      -- the information 

  ![ISIS TLVs](./ISIS-TLVs.png)
    
- Hello Packet Types:
>> - Hello(ESH, ISH, IS-IS Hello [IIH]) -- to form and maintain Adjacencies, choose a DIS(DR) and verify neighbor visibility similar to OSPF.  
    >> - ES-Hello -- sent by an ES to an IS -- In the TCP/IP world the equivalent to the job that ES-IS does is carried out by ARP, DHCP, IRDP or by setting a default gateway.
    >> - IS-Hello -- sent by an IS to an ES
    >> - II-Hello -- used between two ISs
>> - P2P Hello PDUs -- Unicast L1-L2 on point-to-point type interfaces, for efficiency reasons, a single L1-L2 Hello, also called a point-to-point Hello, is used. 
```
NOTE: -- Each router sends Hello packets every 9 seconds by default, DIS each 3 sec.;
      -- Hold time as the Hello value multiplied by the Hello multiplier value.
      -- The default Hello multiplier value is 3, resulting in a Hold time of 9(DIS) and 27(NON-DIS) seconds. 
```
- Packets to form/maintain Adjacencies and LSDB:
>> - IIH  -- Hello in OSPF
>> - CSNP -- like DD in OSPF -- Complete sequence number PDU — `Used to distribute a router's complete link-state database` -- Sent periodic 10 sec. in broadcast networks, in p2p is sent at 5 sec. interval.   
>> - PSNP -- like LSReq||LSAck in OSPF -- Partial sequence number PDU — `Used to acknowledge and request link-state information`
>> - LS-PDU -- like LSU in OSPF to share multiple LSAs(exept in ISIS has only one LS(A|P) that contain multiple TLVs); -- Separate PDUs for L1/L2 adjs. -- LSP — `Used to distribute link-state information`
   
- Link State PDU(LSP)    - like LSA in OSPF. LSP number starts with 0x00 and increments 1 each time for following fragment.
>> - Level 1 PDU type == 18 {in the LSP header} - Intra-Area only
>> - Level 2 PDU type == 20 {in the LSP header} - Intra-Area or Inter-Area

>> - LSP -- like LSU in OSPF to share multiple LSAs; exept in ISIS is only one LSP that has multiple TLVs
>>       -- DIS generates CSNPs in LAN periodicaly and PSNP as LSR, on P2P links PSNP are used as LSAck:
    >>      - CSNP -- Complete SNP == DD of OSPF  *//sent periodicaly on P2P segments and on LAN sent only by DIS(DR) sent periodically Broadcast 10sec P2P 5 sec  
    >>      - PSNP -- Partial SNP similar to CSNP except it is not compleate but partial(see MTu limitation) == DD of OSPF

- Area Router types:
>> - Level 1    -- (Stub) a router from a non backbone area which has interfaces only in the one area. Maintains routing information for the area it belongs.
>> - Level 1/2  -- (ABR) a router at the boundary of one area or toward other Area, *set attached bit(ATT) in L1 PDU(LSP) to indicate that it is attached to L2 backbone area and can reach inter-area prefixes. Also, the routers injects default route in the L1 area.
>> - Level 2    -- (Backbone) a backbone router routes toward other ASes or Areas. Maintains routing information only for BACKBONE Area. 
    
```
Summary:
            Level 1/2: ABRs ...
            Level 2: Backbone
            Level 2: Adjacencies can be in different areas
            Level 1: Has to be in the same area
            Level 1: Routers just have a default route towards the backbone 

SET bits: ATT if ABR, OL - Overload bit if LSDB is overloaded(lack of memory), IS type bit 0x1 = L1 router. 0x2 = L2, 0x3 = L1/2 router, indicates circuit type(!= CirID).

Circuit Types - "L" Bit mask of levels on this interface:
                00: passive          -- Type 0 -- ES to IS -- or Stub Routing in OSPF
                01: Level-1 router   -- Level 1 
                10: Level-2 router   -- Level 2
                11: Level-1-2 router -- Level 3
Circuit ID: 0xNN
Note: Each IS-IS interface is assigned a circuit ID value to identify the interface within the linkstate database. 
All interfaces (loopback, broadcast, and so on) and all point-to-point links share the locally significant value of 0x01, and this value is not incremented.
```
>> - ABRs belongs to only one area, opposite to OSPF where ABRs have interfaces to multiple AREAs.
>> - DIS == DR  -- in IS-IS doesnt exist BDR - default priority 64 { 0 - 127 }. Preempting is by default and if a new router with a higher priority is added, it takes precedence. Area ID and router type must match in a brodcast multiaccess network.
>> - NON-Broadcast -- ISIS doesnt support this type of networks, `for FR/ATM P2p link types must be configured.`            
- AREA IDs: ex: 49.0001.1921.6801.6001.00   -- `*//192.168.16.1 and usually configured on Lo0`
    >> - 49.0001    -- *// area ID AFI 49(1byte)is private range of IP address like RFC1819 and Area number is 0001(2bytes but has no limitation to 2bytes only)
    >> - 1921.6801.6001 -- *// system identifier(6bytes) == OSPF RID
    >> - 00         --*//NET Selector(NSEL), always 00 when the device is a router which indicate "this system" 

- ISIS routes/forward packets by checking destination address:
    1. if destination is in a different area -- routes based on area addres
    2. if destination is in the same area -- routes based on system-id
    3. if it is a L1 router:
        - inter-area packets sent to closet L1/L2 router
        - intra-area packets are routed based on L1 database
    4. if it is a L2 router:
        - inter-area packets are routed based on L2 database
        - intra-area packets are routed based on L1 database   

- Supported Network/Link Types:
    1. Broadcast        -- Multicast address
    2. Point-to-Point   -- Unicast address
```
NOTE: recomended to use P2P subinterfaces on NBMA networks type. 
```
#### Troubleshooting Adjacency:
- Formed adjacency
  - LAN IP subnet must match
  - P2P the address can be unnumbered or /32
  - JunOS needs IP portion of network to function so that IS-IS adjacency will work.
- Unformed adjacency
  - Layer 1 and 2 problem
  - AREA id mismatch in a Level 1 area type
  - minimum supported MTU of 1492
  - Lack of IP configuration on interfaces -- `The adjacencies will stuck in Initializing state if no family INET configured on ISIS interfaces`
  - Lack of ISO-NET configured.
- Neighbor states:
  - DOWN          --Neighbor discovery 
  - New           --Neighbor discovery
  - Two-Way       --Neighbor discovery bidirectional communication
                    
  - Initializing  -- LSDB synchronization Will stuck in this state if MTU is smaller on one side on the other side is ignoring hello packets.
  - UP            -- Initial SPF computation
  - DOWN          -- when there is an area mismatch, hold time expired, authentication failure
  - Rejected      -- also for when authentication doesnt match – it cycles between Down and Reject
```
NOTE: All neignbors in IS-IS form full Adjacencies 
```
### Troubleshooting commands:
```
show isis overview
show isis adjacency
show isis interface
show isis hostname
show isis routes
show isis database [ detail | extensive ] 
show isis database vMX3 extensive | find TLV | match prefix 
show isis spf [ log | brief | result ]
show isis statistics

Debuging:
    set protocols isis traceoptions file isis-log size 10m files 5
    set protocols isis traceoptions flag error detail
    set protocols isis traceoptions flag hello sent receive detailed
    set protocols isis traceoptions flag spf
```
#### ISIS Tunning commands:
```
set protocols isis interface ge-1/0/1.0 level 1 disable        *//allow only L2 Adj on interface. 
set protocols isis interface ge-1/0/2.0 level 2 disable
set protocols isis interface ge-1/0/1.0 point-to-point 
set protocols isis interface ge-1/0/1.0 level 2 metric 10
set protocols isis interface ge-1/0/2.0 level 1 metric 5
                
set protocols isis interface lo0.0 level 2 disable
set protocols isis interface lo0.0 level 1 metric 5         *//by default, lo0's metric is 0 
            
set protocols isis reference-bandwidth 1g                  *// depends on used interface types: 1G / 10G / 40G / 100G
```
----
### vLABs -- Paste in Browser CTRL+ALT+SHIFT && RIGHT CLICK

#### ISIS Single-Area: 
![ISIS Single Area](./vLAB-ISIS-SingleArea.png)

#### R1:
```
delete protocol isis

*//NOTE: unlike OSPF in ISIS loopback interface is always passive and doesn't need to configure it manualy.
set interfaces lo0 unit 0 family inet address 10.100.100.1/32
set interfaces lo0 unit 0 family iso address 49.0001.1010.0100.0001.00

set interfaces ge-0/0/0 unit 0 family inet address 10.100.12.1/24
set interfaces ge-0/0/0 unit 0 family iso
set interfaces ge-0/0/1 unit 0 family inet address 10.100.14.1/24
set interfaces ge-0/0/1 unit 0 family iso
set interfaces ge-0/0/2 unit 0 family inet address 10.100.13.1/24
set interfaces ge-0/0/2 unit 0 family iso

set routing-instances BACKBONE instance-type virtual-router
set routing-instances BACKBONE interface lo0.0
set routing-instances BACKBONE interface ge-0/0/0.0
set routing-instances BACKBONE interface ge-0/0/1.0
set routing-instances BACKBONE interface ge-0/0/2.0

set routing-instances BACKBONE protocols isis interface ge-0/0/0.0 point-to-point
set routing-instances BACKBONE protocols isis interface ge-0/0/1.0 point-to-point
set routing-instances BACKBONE protocols isis interface ge-0/0/2.0 point-to-point

show | compare | no-more

commit and quit
```
#### R2:
```
delete protocol isis

*//NOTE: unlike OSPF in ISIS loopback interface is always passive and doesn't need to configure it manualy.
set interfaces lo0 unit 0 family inet address 10.100.100.2/32
set interfaces lo0 unit 0 family iso address 49.0001.1010.0100.0002.00

set interfaces ge-0/0/0 unit 0 family inet address 10.100.12.2/24
set interfaces ge-0/0/0 unit 0 family iso
set interfaces ge-0/0/1 unit 0 family inet address 10.100.23.1/24
set interfaces ge-0/0/1 unit 0 family iso
set interfaces ge-0/0/2 unit 0 family inet address 10.100.24.1/24
set interfaces ge-0/0/2 unit 0 family iso

set routing-instances BACKBONE instance-type virtual-router
set routing-instances BACKBONE interface lo0.0
set routing-instances BACKBONE interface ge-0/0/0.0
set routing-instances BACKBONE interface ge-0/0/1.0
set routing-instances BACKBONE interface ge-0/0/2.0

set routing-instances BACKBONE protocols isis interface ge-0/0/0.0 point-to-point
set routing-instances BACKBONE protocols isis interface ge-0/0/1.0 point-to-point
set routing-instances BACKBONE protocols isis interface ge-0/0/2.0 point-to-point

show | compare | no-more

commit and quit
```
#### R3:
```
delete protocol isis

*//NOTE: unlike OSPF in ISIS loopback interface is always passive and doesn't need to configure it manualy.
set interfaces lo0 unit 0 family inet address 10.100.100.3/32
set interfaces lo0 unit 0 family iso address 49.0001.1010.0100.0003.00

set interfaces ge-0/0/0 unit 0 family inet address 10.100.34.1/24
set interfaces ge-0/0/0 unit 0 family iso
set interfaces ge-0/0/1 unit 0 family inet address 10.100.23.2/24
set interfaces ge-0/0/1 unit 0 family iso
set interfaces ge-0/0/2 unit 0 family inet address 10.100.13.2/24
set interfaces ge-0/0/2 unit 0 family iso

set routing-instances BACKBONE instance-type virtual-router
set routing-instances BACKBONE interface lo0.0
set routing-instances BACKBONE interface ge-0/0/0.0
set routing-instances BACKBONE interface ge-0/0/1.0
set routing-instances BACKBONE interface ge-0/0/2.0

set routing-instances BACKBONE protocols isis interface ge-0/0/0.0 point-to-point
set routing-instances BACKBONE protocols isis interface ge-0/0/1.0 point-to-point
set routing-instances BACKBONE protocols isis interface ge-0/0/2.0 point-to-point

show | compare | no-more

commit and quit
```
#### R4
```
delete protocol isis

*//NOTE: unlike OSPF in ISIS loopback interface is always passive and doesn't need to configure it manualy.
set interfaces lo0 unit 0 family inet address 10.100.100.4/32
set interfaces lo0 unit 0 family iso address 49.0001.1010.0100.0004.00

set interfaces ge-0/0/0 unit 0 family inet address 10.100.34.2/24
set interfaces ge-0/0/0 unit 0 family iso
set interfaces ge-0/0/1 unit 0 family inet address 10.100.14.2/24
set interfaces ge-0/0/1 unit 0 family iso
set interfaces ge-0/0/2 unit 0 family inet address 10.100.24.2/24
set interfaces ge-0/0/2 unit 0 family iso

set routing-instances BACKBONE instance-type virtual-router
set routing-instances BACKBONE interface lo0.0
set routing-instances BACKBONE interface ge-0/0/0.0
set routing-instances BACKBONE interface ge-0/0/1.0
set routing-instances BACKBONE interface ge-0/0/2.0

set routing-instances BACKBONE protocols isis interface ge-0/0/0.0 point-to-point
set routing-instances BACKBONE protocols isis interface ge-0/0/1.0 point-to-point
set routing-instances BACKBONE protocols isis interface ge-0/0/2.0 point-to-point

show | compare | no-more

commit and quit
```
#### ISIS Multi-Area: 
![ISIS Multi Area](./vLAB-ISIS-MultiArea.png)
#### R1:
```
delete protocol isis

```