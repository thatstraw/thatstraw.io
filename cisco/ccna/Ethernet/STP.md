# Spanning Tree Protocol

The Spanning Tree Protocol (STP) is a network protocol that ensures a loop-free topology for Ethernet networks. STP is crucial in preventing broadcast storms and ensuring redundancy in network paths. By creating a spanning tree that logically disables redundant paths, STP allows for a robust and efficient network design. This protocol helps maintain a stable and reliable network by dynamically detecting and mitigating potential loops and ensuring that there is always a single active path between any two network devices.

## Creating a spanning tree topology

- Switches use one field in the STP BPDU, the Bridge ID field, to elect the a root bridge for the network.
- The switch with the lowest bridge ID becomes the root bridge.
- All ports on the root bridge are put in a forwarding state, and other switches in the topology must have a path to reach the root bridge.

<aside>
💡 The default bridge priority is **32768** on all switches, so by default the MAC address is used as the tie breaker ( lowest MAC address becomes the root bridge)

**Important:
The bridge priority is compared first, If they tie , the MAC address is then compared.**

</aside>

**STP interface cost**

| Speed | STP Cost |
| --- | --- |
| 10 Mbps | 100 |
| 100 Mbps | 19 |
| 1 Gbps | 4 |
| 10 Gbps | 2 |

Important it is important to use long Path Cost of 20Tbps/ port speed.

<aside>
💡 The ports connected to another swtich’s root port must be designated. Because the root port is the switch’s path to the root bridge, another switch must not block it.

</aside>

<aside>
💡 Another important point to remember
Every collision domain has a single STP designated port

</aside>

### Basic STP commands

```jsx
# show the mac address of the switch used in spanning tree

$ show spanning-tree bridge

# display information about the spanning tree
$ show spanning-tree

# display spanning tree info for a specif vlan
$ show spanning-tree vlan <ID> ?detail

# display detailed info about spanning tree

$ show spanning-tree detail

# display spanning tree summary
$ show spanning-tree summary
```

### Summary for stp topology

1. The switch with the lowest BID is elected as the root bridge. All ports on the root bridge are designated ports (forwarding).
2. Each remaing switch will select ONE of its interfaces to be its root port. The interfaces with the lowest root cost will be the root port. Root ports are also in a forwarding state.
    
    Root port selection:
    
    1: Lowest root cost
    
    2: Interface connected to the lowest neighbor bridge ID (Lowest neighbour Bridge ID)
    
    3: Interface connected   on a neighbour switch with lowest port ID (Lowest neighbour Bridge   port ID)  `STP Port ID = port priority (default 128) + port number`
    
3. Designated port selection- Each remaining collision domain will select ONE interface to be the designated port (forwardig state). The other port in the collision domain will be non-designated (blocking).
    
      1: The switch with the lowest root cost will make its port designated
    
    1. If the root cost is the same the switch with the lowest bridge ID will make its port designated
    2. The other switch will make its port non-designated (blocking) 

### Spanning tree port states

| STP Port State | Stable/Transitional | Time |
| --- | --- | --- |
| Blocking | Stable | 20s |
| Listening | Transitional | 15s |
| Learning | Transitional | 15s |
| Forwarding | Stable |  |
- Root/Designated ports remain stable in the forwarding state.
- Non-designated ports remain stable in a blocking state.

<aside>
💡 Note: they remain stable if they are no changes in the network topology. If a new devices is added, an interface is shutdown, or a hardware failure occurs somewhere, they may have to change state.

</aside>

- Listening and Learning are transitional states which are passed through when an interface is activated, or when a Blocking port must transition to a Forwarding state due to a change in the network topology.

### Blocking State - Stable

- Non-designated ports are in a Blocking state
- Interfaces in a blocking state are effectively disabled to prevent loops.
- Interfaces in a Blocking state do not send/receive regular network traffic.
- Interfaces in a Blocking state receives STP BPDUs. ( They need to receive and process BPDUs to be aware of the spanning tree topology and be ready to transition torwad a forwarding state if they need to.)
- Interfaces in a Blocking state do NOT forward STP BPDUs.
- Interfaces in a Blocking state do NOT learn MAC Addresses. (If regular traffic arrives on the interface it is dropped without adding the mac address to the mac address table.)

### Listening State - Transition

- After the Blocking state, interfaces with the Designated or Root role enter the Listening state.
- Only Designated or Root ports enter the Listening state (Non designated ports are always Blocking)
- The Listening state is 15 seconds long by default. This determined by the Forward delay timer.
- An interface in the Listening state only forwards/receives STP BPDUs.
- An interface in the Listening state does NOT send/receive regular traffic. (If regular unicast frame is recieved on a port in the Listening state, it will be discarded.)
- An interface in the Listening state does NOT learn Mac address from regular traffic that arrives on the interface.

### Learning state - Transition

- After the Listening state, a Designated or Root port will enter the Learning state.
- The Learning state is 15 seconds long by default. This is determined by the Forward delay timer (the same timer used for both the Listening and learning states).
- An interface in the Learning state only forwards/receives STP BPDUs.
- An interface in the Learning state does NOT send/receive regular traffic. (If regular unicast frame is recieved on a port in the Listening state, it will be discarded.)
- An interface in the Learning state learns Mac address from regular traffic that arrives on the interface.

### Forwarding State - Stable

- Root and designated ports are in a Forwarding state.
- A port in a forwarding state operates as normal.
- A port in the forwarding state sends/receives BPDUs.
- A port in the forwarding state sends/recieves normal traffic
- A port int the forwarding state learns mac addresses  from frames that arrives on it.

## Spanning Tree Port states

| STP Port State | Send/Receves BPDUs | Frame forwarding 
  (regular traffic) | Mac Address 
learning | Stable/Transitional |
| --- | --- | --- | --- | --- |
| Blocking | NO/YES | NO | NO | Stable |
| Listening | YES/YES | NO | NO | Transitioning |
| Learning | YES/YES | NO | YES | Transitioning |
| Forwarding | YES/YES | YES | YES | Stable |
| Disabled | NO/NO | NO | NO | Stable |

## Spanning Tree Timers

| STP Timer | Purpose | Duration |
| --- | --- | --- |
| Hello | How often the root bridge the sends the BPDUs. | 2 Seconds |
| Forward Delay | How long will the switch stay in the Listening and Learning state( each state is 15 seconds = total 30 seconds) | 15 Seconds |
| Max Age | How long will an interface wait to change the STP topology after ceasing to receive BPDUs. The timer resets every time a BPDU is received. | 20 Seconds (10* 2 seconds Hello) |

<aside>
💡 Important: Switches do not forward the BPDUs out of their root and non-designated ports, only their designated ports.

</aside>

> If another BPDU is recieved before the max age timer counts down to 0, the time will reset back to 20 seconds and no changes will occur.

If another BPDU is not received , the max age timer counts down to 0 and the switch will re-evaluate its STP choices, including the root bridge, and local root, designated, and non-designated ports.

These timers and transitional states are to make sure that loops aren’t accidentally created by an interface moving to forwarding state too soon.
> 

<aside>
💡 A forwarding interface can move directly to a blocking state (there is no worry about creating a loop by blocking an interface.)

A blocking interface cannot move directly to forwarding state. It must go through the listening and learning states.

</aside>

### Spanning tree BPDU

- Ethernet header - Dst: PVST+ uses a destination mac address of  (01:00:0c:cc:cc:cd) for it’s BPDUs.

<aside>
💡 PVST = Only ISL trunk encapsulation
PVST+ = Supports 802.1Q

</aside>

- Regular STP (not Cisco’s PVST+) uses a destination MAC address of 01:80:c2:00:00:00

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/a6cd1675-c24e-465e-96c9-73c21c6481c7/15caff25-d76e-49aa-86fb-aba2b53373ce/image.png)

The following list provides a brief explanation of each of the BPDU fields:

- Protocol ID: This value is always 0.
- Protocol Version: This value is always 0 for classic spanning tree.
- BPDU Type: This field determines which of the two BPDU formats this frame contains—configuration BPDU (0x00) or TCN BPDU (0x80).
- Flags: This field is used to handle changes in the active topology
- Root ID: This field contains the bridge ID (BID) of the root bridge. After convergence, all configuration BPDUs in the bridged network should contain the same value for this field (for a single VLAN). Some network sniffers break out the two BID subfields: bridge priority and bridge MAC address.
- Root Path Cost: This value is the cumulative cost of all links leading to the root bridge. If the value is 0 you know its a root bridge.
- Bridge ID (BID): This value is the identifier of the bridge that created the current BPDU. This field is the same for all BPDUs sent by a single switch (for a single VLAN), but it differs between switches. The BID is a combination of the sender bridge’s priority to become root or designated bridge and the bridge address (a unique MAC address for the bridge.)
- Port ID: The interface that sends the BPDU. This field contains a unique value for every port. This value is a combination of the outbound port’s priority and a unique value to represent the port. The default port priority is 128 for every interface on an EX Series switch. The switch automatically generates the port number and you cannot configure it. For example, ge-1/0/0 contains the value 128:513, whereas ge-1/0/1 contains the value 128:514.
- Message Age: This field records the time since the root bridge originally generated the information from which the current BPDU is derived.
- Max Age: This value is the maximum time that a BPDU is saved. It also influences the bridge table aging timer during the topology change notification process.
- Hello Time: This value is the time between periodic configuration BPDUs.
- Forward Delay: This value is the time a bridge spends in the listening and learning states. It also influences timers during the topology change notification process

### Spanning Tree Optional Features (STP Toolkit)

- Portfast - allows a port to move immediately to the Forwarding state, bypassing Listening and Learning. If used, it must be enabled only on ports connected to end hosts. If enable on a port connected to another switch it could cause a Layer 2 loop.

```jsx
# portfast is enabled at the interface level
SW1(config) interface g0/2
SW1(config-if) spanning-tree portfast edge 
# (you can omit the edge, it can be addeed automatically in the config)

# You can also enable portfast globaly   with the following command 
SW1(config) spanning-tree portfast default

# disable portfast on an interface if you enabled it by default
SW1(config-if) spanning-tree portfast disable
# This enables portfast on all access ports (not trunk ports). You can also enable portfast on trunks
# ports if you need. For example on a ROAS port.
SW1(config-if) spanning-tree portfast trunk
```

<aside>
💡 Important, port fast will only take effect when the interface is an a non-trunking mode. Meaning when the interface is an access port not trunk port. You can still configure portfast on a trunk port but it will not take effect.

</aside>

- BPDU Guard - if an interface with BPDU Guard enabled receives a BPDU from another switch, the interface will be shutdown to prevent loops from forming.

```jsx
# bpdu is enabled at the interface level
SW1(config) interface g0/2
SW1(config-if) spanning-tree bpduguard enable

# You can also enable bpdu guard globaly with the following command
SW1(config) spanning-tree portfast bpduguard default
# This enables BPDU guard on all portfast-enabled interfaces.
```

To enable an interface that was disabled by bpdu guard simply `shutdown` the interface and then `no shutdown` the interface.

- Uplinkfast  - When uplinkfast is enabled a **non-designated port will go to forwarding state immediately if the root port fails**. Instead of 30 seconds of downtime, connectivity is restored immediately.

```jsx
SW3(config)#spanning-tree uplinkfast
```

- Backbone fast

```jsx

```

**Other features that you don’t necessarily need to know for the CCNA**

| Root Guard | If you enable root guard on an interface, even if it receives a superior BPDU (lower bridge ID) on that interface, the switch will not accept the new switch as the root bridge. The interface will be disabled.

SW1(config-if)# spanning-tree guard root |
| --- | --- |
| Loop Guard | If you have enabled loop guard on an interface, even if the interface stops receiving BPDUs, it will not start forwarding. The interface will be disabled. |
|  |   |

## BPDUFilter

- A switch port connected to an end host continues sending BPDUs every 2 seconds.
    - regardless of whether PortFast and/or BPDU Guard are enabled
- If the port doesn’t connect to a switch, sending BPDUs is unnecessary and undesirable for a couple of reasons:
    1. Sending BPDUs uses some bandwidth and processing power on the switch (although it’s minimal).
    2. BPDUs contain information about the LAN’s STP topology.
    3. BPDU filter stops a port from sending BPDUs. Unlike bpduguard it does not disable the port if it recieves a BPDU
    
    BPDUFilter can be enabled in two ways
    
    ```jsx
    SW3(config)#spanning-tree bpdufilter enable
    
    # the port will not send BPDUs
    # the port will ignore any BPDUs it recieves
    # In effect, this disables stp on the port. Use with caution. 
    
    # A better choice is to enable bpdufilter in global config
    SW3(config)#spanning-tree portfast [edge] bpdufilter default
    
    # this will enable BPDUfilter on all portfast-enabled ports
    # the port will not send bpdus
    # if the port receives a bpdu, portfast and bpdu filter are disabled, an it operates as a normal stp port.
    ```
    
    Here's the text written in the image:
    
    - **My recommendation:**
        - Enable PortFast and BPDU Guard however you prefer (per-port or by default).
        - Only enable BPDU Filter by default (global config mode).
            - unless you have a very good reason to enable it per-port
    
    ### BPDU Guard and BPDU Filter can be enabled on the same port at the same time:
    
    - If **BPDU Filter is enabled in global config mode** and the port receives a BPDU:
        1. BPDU Filter will be disabled.
        2. BPDU Guard will be triggered (and err-disable the interface).
    - If **BPDU Filter is enabled in interface config mode** and the port receives a BPDU:
        - The BPDU will be ignored.
        - BPDU Guard will **not** be triggered.

## Basic Spanning tree configuration

```jsx
# changing the spanning tree mode
SW1(config) spanning-tree mode ?

# configuring a switch as the root bridge
# This command sets the STP priority to 24576. If another switch already has the 
# priority lower than 24576, it sets this switch priority to 4096 less than the other switch's
# priority.
SW1(config) spanning-tree vlan 1 root primary

# using the priority
SW1(config) spanning-tree vlan 1 priority <increments of 4096)

# Configuring a switch as the secondary root bridge
# This command sets the STP priority to 28672
SW2(config) spanning-tree vlan 1 root secondary

# Configure stp ports settings (cost and port priority). Not this is configured in 
# interface level not global config mode.Note that port-priority is configured in 
# increments of 32 (16?)

SW2(config-if) spanning-tree vlan 1 ?

```

## Rapid Spanning Tree

### Comparisons of STP versions (Standard vs Cisco)

| Industry Standards (IEEE) | Cisco Versions |
| --- | --- |
| Spanning Tree Protocol (802.1D
- The original STP
- All VLANS  share one STP instance.
-  Therefore, cannot load balance. | Per-VLAN Spanning Tree Plus+(PVST+)
- Cisco Upgrade to 802.1D
- Each VLAN has its own STP instance. 
- Can load balance by blocking different ports in each VLAN. |
| Rapid Spanning Tree Protocol( 802.1w)
- Much   faster at converging/adapting to network changes than 802.1D
- All VLANs share one STP instance.
Therefore, cannot load balance | Rapid Per-VLAN Spanning Tree Plus (Rapid PVST+)
- Cisco upgrade to 802.1w
- Each VLAN has its own STP instance
- Can load balance by blocking different ports in each VLAN. |
| Multiple Spanning Tree Protocol (802.1s)
- Uses modified RSTP mechanics. 
- Can group multiple VLANS into different instances (i.e VLANs 1-5 instance 1, VLANs 6-10 in instance 2 to perform load balancing.
- Much superior than the cisco standards.
- Easier to configure primary and second root bridges when you have too many VLANs.
- Cisco hasn’t developed their own version of MSTP, so cisco devices simply run the industry standard MSTP |  |

**Cisco summary on RSTP**

‘RSTP is not a timer-based spanning tree algorithm like 802.1D. Therefore, RSTP offers an improvement over the 30 seconds or more that 802.1D takes to move a link to forwarding. The heard of the protocol is a new bridge-bridge handshake mechanism, which allows ports to move directly to forwarding.

**Similarities between STP and RSTP**

- RSP servers the same purpose as STP, blocking specific ports to prevent Layer 2 loops.
- RSTP elects a root bridge with the same rules as STP.
- RSTP elects the root ports with the same rules as STP.
- RSTP elects designated ports with the same rules as STP.

| Speed | STP Cost | RSTP Cost |
| --- | --- | --- |
| 10 Mbps | 100 | 2,000,000 |
| 100 Mbps | 19 | 200,000 |
| 1 Gbps | 4 | 20,000 |
| 10 Gbps | 2 | 2000 |
| 100Gbps | X | 200 |
| 1Tbs | X | 20 |

## Rapid Spanning Tree Port states

| STP Port State | Send/Receives BPDUs | Frame forwarding 
  (regular traffic) | Mac Address 
learning | Stable/Transitional |
| --- | --- | --- | --- | --- |
| Blocking | NO/YES | NO | NO | Stable |
| Learning | YES/YES | NO | YES | Transitioning |
| Forwarding | YES/YES | YES | YES | Stable |
- If a port is administratively disabled (shutdown command) = discarding state (This was disabled state in STP)
- If a port is enabled but blocking traffic to prevent Layer 2 loops = discarding state (previously the blocking state)
- The root port role remains unchanged in RSTP.
    - → The port that is closest to the root bridge becomes the root port for the switch
    - → The root bridge is the only switch that doesn’t have a root port.
- The designated port role remains unchanged in RSTP.
    - → The port on a segment (collision domain) that sends the best BPDU is that segment’s designated port (only one per segment).
- The non-designated port role is split into two separate roles in RSTP:
    - the alternate port role
    - backup port role

### RSTP: Alternate port role

- The RSTP alternate port role is a discarding port that receives a superior BPDU from another switch.
- This is the same as what you’ve learned about blocking ports in classic STP.
- An alternate port basically functions as a backup to the root port.
- If the root port fails, the switch can immediately move its best alternate port to forwarding (as the new root port).

<aside>
💡 The immediate move to forwarding state functions like a clasic STP optional featuer call uplinkfast. Because it is built into RSTP, you do not need to activate UplinkFast when using RSTP/Rapid PVST+

</aside>

- One more STP optional feature than was built into RSTP is BackboneFast.
- BackboneFast allows a switch to expire the max age timers on its interface and rapidly forward the superior BPDUs.
- This functinality is built into RSTP, so it does not need to be configured.

**UplinkFast/BackboneFast Summary**

- UplinkFast and BackboneFast are two optional featues in classic STP. They myst be configured to operate on the swtich (not necessary to know for CCNA).
- Both features are built into RSTP, so you do not have to configure them. They operate by default.
- You do not need to have a detailed understanding of them for the CCNA. Know their names and their basic purpose ( to help blocking/discarding ports rapidly move to forwarding.)

### RSTP:  Backup port role

- The RSTP backup port role is a discarding port that recieves a superior BPDU from another interface on the same switch.
- This only happens when two interfaces are connected to the same collision domain (via hub).
- Hubs are not used in modern networks, so you will probably not encounter an RSTP backup port.
- RSTP backup port functions as a backup for a designated port.

<aside>
💡 As for how the swich chooses which interfaces should be the designated and which one should be the backup. The interface with the lowest port ID will be selected as the designated port, and the other will be the backup port.

</aside>

> Rapid STP is compatible with classic stp. The interface(s) on the Rapid STP-enabled switch connected to the classic stp-enabled switch will opearte in classic STP mode (timers, block→ listening→ learning→ forwarding process etc.)
> 

### STP and RSTP BPDU comparison

- RSTP Protocol Versition Identifier is 2 where is STP is 0
- BPDU type for RSTP is 0x02 where STP is 0x00
- RSTP uses all the 8 bits of the BPDU flags whilst STP  only uses the first bit and last bit.

<aside>
💡 Here is another major difference:
In clasic STP, only the root bridge originated BPDUs, and other switches just forwarded the BPDUs they recieved. In rapid STP, ALL the switches originate and send their own BPDUs from their designated ports.

All switches running STP send their own BPDUs every hello time (2 seconds)

Switches ‘age’ the BPDU information much more quickly. In classic STP, a switch waits hello intervals (20 seconds). In rapid STP, a switch considers a neighbor loss if it misses 3 BPDUs (6 seconds).  It will then flush all MAC addresses learned on that interface.

</aside>

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/a6cd1675-c24e-465e-96c9-73c21c6481c7/04c8e750-7b80-4e25-b1dd-c023c223ac5f/image.png)

```jsx
RSTP is backward compatible with STP. If a device configured for RSTP receives STP BPDUs, it reverts to STP. In a pure RSTP 
environment, a single type of the BPDU exists named Rapid Spanning Tree BPDU (RST BPDU). RST BPDUs use a similar format 
to the STP configuration BPDUs. RSTP devices detect the type of BPDU by looking at the protocol version and BPDU type fields. 
The BPDUs contain several new flags, as shown on the graphic. The following is a brief description of the flags:
• TCN Acknowledgment: This flag is used when acknowledging STP TCNs;
• Agreement and Proposal: These flags are used to help quickly transition a new designated port to the forwarding 
state;
• Forwarding and Learning: These flags are used to advertise the state of the sending port;
• Port Role: This flag specifies the role of the sending port: 0 = Unknown, 1 = Alternate or Backup, 2 = Root, and 3= 
Designated; and
• Topology Change: RSTP uses configuration BPDUs with this bit set to notify other switches that the topology has 
changed.
RST BPDUs contain a Version 1 Length field that is always set to 0x0000. This field allows for future extensions to RSTP.
```

### RSTP Link Types

- RSTP distinguishes between three different link types.
1. Edge: a port that is connected to an end host. Moves directly to forwarding, without negotiation.
    
    <aside>
    💡 Does this sounds familiar: Yes Portfast, this feature was built into RSTP. This is another STP optional feature that was built into RSTP.
    
    Edge ports are connected to end hosts.
    Because there is no risk of creating a loop, they can move straight to the forwarding state without the negotiation process. 
    
    They function like a classic STP port with PortFast enabled.
    
    </aside>
    
2. Point-to-Point: a direct connection between two switches.
    
    <aside>
    💡 Point-to-point ports connect directly to another switch.
    They function in full-duplex
    
    You don’t need to configure the interface as point-to-point (it should be detected)
    
    However if you want to automaticaly do that, you issue the following command:
    sw1(config-if)# spanning-tree link-type point-to-point
    
    </aside>
    
3. Shared a connection to a hub. Must operate in half-duplex mode.
    
    <aside>
    💡 Shared ports connect to another switch (or switches) via a hub
    
    They function in half duplex.
    
    You don’t need to configure the interface as shared (it should be detected)
    
    However if you want to automaticaly do that, you issue the following command:
    sw1(config-if)# spanning-tree link-type shared
    
    </aside>
    

# Configuring MSTP

```jsx

# mst configuration mode
SW1(config)# spanning-tree mst configuration

# creating instances
SW1(config-mst)# instance 1 vlan 100,300
SW1(config-mst)# instance 2 vlan 200,400

# Note there is an invincible catch all instance 0 that contains all vlans which were not explicitily assigned to an instance.

# making a switch a primary root bridge for an instance
SW1(config)# spanning-tree mst 1 root primary

# secondary
SW1(config)# spanning-tree mst 1 root secondary

# or using the priority
SW1(config)# spanning-tree mst 1 priority <priority>

# change the spanning tree mode:
SW1(config)# spanning-tree mode ?
```

## Debuging Spanning tree events

```jsx
SW3# debug spanning-tree events 
Spanning Tree event debugging is on

# show errdisable recovery
config # errdisable recovery interval <ms>
config # errdisable recovery cause bpduguard
```