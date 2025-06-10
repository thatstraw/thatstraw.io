# Configuring Vlans
# VLANS

```jsx
// Define a vlan

> configure
#  set vlans NAME vlan-id ID description DESCRIPTION
# run show vlans

// set interface mode
# set interfaces ge-0/0/0 unit 0 family ethernet-switching port-mode trunk

/// set vlan members
# set interfaces ge-0/0/0 unit 0 family ethernet-switching vlan members (vlan_name | [vlan list]

C// Configuring an access vlan is just the same except that you only have one interfaces

c// you can copy configuration from one interface to another interface

## copy interfaces ge-0/0/0 to ge-0/0/1
#
## replace pattern Workstations with Access

#// Enable tagging for sub-interfaces

## set interface ge-0/0/0 vlan-tagging
#
/// Set vlan-id for the subinteface unit

# set interfaces ge-0/0/0 unit 10  vlan ID 10

# set interfaces ge-0/0/0 unit 0 family inet address 10.10.10.23/24
```

## Configuring Voice Vlan

```jsx
// Define the voice vlan

# set vlans Voice vlan-id 50

// apply the voice vlan to ports

# set switch-options voip interface ge-0/0/0 vlan Voice

// or
# set ethernet-switching-options voip interface ge-0/0/0 vlan Voice

```

Sure! Let’s break everything down step by step so it’s easy to understand, starting with **what an IRB is**, then moving on to **how to configure SVIs (Switched Virtual Interfaces)** on a **modern Juniper switch** running **Junos OS**.

---

## 🌐 What is an IRB in Juniper?

**IRB** stands for **Integrated Routing and Bridging**.

In simple terms:

- It's a **virtual interface** that connects a Layer 2 VLAN (switching) to a Layer 3 interface (routing).
- This allows **hosts in a VLAN to communicate with other VLANs or routed networks** via the switch itself.

If you’re familiar with Cisco, the IRB interface in Juniper is equivalent to an **SVI (Switched Virtual Interface)** in Cisco.

### Why use IRB?

- You want devices in a VLAN to **get a default gateway** on the switch.
- You want your Juniper switch to **perform routing between VLANs** (inter-VLAN routing).
- You want a **Layer 3 interface** for a VLAN.

---

## 🔧 Configuring SVIs / IRB Interfaces on a Juniper Switch

Let’s assume:

- You have a switch with **VLAN 10** for Users and **VLAN 20** for Servers.
- You want to configure **SVIs** for both VLANs so that the switch can route between them.

We'll walk through:

1. Creating VLANs
2. Assigning VLANs to interfaces
3. Creating IRB interfaces for routing
4. Associating VLANs with IRBs
5. Enabling Layer 3 routing

---

### 1. **Create VLANs**

```
set vlans Users vlan-id 10
set vlans Servers vlan-id 20

```

This creates two VLANs:

- **Users** with VLAN ID 10
- **Servers** with VLAN ID 20

---

### 2. **Assign VLANs to Interfaces**

Let’s say:

- `ge-0/0/1` connects to a user PC
- `ge-0/0/2` connects to a server

Set them as access ports and assign them to VLANs:

```
set interfaces ge-0/0/1 unit 0 family ethernet-switching port-mode access
set interfaces ge-0/0/1 unit 0 family ethernet-switching vlan members Users

set interfaces ge-0/0/2 unit 0 family ethernet-switching port-mode access
set interfaces ge-0/0/2 unit 0 family ethernet-switching vlan members Servers

```

---

### 3. **Create IRB Interfaces (SVIs)**

Now create Layer 3 interfaces for both VLANs using the `irb` interface:

```
set interfaces irb unit 10 family inet address 192.168.10.1/24
set interfaces irb unit 20 family inet address 192.168.20.1/24

```

This creates:

- `irb.10` as the gateway for VLAN 10: `192.168.10.1`
- `irb.20` as the gateway for VLAN 20: `192.168.20.1`

---

### 4. **Link VLANs to IRB Interfaces**

You now bind each VLAN to its corresponding IRB:

```
set vlans Users l3-interface irb.10
set vlans Servers l3-interface irb.20

```

This tells Junos:

- VLAN 10 uses `irb.10` for Layer 3 services
- VLAN 20 uses `irb.20` for Layer 3 services

---

### 5. **Enable Routing (if needed)**

If you want the switch to route between VLANs (which is usually the case for SVIs), make sure the routing instance (usually `default`) is active.

Most EX/QFX switches automatically support basic routing if the virtual router is enabled, but you can ensure it's active:

```
show routing-options

```

You don’t always need to configure this unless you're doing advanced routing.

---

## ✅ Verification Commands

After configuration, use these commands to verify:

```bash
show vlans
show interfaces irb
show configuration interfaces irb
show ethernet-switching table
show route

```

You can also ping from one VLAN to another to test inter-VLAN routing.

---

## 🧠 Quick Summary

| Term | Meaning |
| --- | --- |
| VLAN | Logical broadcast domain |
| IRB | Virtual L3 interface for routing VLAN traffic |
| SVI | Cisco term for IRB (same idea) |
| `irb.x` | Interface for VLAN x, acts as default gateway |
| `set vlans VLAN_NAME l3-interface irb.x` | Binds VLAN to its routed IRB |

---

Let me know if you want an example with trunk ports, DHCP, or dynamic routing on top of this!

configure voic vlan

![image.png](attachment:38c4f78a-8e46-4ce8-8a8c-e864c5acb052:image.png)