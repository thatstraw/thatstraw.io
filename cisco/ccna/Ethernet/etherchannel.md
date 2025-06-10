> When the bandwidth of the interfaces connected to end hosts is greater than the bandwidth of the connection to the distribution switch(es), this is called oversubscription. Some oversubscription is acceptable, but too much will cause congestion.
> 

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/a6cd1675-c24e-465e-96c9-73c21c6481c7/b256f846-7f50-4c25-95a8-fbf1d1d9711a/image.png)

- If you connect two switches together with multiple links, all except one will be disabled by spanning tree.
- If all of ASW1’s interfaces were forwarding, Layer 2 loops would form between the ASW1 and DSW1, leading to broadcast storms.
- Other links will be unused unless the active link fails. In that case, one of the inactive links will start forwarding.

## Etherchannel

- EtherChannel groups multiple interfaces together to act as a single interface.
- STP will treat this single group as a single interface.
    
```  
    💡 Traffic using the EtherChannel will be load balanced among the physical interfaces in the group. An algorithm is used to determine which traffic will use which physical interface. More details on this later!
```
    
- Some other names for an EtherChannel are:
    - Port Channel
    - LAG (Link Aggregation Group)

### EtherChannel Load-Balancing

- Etherchannel load balances based on flows.
- A flow is a communication between nodes in the network.
- Frame in the same flow will be forwarded using the same physical interface.
- If frames in the same flow were forwarded using different physical interfaces, some frames may arrive at the destination out of order, which can cause problems. Some applications can deal with frames arriving out of order but some applications can’t.
- Etherchannel load balances by using different physical interfaces in the etherchannel for different flows.
- You can change the inputs used in the interface selection calculation.
- Inputs that can be used:
    - Source MAC
    - Destination MAC
    - Source and Destination MAC
    - Source IP
    - Destination IP
    - Source and Destination IP

## Etherchannel Configuration

**Configuring EtherChannel Load-Balancing**

```jsx
# display the default method of load-balancing being used.
SW1# show etherchannel load-balance

SW1# conf t
SW1(config)# port-channel load-balance src-dst-mac
```

```
💡 Notice that when showing etherchannel configuration on a cisco switch you use the etherchannel and when configuring etherchannel you use port-channel instead.

```

**Configuring EtherChannel**

- There are three methods of EtherChannel configuration on a cisco switch:
- **PAgP (Port Aggregation Protocol)**
    
    → Cisco proprietary protocol
    
    → Dynamically negotiates the creating/**maintenance** of the EtherChannel (like DTP does for trunks).
    
- **LACP (Link Aggregation Control Protocol)**
    
    → Industry standard protocol (802.3ad)
    
    → Dynamically negotiates the creating/**maintenance** of the EtherChannel (like DTP does for trunks).
    
- Static EtherChannel
    
    → A protocol isn’t used to determine if an EtherChannel should be formed.
    
    → Interfaces are statically configured to form an EtherChannel.
    
- Up to 8 interfaces can be formed into a single EtherChannel (LACP allows up to 16, but only 8 will be active, the other 8 will be in standby mode, wating for an active interface to fail.)

### PAgP Configuration

```jsx
SW1# config t

# Selecting a range of interfaces you want to configure etherchannel on
SW1(config)# interface range g0/0-4

# **channel-group** *number* **mode** *mode*

SW1(config-if)# channel-group 1 mode ? # display context sensitive help
SW1(config-if)# channel-group 1 mode diserable
```


```
💡 The channel group number has to match for member interfaces on the same switch. However, it doesn’t have to match the channel-group number on the other switch.

(channel-group 1 on ASW1 can form an EtherChannel with channel-group 2 on DSW1)

The number is just used to identify the virtual interface on a switch. Because you can have multiple etherchannels on a single swtich. You need this number to identify them.
```


**Modes for forming etherchannel using PAgP**

```jsx
auto + auto = no EtherChannel
desirable + auto = EtherChannel
desireable + desirable = Etherchannel
```

### LACP Configuration

```jsx
SW1# config t

# Selecting a range of interfaces you want to configure etherchannel on
SW1(config)# interface range g0/0-4

# **channel-group** *number* **mode** *mode*

SW1(config-if)# channel-group 1 mode ? # display context sensitive help
SW1(config-if)# channel-group 1 mode active
```

**Modes for forming etherchannel using LACP**

```jsx
passive + passive = No EtherChannel
passive + active = EtherChannel
active + active = EtherChannel
```

### Static Configuration

```jsx
SW1# config t

# Selecting a range of interfaces you want to configure etherchannel on
SW1(config)# interface range g0/0-4

# **channel-group** *number* **mode** *mode*

SW1(config-if)# channel-group 1 mode ? # display context sensitive help
SW1(config-if)# channel-group 1 mode on
```

**Modes for forming etherchannel using Static**

```jsx
On mode only works with on mode (on + diserable or on + active will not work)
```

Another command you should know is the `channel-protocol` command it allows you to specify which protocol (PAgP or LACP) should be used for the EtherChannel. This manually configures the etherchannel negotiation protocol that the member interfaces should use. This is actually not a very useful command because you don’t need to configure it, because if you configure the mode to be desirable  or auto, the intherface will automatically use PAgP. Same for active or passive the interface will use LACP.

**Configuring PortChannel interface as trunk**

```jsx
SW(config)# int port-channel 1
SW(config-if)# switchport trunk encapsulation dot1q
SW(config-if)# switchport mode trunk
SW(config-if)# do shw int trunk
```

### More important things to know about EtherChannel

- Member interfaces must have matching configurations
    
    → Same duplex (full/half)
    
    → Same speed
    
    → Same switchport mode (access/trunk)
    
    → Same allowed VLANS/native VLAN (for trunk interfaces)
    
- If an interface’s configuraiton do not match the others, it will be excluded from the EtherChannel

**Verify etherchannel summary**

```jsx
SW1# show etherchannel summary
SW1# show etherchannel port-channel
```

## Layer 3 EtherChannel

```
💡 Routed ports don’t forward layer 2 broadcasts frames so no layer 2 loops can be formed
```

**Configuring layer 3 etherchannel**

```jsx
SW1# config t

# Selecting a range of interfaces you want to configure etherchannel on
SW1(config)# interface range g0/0-4

# This converts these interfaces into routed ports
SW1(config-if)# no switchport

# **channel-group** *number* **mode** *mode*

SW1(config-if)# channel-group 1 mode ? # display context sensitive help
SW1(config-if)# channel-group 1 mode on

# Since this is a layer 3 etherchannel we need to configure the ip address
SW1(config)# int pot1
SW1(config-if)# ip address 192.168.1.5 255.255.255.0

```