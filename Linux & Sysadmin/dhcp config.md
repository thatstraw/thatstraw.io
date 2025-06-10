# configuring dhcp server on linux

### Installing the dhcp package:

```jsx
# debian/ubuntu
$ sudo apt-get install isc-dhcp-server

# redhat distros
$ sudo dnf install dhcp-server

```

### Configuration

```jsx
$ sudo cp /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.bak
$ sudo nano /etc/dhcp/dhcpd.conf
```

add a section for a subnet if it doesn’t already exist and modify it according to our network settings:

```jsx
subnet 192.168.1.0 netmask 255.255.255.0 {
  range 192.168.1.10 192.168.1.50;
  option subnet-mask 255.255.255.0;
  option routers 192.168.1.1;
  option domain-name-servers 8.8.8.8, 8.8.4.4;
  option domain-name "example.com";
}
```

### **Starting and Verifying DHCP Server**

```jsx

# debian
$ sudo service isc-dhcp-server start
$ sudo service isc-dhcp-server status

# redhat derivatives
$ sudo service dhcpd start
$ sudo service dhcpd status
```

## Troubleshooting

**Checking logs**

```jsx
sudo cat /var/log/syslog | grep dhcp
sudo journalctl -u isc-dhcp-server
sudo tail -f /var/log/syslog
```

**Checking for syntax errors in the dhcp config file**

```jsx
$ sudo dhcpd -t
```

### Ubuntu client network settings to use DHCP

To configure your client to use a DHCP on a network interface eth0 on Ubuntu or Debian Linux systems enter the following lines in your **`/etc/network/interfaces`** file:

```jsx
auto eth0
iface eth0 inet dhcp

```

### Key Points

1. **Subnet Declaration Requirement:**
    - The DHCP server needs a subnet declaration for every network it is directly connected to and listening on. This allows the server to understand the network topology and correctly handle DHCP requests.
2. **Relay Agent Configuration:**
    - When configuring a DHCP server to provide addresses for a subnet via a relay agent (like FW2), you still need a subnet declaration for the network to which the DHCP server is directly connected. This ensures the server can correctly receive and process the relayed requests.