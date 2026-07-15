# Week 8: Intranet Services and Security
System administration is fundamentally about providing reliable, secure services to users. In the first seven weeks, you learned how to install, configure, monitor, and network Linux systems. This week, we bring all that knowledge together to focus on a critical responsibility: securing the services that run on your network.

An intranet is a private network internal to an organization that hosts essential services such as web servers, email, file sharing, and authentication. While these services enable productivity, they also introduce attacks. A misconfigured web server, an open email relay, or a weakly protected SSH daemon can expose the entire organization to data breaches, ransomware, and unauthorized access.

## The Security Mindset Core Principles
Before touching any configuration file, you must adopt a security mindset. Security is not a product or a one-time task; it is a continuous process of risk management.

| Principle | Description |
|-----------|-------------|
| Least Privilege | Give every user, process, and service only the minimum permissions necessary to perform its function. A web server does not need root access; a backup user does not need shell access. |
| Defence in Depth | Do not rely on a single security measure. Combine firewalls, host-based access control, service configuration, encryption, and monitoring. If one layer fails, others still protect you. |
| Assume Breach | Design your systems as if an attacker is already inside. This mindset drives you to encrypt sensitive data, segment networks, and monitor for anomalous behaviour. |
| Minimize Attack Surface | Every running service, open port, and installed package is a potential entry point. Disable, uninstall, or firewall anything that is not strictly required. |
| Secure by Default | Many services ship with insecure defaults (e.g., open relays, weak ciphers, default credentials). Always review and harden configurations before exposing a service. |

“You can never make a system 100% secure unless you unplug the machine from all networks, turn it off and lock it in a safe”. This reminds us that security is about reducing risk, not eliminating it entirely.


## Common Intranet Services
To secure services, you must first know what is running. Below is a reference table of typical intranet services, their protocols, and default ports.

| Service | Protocol | Port(s) | Purpose |
|---------|----------|---------|---------|
| Web Server | HTTP / HTTPS | 80, 443 | Internal websites, web applications, and APIs |
| Email (SMTP) | SMTP (with TLS) | 25, 587 | Sending email between internal servers and to the internet |
| Email (IMAP/POP) | IMAP / POP3 | 143, 993, 110, 995 | Retrieving email from a mail server |
| DNS | DNS (UDP/TCP) | 53 | Internal name resolution (e.g., `intranet.company.local`) |
| File Sharing (NFS) | NFS (v3/v4) | 2049 | Sharing files between Linux/Unix systems |
| File Sharing (SMB) | SMB/CIFS | 445, 139 | Sharing files with Windows clients |
| Authentication (LDAP) | LDAP / LDAPS | 389, 636 | Centralised user and group directories |
| Authentication (Kerberos) | Kerberos | 88, 749 | Single sign-on and strong mutual authentication |
| Remote Access | SSH | 22 | Secure remote administration and file transfer (`scp`/`sftp`) |
| Database | MySQL / PostgreSQL | 3306, 5432 | Internal databases for applications |
| Time Synchronisation | NTP | 123 | Synchronising system clocks across the organisation |

## Layered Security Model
The layered security model ensures that a failure in one control does not compromise the entire system. For intranet services, we typically implement the following layers (from outermost to innermost):

```text
┌─────────────────────────────────────────────┐
│         Physical Security                   │  (Data centres, locked racks)
├─────────────────────────────────────────────┤
│         Network Security (Firewalls)        │  (iptables/nftables, network ACLs)
├─────────────────────────────────────────────┤
│      Host Security (TCP Wrappers, SELinux)  │  (Restrict which hosts/services)
├─────────────────────────────────────────────┤
│      Service Security (Configuration)       │  (Hardened config files)
├─────────────────────────────────────────────┤
│      Application Security (Code, Updates)   │  (Patching, secure coding)
├─────────────────────────────────────────────┤
│         Data Security (Encryption)          │  (TLS, disk encryption)
└─────────────────────────────────────────────┘
```


## TCP Wrappers: Host-Based Access Control
TCP Wrappers provide a simple, host-based access control mechanism for services. 

### Key files:

`/etc/hosts.allow`: List of hosts that are explicitly allowed.

`/etc/hosts.deny`: List of hosts that are explicitly denied.

Assume you have an Ubuntu server and want only computers from `192.168.1.x` network to SSH into it. You can edit `hosts` as below:
```bash
sudo nano /etc/hosts.allow
```

Add the following to this file:
```bash
sshd: 192.168.1.
```
This means you are allowing clients from IP addresses starting from `192.168.1`, e.g., `192.168.1.10` or `192.168.1.50` and etc.


## Linux Firewalls
Firewalls filter network traffic based on rules. In Linux, the kernel’s Netfilter framework provides packet filtering, and iptables (legacy) and nftables (modern) are the userspace utilities to configure it.


### `iptables`
`iptables` organises rules into tables (e.g., filter for packet filtering) and chains (e.g., INPUT, OUTPUT, FORWARD). Each chain has a default policy (e.g., ACCEPT or DROP).

```bash
# View current rules (with line numbers)
sudo iptables -L -v -n --line-numbers
```
In the above command, `-L` helps in listing all the rules in the selected chain. `-v` is for verbose ouput which makes commands show interface names and rule options etc. `-n` is for outputing IP addresses and port numbers in numeric format.

Below commands can be used for setting up firewall rules.
```bash
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT
```

Below are the examples:
```bash
# Allow loopback (essential for internal communication)
sudo iptables -A INPUT -i lo -j ACCEPT

# Allow established/related connections (stateful firewall)
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH (port 22) from internal subnet only
sudo iptables -A INPUT -p tcp --dport 22 -s 192.168.1.0/24 -j ACCEPT

# Allow HTTP/HTTPS for web server
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
```

Table below provides details for these commands and their parameters:

| Parameter | Example | Example Explanation |
|---|---|---|
| `iptables` | `sudo iptables -L` | Executes the iptables firewall management tool and lists firewall rules |
| `sudo` | `sudo iptables` | Runs the command with administrative/root privileges |
| `-A` | `iptables -A INPUT -j ACCEPT` | Adds a new rule to the specified chain |
| `INPUT` | `iptables -A INPUT ...` | Applies the rule to incoming network traffic |
| `-i` | `iptables -A INPUT -i lo -j ACCEPT` | Specifies the network interface where packets enter |
| `lo` | `-i lo` | Matches traffic from the loopback interface (`127.0.0.1`) |
| `-j` | `-j ACCEPT` | Specifies the action to take when a rule matches |
| `ACCEPT` | `-j ACCEPT` | Allows the matching packet through the firewall |
| `-m` | `iptables -A INPUT -m state ...` | Loads an iptables matching module |
| `state` | `-m state` | Enables connection state tracking |
| `ESTABLISHED` | `--state ESTABLISHED` | Allows packets belonging to an existing connection |
| `RELATED` | `--state RELATED` | Allows packets related to an existing connection |
| `-p` | `-p tcp` | Specifies the network protocol to match |
| `tcp` | `-p tcp` | Matches TCP traffic such as SSH, HTTP, and HTTPS |
| `-s` | `-s 192.168.1.0/24` | Specifies the source IP address or network range |
| `DROP` | `-j DROP` | Silently blocks and discards packets |



### `nftables`
`nftables` replaces iptables in newer distributions (RHEL 8+, Debian 10+, Ubuntu 20.04+). It provides a simpler syntax, better performance, and unified IPv4/IPv6 handling.

The command below shows the current version of `nftables`
```bash
sudo nft --version
```
In order to see the current firewall rules, below command can be used:

```bash
sudo nft list ruleset
```
`nft` uses the below hierarchy:
```text
Table
 └── Chain
      └── Rules

# Example:

table inet filter
 |
 └── chain input
       |
       ├── allow SSH
       ├── allow HTTP
       └── block everything else
```

To create a firewall table, below command can be used:
```bash
# Create a table (container for chains)
sudo nft add table inet filter
```

The table below shows the details of the parameters used in the above command:

| Parameter | Example | Meaning |
|-----------|---------|---------|
| `add` | `add table` | Creates a new object |
| `table` | `table inet filter` | Creates a firewall table |
| `inet` | `table inet` | Supports both IPv4 and IPv6 |
| `filter` | `filter` | Table name |

Below commands can be used to create input and forward chains:
```bash
# Create input and forward chains with base hooks
sudo nft add chain inet filter input { 
    type filter hook input priority 0 \; 
}

sudo nft add chain inet filter forward { 
    type filter hook forward priority 0\; 
}
```

Adding rules in `nft` are simple:
```bash
# Add rules
sudo nft add rule inet filter input iif lo accept
sudo nft add rule inet filter input ct state established,related accept
sudo nft add rule inet filter input tcp dport 22 accept
sudo nft add rule inet filter input drop   # default deny
```
# nftables Rule Parameters Reference

| Parameter | Example | Example Explanation |
|---|---|---|
| `add` | `nft add rule` | Adds a new rule to an nftables chain |
| `rule` | `nft add rule inet filter input ...` | Specifies that a firewall rule is being created |
| `inet` | `nft add rule inet filter input ...` | Defines the address family; supports both IPv4 and IPv6 |
| `filter` | `nft add rule inet filter input ...` | Name of the nftables table where the rule is added |
| `input` | `nft add rule inet filter input ...` | Chain that processes incoming network traffic |
| `iif` | `iif lo` | Matches packets arriving through a specific network interface |
| `lo` | `iif lo accept` | Loopback interface (`127.0.0.1`) used for internal communication |
| `accept` | `iif lo accept` | Allows packets that match the rule |
| `ct` | `ct state established,related` | Uses connection tracking to check connection status |
| `state` | `ct state established,related` | Matches packets based on connection state |
| `tcp` | `tcp dport 22 accept` | Matches TCP protocol traffic |
| `drop` | `input drop` | Blocks and silently discards packets |


# 6. SSH – Fundamentals and Hardening
SSH (Secure Shell) is the cornerstone of remote system administration. Since almost every administrative task relies on SSH, it is the most critical service to secure. We cover both the basics (so you understand what you are hardening) and the hardening measures.


## SSH Fundamentals
SSH is a cryptographic network protocol that provides secure remote login, command execution, and file transfer over an encrypted channel. It replaces insecure protocols like `telnet`, `rlogin`, and `ftp` which transmit data (including passwords) in plain text.


### Client-Server Model:

- Server daemon (`sshd`): runs on the remote machine, listens on port 22 (by default).
- Client (`ssh`): runs on your local workstation to initiate connections.

### Connecting to a remote server:
The SSH server package is called OpenSSH Server. Make sure to install it before connection to a remote server:
```bash
sudo apt update
sudo apt install openssh-server
```

Check for SSH service status:
```bash
sudo systemctl status ssh
```
If service is not running, make sure to start the service:
```bash
sudo systemctl start ssh
```
After rebooting, enabling SSH is necessary:

```bash
sudo systemctl enable ssh
```

Once the service is up and running, below syntax can be used to connect to a remote server.

```bash
ssh username@hostname_or_ip

# Example:
ssh john@192.168.1.100
```
After successful authentication, the prompt might look something like this:
```bash
john@ubuntu-server:~$
```

## Exercise
Write a Bash script that audits a Linux server’s open network ports.

Your script should:
- Define an approved port list containing:
    - SSH (22)
    - HTTP (80)
    - HTTPS (443)
- Use the `ss` command to scan currently open TCP ports.
- Compare the detected ports against the approved list.
- If an unauthorized port is found:
    - Display a warning message.
    - Block the port using iptables.
    - Record that a security violation occurred.
- At the end of the script:
    - Display a success message if all ports are approved.
    - Otherwise, display a failure message indicating unauthorized ports were blocked.

### Solution
```bash
#!/bin/bash

# Week 8 - Exercise

APPROVED_PORTS=(22 80 443)

echo "Scanning ports..."
VIOLATIONS=0

OPEN_PORTS=$(ss -4 -tuln | awk 'NR>1 {print $5}' | awk -F: '{print $NF}')

for PORT in $OPEN_PORTS; do
	if [[ ! " ${APPROVED_PORTS[*]} " =~ " {$PORT} " ]]; then
		echo "WARNING: Unauthorized port open: $PORT"
		echo "Blocking the port with iptables..."
		sudo iptables -A INPUT -p tcp --dport "$PORT" -j DROP
		VIOLATIONS=1
	fi
done

if [[ $VIOLATIONS -eq 0 ]]; then
	echo "OK: All open ports are approved."
else
	echo "Security checked failed. Unauthorized ports have been blocked."
fi
```
## Ungraded Exercises

### Exercise 1
Run two virtual machines in virtualbox. Either you run two different linux distributions (Ubuntu and Fedora) or simple Clone Ubuntu in Virtualbox and run both of them separately.

Once both the VMs are up and running, SSH from one VM to another and copy a file between them. 

### Exercise 2
You are the administrator of an Ubuntu server.

The server should:
- Allow all outgoing connections
- Allow incoming SSH connections
- Allow incoming web traffic (HTTP/HTTPS)
- Allow local loopback traffic
- Drop all other incoming traffic

You are required to use `nft` for completing above steps. Make sure to verify the rules are in place.

### Exercise 3
Write a Bash script called `security_audit.sh` that performs a basic Linux security audit.

Your script should:
- Create a `header_func()` function that displays:
    - Report title
    - Current date
    - Hostname
- Create a `system_information()` function that displays:
    - Kernel version
    - Current logged-in user
- Create a `check_users()` function that:
    - Checks `/etc/shadow` for users with empty passwords
    - Displays the usernames found or a message if none exist
- Create a `check_services()` function that:
    - Checks if `telnet`, `ftp`, or `rsh` services are running
    - Displays a warning if any risky service is detected
- Call all functions in the correct order to generate a complete security audit report.

Requirements: Use Bash functions, variables, conditionals, loops, and command substitution.

## References
- [Linux Security HOW TO](https://tldp.org/HOWTO/Security-HOWTO/)
- [IPtables](https://man.archlinux.org/man/iptables.8)
- [NFTables](https://wiki.nftables.org/wiki-nftables/index.php/Main_Page)
- man pages: `sshd_config`, `iptables`, `nft`, `ss`, `hosts.allow`, `hosts.deny`