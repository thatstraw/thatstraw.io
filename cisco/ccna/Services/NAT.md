## Private IPv4 Addresses (RFC 1918)

- IPv4 doesn't provide enough addresses for all devices that need an IP address in the modern world.
- The long-term solution is to switch to IPv6
- There are three main short-term solution:
    1. CIDR
    2. Private IPv4 addreses
    3. NAT
- RFC 1918 specifies the following IPv4 address ranges as private:
    1. 10.0.0.0/8 (10.0.0.0 to 10.255.255.255) → CLASS A
    2. 172.16.0.0/12 (172.16.0.0 to 172.31.255.255) → CLASS B
    3. 192.168.0.0/16 (192.168.0.0 to 192.168.255.255) → CLASS C
- You are free to use these addresses in your networks. They don’t have to be globally unique.

```
Private IP address cannot be used over the internet!
```

```
Problems Without NAT
1. Duplicate addresses

2. Private IP addresses can’t be used over the the internet, so the PCs can’t access the internet
```

## Network Address Translation (NAT)

- Network Address Translation (NAT) is used to modify the source and/or destination IP addresses of packets.
- There are various reasons to use NAT, but the most common reason is to allow hosts with private IP addresses to communicate with other hosts over the internet.
- There are different types of NAT, for the CCNA you have to understand source NAT and how to configure it on cisco routers.

### Static Source NAT

- Static NAT involves statically configuring one-to-one mappings of private addresses to public IP addresses.


```
It doesn’t have to be private to public, you can NAT any address to any other address. But to keep it simple let’s think of it as mapping 1 private  IP  to 1 public IP.
```


- An inside local IP address is mapped to an inside global IP address.
    
    → Inside Local = The IP address of the inside host, from the perspective of the local network.
                                   * the IP address actually configured on a the inside host, ussually a private                       address.
    
    → Inside Global  =The IP address of the inside host, from the perspective of outside hosts.
    
    *the ip address of the inside host after NAT, ussually a public address.
    



```
Static NAT allows devices with private IP addresses to communicate over the internet. However , because it requires a one-to-one IP address mapping, it doesn’t help preserve IP addresses.
```


### Configuring Static NAT

```bash

# Define the insdie interface(s) connected to the internal network
R1(config)# int g0/1
R1(config-if)# ip nat inside

Define the outside interface(s) connected to the external network
R1(config)# int g0/0
R1(config-if)# ip nat outside

# configure the one to one ip address mapings (ip nat inside source static insdie local IP
inside global ip)
R1(config)# ip nat inside source static 192.168.0.167 100.0.0.1
R1(config)# ip nat inside source static 192.168.0.168 100.0.0.2

R1# show ip nat translation
```

→ Outside Local = The ip address of the outside host, from the perpective of the local network.

→ Outside Global = The IP address of the outside host, from the perspective of the outside network.

The values for these will always be the same, unless destination NAT is used.

```bash
# Clear dynamic ip nat translation
R1# clear ip nat translation *

# Display nat statics
R1# show ip nat statics
```