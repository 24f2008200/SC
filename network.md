## Network Diagnostic Commands — Full Guide

---

## 1. CONNECTIVITY TESTING

### `ping -c 4 google.com`
- Tests if a host is **reachable**
- `-c 4` → send exactly **4 packets** then stop
- Without `-c` it pings forever

| Variation | Meaning |
|---|---|
| `ping -c 4 google.com` | Ping by domain |
| `ping -c 4 8.8.8.8` | Ping by IP (Google DNS) — tests without DNS |
| `ping -i 0.5` | Ping every 0.5 seconds |
| `ping -s 1000` | Send larger packet size |

> If `ping google.com` fails but `ping 8.8.8.8` works → **DNS problem**, not network

---

### `traceroute google.com`
- Shows **every hop** (router) between you and destination
- Reveals **where packets are getting stuck**

```
1  192.168.1.1    (your router)
2  10.0.0.1       (ISP)
3  ...
N  google.com
```

---

### `curl google.com`
- Tests **HTTP connectivity** (layer 7, not just ping)
- Ping can succeed but HTTP can fail (firewall blocking port 80)

| Variation | Meaning |
|---|---|
| `curl google.com` | HTTP GET |
| `curl -I google.com` | Headers only |
| `curl -v google.com` | Verbose (debug) |
| `curl https://google.com` | Test HTTPS |

---

## 2. DNS RESOLUTION

### `dig google.com`
- Detailed DNS lookup — shows **full DNS response**
- More powerful than nslookup

```bash
dig google.com          # A record (IPv4)
dig google.com MX       # Mail records
dig google.com +short   # Just the IP
dig @8.8.8.8 google.com # Query specific DNS server
```

---

### `nslookup google.com`
- Simpler DNS lookup tool
- Good for quick checks

```bash
nslookup google.com           # basic lookup
nslookup google.com 8.8.8.8   # use specific DNS server
```

---

### `host google.com`
- Simplest DNS lookup — one line output
```bash
host google.com        # → google.com has address 142.250.x.x
host 142.250.x.x       # reverse lookup → IP to domain
```

---

### `whois google.com`
- Shows **domain registration info** — owner, registrar, expiry
- Not for network troubleshooting — more for domain research

---

## 3. NETWORK INTERFACES

### `ip a`
- Modern command — shows **all network interfaces and IPs**
- Replacement for `ifconfig`

```bash
ip a                  # all interfaces
ip a show eth0        # specific interface
ip r                  # routing table
ip link               # link status only
```

---

### `ifconfig -a`
- Older command — same purpose as `ip a`
- `-a` shows **all** interfaces including inactive ones
- Deprecated on modern Linux but still widely used

---

### `hostname -I`
- Quickly prints **just your IP address(es)**
- Simpler than parsing `ip a` output

---

### `nmcli dev show`
- NetworkManager CLI — shows **detailed interface info**
- Includes DNS, gateway, connection name

```bash
nmcli dev show          # all devices
nmcli dev show eth0     # specific device
nmcli con show          # all connections
```

---

## 4. PORT & SOCKET CHECKING

### `netstat -tuln | grep 8080`
| Flag | Meaning |
|---|---|
| `-t` | TCP connections |
| `-u` | UDP connections |
| `-l` | **Listening** ports only |
| `-n` | Show numbers (not names) |

```bash
netstat -tuln           # all listening ports
netstat -tuln | grep 8080   # is 8080 open?
netstat -an | grep 80   # all connections on port 80
```

---

### `ss -ltnp | grep 8080`
- **Modern replacement** for `netstat` (faster)
- `-p` additionally shows the **process name** using the port

```bash
ss -ltnp              # all listening TCP with process
ss -ltnp | grep 8080  # who is using 8080?
ss -s                 # socket summary
```

---

### `lsof -i :8080`
- Lists **which process** has port 8080 open
- More detailed than ss/netstat

```bash
lsof -i :8080         # who owns port 8080?
lsof -i TCP           # all TCP connections
lsof -i -n -P        # all with numeric ports
```

---

## 5. FIREWALL

### `ufw status`
- **Ubuntu's simple firewall** status
```bash
ufw status            # basic status
ufw status verbose    # detailed rules
ufw allow 8080        # open port
ufw deny 8080         # block port
```

---

### `iptables -L`
- Low-level **Linux firewall rules**
- More powerful but complex

```bash
iptables -L           # list all rules
iptables -L -n -v     # with packet counts and numeric
iptables-save         # export all rules to stdout
```

---

### `firewall-cmd --list-all`
- **RHEL/CentOS** firewall (firewalld)
```bash
firewall-cmd --list-all           # all rules
firewall-cmd --add-port=8080/tcp  # open port
firewall-cmd --permanent ...      # persist after reboot
```

---

## Quick Troubleshooting Guide:

| Problem | Commands to Use |
|---|---|
| Can I reach the network? | `ping 8.8.8.8` |
| Is DNS working? | `dig google.com`, `nslookup` |
| Where is the packet dropping? | `traceroute` |
| What's my IP? | `ip a`, `hostname -I` |
| Is port 8080 open? | `ss -ltnp`, `netstat -tuln`, `lsof -i` |
| Is firewall blocking? | `ufw status`, `iptables -L` |
| Is HTTP working? | `curl google.com` |
