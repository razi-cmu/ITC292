# Week 7: Networking in Linux

System administration is not just about managing a single machine but it's about connecting that machine to the world. Modern Linux systems rarely operate in isolation; they are part of local networks, connect to the Internet, and communicate with other systems constantly. A system administrator must understand how networks function, how to configure network interfaces, how to diagnose connectivity issues, and how to secure network services.

This week covers two interrelated pillars of system administration: intra-networking (communication within a local network) and inter-networking (communication between different networks). We will explore the Linux network stack, configuration tools, troubleshooting techniques, and the foundational protocols that make networked systems work.

## Networking Fundamentals

## OSI Model vs TCP/IP Model
Networking in Linux is built upon the TCP/IP model, which defines how data is transmitted across networks. While the OSI model has seven layers, the TCP/IP model condenses them into four/five layers:

| Feature | OSI Model | TCP/IP Model |
|---------|-----------|--------------|
| **Full Form** | Open Systems Interconnection | Transmission Control Protocol / Internet Protocol |
| **Purpose** | Conceptual reference model for network communication | Practical protocol suite used on the Internet |
| **Top Layer** | Application | Application |
| **Bottom Layer** | Physical | Network Access (Link) |
| **Session Layer** | Separate layer | Combined into the Application layer |
| **Presentation Layer** | Separate layer | Combined into the Application layer |
| **Application Layer** | Provides network services to applications | Combines Application, Presentation, and Session functions |
| **Transport Layer** | End-to-end communication, segmentation, reliability | Same purpose (TCP/UDP) |
| **Network Layer** | Logical addressing and routing | Internet layer handles IP routing |
| **Data Link Layer** | Framing, MAC addressing, error detection | Combined with Physical as Network Access layer |
| **Physical Layer** | Transmission of raw bits over media | Combined into Network Access layer |
| **Protocol Dependency** | Protocol-independent | Protocol-dependent (built around TCP/IP protocols) |
| **Usage** | Primarily used for learning and troubleshooting | Used in real-world networking and the Internet |
| **Flexibility** | More modular and easier to modify | Less modular but highly practical |
| **Examples of Protocols** | No specific protocols defined | HTTP, HTTPS, FTP, SSH, TCP, UDP, IP, ICMP, ARP |

System administrators primarily work at the Internet and Transport layers when configuring networks, and at the Application layer when managing services.

## The Linux Network Stack
Linux has a robust, modular network stack built directly into the kernel. All network traffic passes through the kernel, which handles packet routing, filtering, and delivery to user-space applications via the socket interface.

### Key Components:

| Component | Description |
|-----------|-------------|
| **Network Interface Cards (NICs)** | Physical or virtual hardware that connects the system to a network. |
| **Device Drivers** | Kernel modules that manage NIC hardware. |
| **Protocol Stack** | Implements networking protocols such as TCP/IP, UDP, ICMP, and others. |
| **Socket Interface** | The API through which applications communicate over the network using system calls such as `socket()`, `bind()`, and `connect()`. |

## IP Addressing Fundamentals
### IPv4 Addresses
IPv4 addresses are 32-bit numbers, typically written in dotted-decimal notation (e.g., `192.168.1.10`). Each address is divided into a network portion and a host portion using a subnet mask.

### IPv6 Addresses
IPv6 addresses are 128-bit, written in hexadecimal (e.g., 2001:db8::1). IPv6 is increasingly important as IPv4 address space has been exhausted.

### CIDR Notation
CIDR (Classless Inter-Domain Routing) combines an IP address with a subnet mask using a slash notation. For example, `192.168.1.0/24` means 24 bits are allocated to the network, leaving 8 bits for hosts (256 possible addresses). You'll learn more about these notations in future networking courses.

## Intra-Networking (Local Communication)
Intra-networking refers to communication within a single network, such as a Local Area Network (LAN). This operates primarily at Layer 2 (Data Link layer) of the OSI model.

### Ethernet and MAC Addresses
Every network interface has a unique 48-bit MAC (Media Access Control) address. MAC addresses are used for communication within the same network segment. Unlike IP addresses, MAC addresses are burned into the hardware and are not routable across the Internet.

### ARP (Address Resolution Protocol)
ARP resolves IP addresses to MAC addresses. When a system wants to communicate with another device on the same network, it broadcasts an ARP request: "Who has IP `192.168.1.1`? The device with that IP responds with its MAC address.

The command below can be used to find the MAC address:

```bash
ip link show
```

## Inter-Networking (Routing and Internetworking)
Inter-networking refers to communication between different networks. This operates at Layer 3 (Network layer) and is the basis of the Internet.

### IP Routing
Routing is the process of forwarding packets from one network to another. Each system maintains a routing table that determines where to send packets based on their destination IP address.

### Default Gateway
The default gateway is the router that handles traffic destined for networks outside the local subnet. It is typically the first hop for outbound traffic.

### Routing Protocols (Brief Overview)
Static Routing: Routes are manually configured by the administrator

Dynamic Routing: Protocols like RIP, OSPF, and BGP automatically learn and share routes between routers

For system administrators, understanding the routing table and default gateway is essential.

```bash
ip a
```

The above command shows the IP address along with other information like MAC address, loopback address and etc. Same results can be achieved using the following command as well:

```bash
ip addr
```

## Network Manager
NetworkManager is the modern tool for managing both wired and wireless connections. It provides GUI, CLI, and API interfaces.

Key CLI tool: nmcli (Network Manager Command Line Interface)

```bash
# Show all connections
nmcli connection show

# Show active connections
nmcli device status
```

## The `ip` Command: Modern Network Management
The `ip` command is part of the `iproute2` suite and is the modern replacement for `ifconfig` and `route`. It is the standard tool for network configuration on Linux.

| Task                     | Command                                      |
|--------------------------|----------------------------------------------|
| Show all interfaces      | `ip addr show` (or ip a)                       |
| Show routing table       | `ip route show` (or ip r)                      |
| Bring interface up       | `ip link set eth0 up`                          |
| Bring interface down     | `ip link set eth0 down`                        |
| Add an IP address        | `ip addr add 192.168.1.10/24 dev eth0`        |
| Remove an IP address     | `ip addr del 192.168.1.10/24 dev eth0`         |
| Add a default gateway    | `ip route add default via 192.168.1.1`         |

While `iproute2` is the modern standard, you will encounter legacy tools in many production environments. It is valuable to know both.

## DNS (Domain Name System)
DNS translates human-readable domain names (e.g., example.com) to IP addresses.

| File               | Purpose                                              |
|--------------------|------------------------------------------------------|
| `/etc/resolv.conf`   | Specifies DNS servers to use                        |
| `/etc/hosts`         | Local hostname-to-IP mappings (overrides DNS)       |

Below are some of the common DNS tools:

Tool	Purpose
dig	Detailed DNS query tool
nslookup	Simpler DNS query tool
host	Quick hostname lookup

Let's see some of the examples:

```bash
# Query a domain
dig example.com
```
The above command asks the operating system to use the default DNS to resolve `example.com`, which means an IP address for this URL is searched in the default DNS.

Sometimes, a DNS is also provided instead of using the default DNS. For example, in the below command Google's DNS (`8.8.8.8`) is provided.
```bash
# Query a specific DNS server
dig @8.8.8.8 example.com
```
Sometimes comparing the results from above two commands can help in troubleshooting, e.g., if `dig example.com` fails but `dig @8.8.8.8 example.com` works; it means the internet is fine but local network's DNS configuration is broken.

A Reverse Lookup is also possible, which means an IP address is provided and domain name is requested.

```bash
# Reverse lookup
dig -x 8.8.8.8
```

## Network Troubleshooting
Systematic troubleshooting saves time. Follow a layered approach: start at the physical layer and work your way up.

### Common Troubleshooting Workflow
1. Check physical connectivity: `ip link`
2. Check IP configuration: `ip addr`
3. Test local gateway: `ping <gateway_ip>`
> gateway ip can be found using `ip route`
4. Test external connectivity: `ping 8.8.8.8`
5. Check DNS resolution: `dig example.com`

#### Essential Tools
| Tool        | Purpose                                                     |
|------------|-------------------------------------------------------------|
| ping        | Test basic connectivity (ICMP echo)                         |
| traceroute  | Show the path packets take to a destination                 |
| ss          | Show network connections, listening ports (replaces netstat)|
| tcpdump     | Capture and analyze network packets                         |
| nmcli       | Manage NetworkManager connections                           |


#### Examples:

```bash
# Test connectivity
ping -c 4 8.8.8.8
```
In the above command `-c 4` means ping is done 4 times.

It is a good idea to know which of the ports are open or closed.
```bash
# Show all listening ports
ss -tuln
```

A deeper dive into packets is essential at times to troubleshoot security issues. `tcpdump` is a handy utility for this purpose. 
```bash
# Capture packets on an interface
sudo tcpdump -i eth0 -n
# eth0 is the NIC. You might have a different NIC in your system. Check with ip a.
```
Once you run this, you will see a continuous stream of text scrolling by in your terminal. Each line represents a single packet of data.

## Network Services Overview
System administrators manage many network services. Here is a brief overview of common ones:

| Service | Purpose                          | Daemon              |
|--------|----------------------------------|---------------------|
| SSH    | Secure remote access             | sshd                |
| DHCP   | Automatic IP address assignment  | dhcpd              |
| DNS    | Name resolution                 | named / dnsmasq    |
| NTP    | Time synchronization             | ntpd / chronyd     |
| NFS    | Network file sharing            | nfsd               |
| Samba  | Windows file/print sharing      | smbd               |

### SSH (Secure remote access)
SSH (Secure Shell) is one of the most important tools in a system administrator's toolkit. It is a cryptographic network protocol that provides secure remote login, command execution, and file transfer over an encrypted channel. 

| Component        | Purpose                                                                 |
|----------------|-------------------------------------------------------------------------|
| SSH Client (ssh) | The tool you run on your local machine to initiate a connection to a remote system |
| SSH Server (sshd)| The daemon running on the remote machine that listens for incoming SSH connections |

When you connect via SSH, the server and client negotiate encryption parameters, authenticate the user, and then establish a secure, encrypted session.

```bash
ssh username@ip_address
```

The above command will help in remoting into another system using the username provided. It is necessary to have that username enabled in the remoting system for the above command to work.

## Exercise
Write a Bash script called `host_discovery.sh` that discovers all active hosts on a local subnet and generates a report. The script should do the following: 
- Define a subnet variable (e.g., `SUBNET="192.168.1"`)
- Use a for loop to iterate through IP addresses from 1 to 254. 
- For each IP, use the `fping` command with a single packet and a timeout of `1000ms` to check if the host is alive. 
- If the host responds, print its IP address with the label "Alive" in a formatted table. 
- The script should save the report to a file named `host_report_YYYYMMDD.txt` (where YYYYMMDD is the current date) using tee so that output appears both on the screen and in the file. 

### Solution
```bash
#!/bin/bash

SUBNET="10.0.2"
REPORT_FILE="host_report_$(date +%Y%m%d).txt"

echo "Scanning ${SUBNET}.0/24 with fping..."
{
	echo "----------------------------------------"
	echo "Host Discovery Report - $(date)"
	echo "Subnet: ${SUBNET}.0/24"
	echo "----------------------------------------"
	
	for i in {1..254}; do
		ip="${SUBNET}.${i}"
		if fping -c 1 -t 1000 "$ip" &>/dev/null; then
			printf "%-15s %s\n" "$ip" "Alive"
		fi
	done
	
	echo "----------------------------------------"
} | tee "$REPORT_FILE"

echo "Report saved to $REPORT_FILE"

```

## Ungraded Exercises

### Exercise 1: 
Perform the following activities:
- Use the `ip` command to explore your system's network configuration.
- List all network interfaces on your system.
- Identify the IP address, and MAC address of each interface.
- View the routing table and identify the default gateway.


## Exercise 2:
Write a Bash script `port_discovery.sh` that scans all the ports on your system and lists only that are open. Your script should do the following:
- Scan all the ports using `ss`
- Filter only the ports that are open, e.g., `grep LISTEN`
- Only show the protocol, address and port in the output.

> Hint: use `while read -r proto state recvq sendq local peer process; do` to read all the variables from `ss` into different variables and then echo only `proto` and `local`.

## Exercise 3:
Write a Bash script `network_monitor.sh` that monitors the connectivity of a server with the IP address `8.8.8.8` by performing four ping attempts. If the server is reachable, display a message with the current time indicating that the server is reachable. If the server is unreachable, append a failure report to a text file named `server_report_YYYYMMDD.txt`, where `YYYYMMDD` is the current date. Each failure report should include a separator line, the current date and time, and a timestamped message indicating that the server is not reachable. Feel free to play around with this exercise by adding variations to it as follows:
- Take server IP address input from console
- Keep pinging for 100 times
- Log not only the failure but the successful attempts as well
- Log count of failures and successes as well. When failure reach 20 times, stop the script.




## References
- [The Linux Networking Overview](https://tldp.org/HOWTO/Networking-Overview-HOWTO.html)
- [Linux Networking - Kernel Docs](https://docs.kernel.org/networking/index.html)
- [Redhat Networking Basics](https://www.redhat.com/en/blog/sysadmin-essentials-networking-basics)
- Manual pages: `ip`, `ss`, `ping`, `dig`, `tcpdump`, `nmcli`