#  Networking Commands Cheatsheet

> A comprehensive guide to essential networking commands for Linux, macOS, and Windows.  

---


## 1. Connectivity Testing

### `ping` — Test Host Reachability

Sends ICMP echo request packets to a target host and waits for replies. Used to verify if a host is reachable and measure round-trip time (latency).

```bash
# Linux / macOS / Windows
ping google.com
ping 192.168.1.1

# Linux/macOS: Send only 4 packets
ping -c 4 google.com

# Windows: Send only 4 packets
ping -n 4 google.com
```

**Key Output Fields:**
| Field | Meaning |
|-------|---------|
| `64 bytes` | Packet size received |
| `icmp_seq` | Sequence number of the packet |
| `TTL` | Time To Live (hops before discard) |
| `time=5.62ms` | Round-trip latency |

---

### `traceroute` (Linux/macOS) / `tracert` (Windows) — Trace Network Path

Displays the route (hops) that packets take to reach a destination. Useful for identifying where delays or failures occur in the network path.

```bash
# Linux / macOS
traceroute google.com
traceroute 8.8.8.8

# Windows
tracert google.com
tracert 8.8.8.8
```

**Output Explained:**
- Each line = one hop (router)
- Three time values = RTT of three probe packets
- `*` = timeout or no response from that hop

---

### `mtr` — My Traceroute (Linux)

Combines `ping` and `traceroute` into a single real-time tool. Continuously shows latency and packet loss per hop.

```bash
mtr google.com
mtr --report google.com    # Generate a report
```

---

## 2. Network Configuration

### `ip` — Modern Linux Network Tool

Replaces older commands like `ifconfig` and `route`. The Swiss Army knife for Linux networking.

```bash
# Show all network interfaces and IPs
ip addr show
ip a                    # shorthand

# Show routing table
ip route show
ip r                    # shorthand

# Show link (MAC) status
ip link show
ip l                    # shorthand

# Bring an interface up/down
ip link set eth0 up
ip link set eth0 down

# Add an IP address
ip addr add 192.168.1.50/24 dev eth0
```

---

### `ifconfig` — Legacy Interface Config (Linux/macOS)

Still widely used but deprecated in favor of `ip` on Linux.

```bash
# Show all interfaces
ifconfig

# Show specific interface
ifconfig eth0

# Enable/disable interface
ifconfig eth0 up
ifconfig eth0 down
```

---

### `ipconfig` — Windows Network Config

```cmd
:: Basic IP info
ipconfig

:: Detailed info (MAC, DHCP, DNS)
ipconfig /all

:: Release DHCP IP
ipconfig /release

:: Renew DHCP IP
ipconfig /renew

:: Clear DNS cache
ipconfig /flushdns

:: Display DNS cache
ipconfig /displaydns
```

---

### `hostname` — System Name

```bash
# Show hostname
hostname

# Show fully qualified domain name (FQDN)
hostname -f        # Linux/macOS

# Set hostname (Linux)
hostname new-hostname
```

---

## 3. DNS & Name Resolution

### `nslookup` — Query DNS (All Platforms)

```bash
# Resolve domain to IP
nslookup google.com

# Reverse lookup (IP to domain)
nslookup 8.8.8.8

# Query specific record type
nslookup -type=MX google.com
nslookup -type=NS google.com
```

---

### `dig` — DNS Information Groper (Linux/macOS)

More detailed and powerful than `nslookup`.

```bash
# Basic query
dig google.com

# Specific record type
dig google.com A
dig google.com MX
dig google.com NS
dig google.com TXT

# Short answer only
dig +short google.com

# Trace full DNS resolution path
dig +trace google.com

# Query specific DNS server
dig @8.8.8.8 google.com
```

---

### `host` — Simple DNS Lookup

```bash
host google.com
host -a google.com          # All records
host -t MX google.com       # Mail records
```

---

## 4. Active Connections & Ports

### `netstat` — Network Statistics (All Platforms)

```bash
# Show all active connections
netstat -a

# Show listening ports
netstat -l

# Show numeric addresses (faster)
netstat -n

# Show process IDs (Windows/Linux)
netstat -o                  # Windows
netstat -tulpn              # Linux (needs root)

# Show routing table
netstat -r

# Show interface statistics
netstat -i

# Combined: all TCP/UDP listening ports with PIDs
netstat -tulpn              # Linux
netstat -ano                # Windows
```

---

### `ss` — Socket Statistics (Linux)

Modern, faster replacement for `netstat`.

```bash
# All TCP connections
ss -t

# All UDP connections
ss -u

# Listening sockets
ss -l

# All TCP/UDP, numeric, all states
ss -tuna

# Show process using the socket
ss -tulnp

# Show memory usage
ss -m
```

---

### `lsof` — List Open Files (Linux/macOS)

Everything in Unix is a file — including network sockets.

```bash
# Show processes using network ports
lsof -i

# Specific port
lsof -i :80
lsof -i :22

# Specific protocol
lsof -i TCP
lsof -i UDP

# Show listening ports
lsof -i -P -n | grep LISTEN
```

---

## 5. Routing

### `route` — View/Manage Routing Table

```bash
# Linux: Show routing table
route -n

# Windows: Show routing table
route print

# Linux: Add static route
sudo route add -net 192.168.10.0/24 gw 192.168.1.1

# Windows: Add static route
route add 192.168.10.0 mask 255.255.255.0 192.168.1.1

# Delete route
route delete 192.168.10.0
```

---

### `ip route` — Modern Linux Routing

```bash
# Show routing table
ip route show

# Add route
ip route add 192.168.10.0/24 via 192.168.1.1

# Delete route
ip route del 192.168.10.0/24

# Default gateway
ip route add default via 192.168.1.1
```

---

### `arp` — Address Resolution Protocol

Maps IP addresses to MAC addresses on the local network.

```bash
# Show ARP cache
arp -a

# Linux: Show in table format
ip neigh show

# Delete ARP entry
arp -d 192.168.1.1
```

---

## 6. Packet Capture & Analysis

### `tcpdump` — Command-Line Packet Analyzer (Linux/macOS)

```bash
# Capture on specific interface
sudo tcpdump -i eth0

# Capture specific host
sudo tcpdump host 192.168.1.1

# Capture specific port
sudo tcpdump port 80
sudo tcpdump port 443

# Capture and save to file
sudo tcpdump -w capture.pcap

# Read from file
sudo tcpdump -r capture.pcap

# Capture HTTP traffic
sudo tcpdump -i eth0 port 80 -A

# Limit packet count
sudo tcpdump -c 100

# Filter by protocol
sudo tcpdump icmp
sudo tcpdump tcp
sudo tcpdump udp
```

---

### `tshark` — Terminal Wireshark

```bash
# Capture on interface
sudo tshark -i eth0

# Capture specific port
sudo tshark -i eth0 -f "port 80"

# Save to file
sudo tshark -i eth0 -w capture.pcap
```

---

### `wireshark` — GUI Packet Analyzer

```bash
# Launch Wireshark (Linux)
sudo wireshark &

# Open capture file
wireshark capture.pcap
```

---

## 7. Network Scanning & Security

### `nmap` — Network Mapper

Powerful tool for network discovery and security auditing.

```bash
# Basic host scan
nmap 192.168.1.1

# Scan entire subnet
nmap 192.168.1.0/24

# Scan specific ports
nmap -p 22,80,443 192.168.1.1

# Scan all ports
nmap -p- 192.168.1.1

# OS detection
nmap -O 192.168.1.1

# Service version detection
nmap -sV 192.168.1.1

# Aggressive scan (OS + version + scripts + traceroute)
nmap -A 192.168.1.1

# Stealth scan
nmap -sS 192.168.1.1

# Scan with verbose output
nmap -v 192.168.1.1
```

---

### `nc` / `netcat` — Swiss Army Knife of Networking

```bash
# Test if port is open
nc -zv google.com 80
nc -zv 192.168.1.1 22

# Scan range of ports
nc -zv 192.168.1.1 20-80

# Create a simple server (listen on port 1234)
nc -l 1234

# Connect to server
nc localhost 1234

# Transfer file (receiver)
nc -l 1234 > received_file.txt

# Transfer file (sender)
cat file.txt | nc localhost 1234

# UDP mode
nc -u -l 1234
```

---

## 8. File Transfer & Remote Access

### `ssh` — Secure Shell

```bash
# Connect to remote server
ssh user@192.168.1.1
ssh user@server.com

# Connect with specific port
ssh -p 2222 user@server.com

# Execute command remotely
ssh user@server.com "ls -la"

# Copy file via SCP
scp file.txt user@server.com:/home/user/
scp -r folder/ user@server.com:/home/user/

# Copy from remote to local
scp user@server.com:/home/user/file.txt ./

# SSH with key
ssh -i ~/.ssh/id_rsa user@server.com
```

---

### `curl` — Transfer Data from URLs

```bash
# Simple GET request
curl https://api.example.com

# Save output to file
curl -o file.html https://example.com

# Follow redirects
curl -L https://bit.ly/xxx

# Show headers only
curl -I https://example.com

# POST request with data
curl -X POST -d "name=John&age=30" https://api.example.com

# POST JSON data
curl -X POST -H "Content-Type: application/json"   -d '{"name":"John"}' https://api.example.com

# Include headers in output
curl -v https://example.com

# Download with resume support
curl -C - -O https://example.com/largefile.zip
```

---

### `wget` — Web Downloader (Linux)

```bash
# Download file
wget https://example.com/file.zip

# Download with custom name
wget -O myfile.zip https://example.com/file.zip

# Resume download
wget -c https://example.com/file.zip

# Download entire website
wget --mirror --convert-links --adjust-extension   --page-requisites --no-parent https://example.com

# Download in background
wget -b https://example.com/largefile.zip
```

---

### `ftp` / `sftp` — File Transfer Protocol

```bash
# FTP connection
ftp hostname
ftp 192.168.1.1

# SFTP (secure)
sftp user@hostname

# Common FTP commands:
#   ls      - list files
#   cd      - change directory
#   get     - download file
#   put     - upload file
#   mget    - download multiple files
#   mput    - upload multiple files
#   binary  - binary mode
#   ascii   - ASCII mode
#   quit    - exit
```

---

## 9. Advanced Utilities

### `ethtool` — Ethernet Interface Settings (Linux)

```bash
# Show interface info
sudo ethtool eth0

# Show link status
sudo ethtool eth0 | grep Speed

# Show statistics
sudo ethtool -S eth0

# Test interface
sudo ethtool -t eth0
```

---

### `iw` / `iwconfig` — Wireless Tools (Linux)

```bash
# Show wireless interfaces
iw dev

# Scan for networks
iw dev wlan0 scan

# Show connection info
iw dev wlan0 link

# Legacy tool
iwconfig
```

---

### `netsh` — Network Shell (Windows)

```cmd
:: Show interface config
netsh interface ip show config

:: Show WiFi profiles
netsh wlan show profiles

:: Show WiFi password
netsh wlan show profile name="WiFiName" key=clear

:: Reset network stack
netsh winsock reset
netsh int ip reset

:: Show firewall status
netsh advfirewall show currentprofile
```

---

### `telnet` — Test Port Connectivity

```bash
# Test if port is open
telnet google.com 80
telnet 192.168.1.1 22

# Exit telnet: Ctrl+] then type 'quit'
```

>  **Note:** Telnet is unencrypted. Use `ssh` or `nc` instead for secure connections.

---

##  Quick Reference: Linux vs Windows

| Task | Linux/macOS | Windows |
|------|-------------|---------|
| **Show IP config** | `ip addr` / `ifconfig` | `ipconfig` |
| **Show routes** | `ip route` / `route -n` | `route print` |
| **Ping** | `ping -c 4` | `ping -n 4` |
| **Trace route** | `traceroute` | `tracert` |
| **DNS lookup** | `dig` / `nslookup` | `nslookup` |
| **Active connections** | `ss` / `netstat` | `netstat` |
| **ARP table** | `ip neigh` / `arp -a` | `arp -a` |
| **Packet capture** | `tcpdump` | Wireshark GUI |
| **Network config** | `ip` / `nmcli` | `netsh` |
| **Port test** | `nc -zv` | `Test-NetConnection` |

---

##  Pro Tips

1. **Use `sudo`** for commands that need root access (packet capture, interface changes)
2. **Combine commands** with pipes: `netstat -tulpn | grep :80`
3. **Use `man`** for detailed help: `man ping`, `man nmap`
4. **Save outputs** for later analysis: `ping google.com > ping.log`
5. **Use `-h` or `--help`** for quick syntax reminders

---

##  Additional Resources

- [Wireshark](https://www.wireshark.org/) — GUI packet analyzer
- [Nmap Documentation](https://nmap.org/book/) — Official Nmap guide
- [TCPDUMP Guide](https://www.tcpdump.org/manpages/tcpdump.1.html) — Official man page
- [Linux ip command](https://man7.org/linux/man-pages/man8/ip.8.html) — Official man page

---

*Made with  for network engineers, sysadmins, and developers.*

