

## Step 1: Reconnaissance

### Theory
Reconnaissance is the first phase of any penetration test — gathering information about a target before interacting with it.
- **Passive Recon:** Collecting info without directly touching the target (public records, search engines). Undetectable by the target.
- **Active Recon:** Lightly interacting with the target (pings, connection attempts) to confirm it's live and gather service info. Can be detected/logged by the target.

### Passive Recon Commands
```bash
# Whois - domain registration info (registrar, name servers, expiry)
whois google.com

# Nslookup - resolves domain to IP, confirms DNS resolution
nslookup google.com
nslookup 8.8.8.8          # reverse lookup: IP to hostname
```

**Google Dorking** (search engine operators to find exposed info):
site:example.com filetype:pdf
intitle:"index of" "parent directory"

**Shodan** — search engine for internet-connected devices/services (shodan.io), reveals open ports/banners without scanning yourself.

### Active Recon Commands
```bash
# Ping sweep - discover live hosts on the network
nmap -sn 192.168.56.0/24

# Banner grabbing - identify service + version running on a port
nc -nv 192.168.56.102 21   # FTP
nc -nv 192.168.56.102 22   # SSH
nc -nv 192.168.56.102 80   # HTTP
```

---

## Step 2: Port & Service Scanning

### Theory
Port scanning identifies which network doors (ports) are open on a target and what services are listening. This tells an attacker (or defender) the target's attack surface.

### Commands
```bash
# TCP SYN scan (stealth scan) - checks which TCP ports are open
sudo nmap -sS 192.168.56.102

# UDP scan - checks UDP ports (DNS, SNMP, DHCP etc.)
sudo nmap -sU 192.168.56.102

# Service version detection - identifies exact software + version
sudo nmap -sV 192.168.56.102

# OS detection - guesses target's operating system
sudo nmap -O 192.168.56.102

# Combined full scan, saved to a report file
sudo nmap -sS -sV -O -p- 192.168.56.102 -oN "Nmap scan report.txt"
```

**Key finding:** Service version detection revealed `vsftpd 2.3.4` on port 21 — a version with a known backdoor vulnerability (relevant for Task 4 exploitation).

---

## Step 3: Vulnerability Scanning with OpenVAS/GVM

### Theory
Vulnerability scanning goes a step further than port scanning — it doesn't just find open ports, it actively checks each service against a database of known vulnerabilities (CVEs) and rates them by severity: **Critical, High, Medium, Low**.

GVM (Greenbone Vulnerability Manager) is the modern name for OpenVAS.

### Setup Commands
```bash
sudo apt install gvm -y          # install
sudo gvm-setup                   # one-time initialization, generates admin password
sudo gvm-check-setup             # verify installation
sudo gvm-start                   # start services (web UI at https://127.0.0.1:9392)
```

### Workflow (via Web UI)
1. **Configuration → Targets** → create target with Metasploitable2's IP
2. **Scans → Tasks** → create new task, select target + "Full and fast" scan config
3. Click ▶ to start scan (takes 15 min–1 hr)
4. **Reports** tab → view results once status = "Done"
5. Export report as PDF

**Key finding:** Scan confirmed the Critical-severity vsftpd 2.3.4 backdoor found manually in Step 2, plus additional High/Medium findings like unencrypted Telnet and weak SSH ciphers.

---

## Step 4: Packet Analysis with Wireshark

### Theory
Wireshark captures and inspects raw network traffic packet-by-packet. This reveals what's actually being sent over the wire — including data that should be encrypted but isn't (like plaintext credentials).

### Commands & Filters
```bash
sudo wireshark          # launch (needs sudo for packet capture permissions)
```

Wireshark display filters used:
http # isolate HTTP traffic
ftp # isolate FTP traffic
ftp.request.command == "USER" || ftp.request.command == "PASS" # extract FTP login credentials
dns # isolate DNS traffic
tcp.flags.syn == 1 && tcp.flags.ack == 0 # isolate SYN packets (flood detection


Traffic generation commands (run alongside capture):
```bash
curl http://192.168.56.102        # generate HTTP traffic
ftp 192.168.56.102                # generate FTP traffic (enter any creds)
nslookup google.com               # generate DNS traffic
```

### SYN Flood Simulation
```bash
sudo apt install hping3 -y
sudo hping3 -S -p 80 --flood 192.168.56.102
# -S = SYN packets, -p 80 = target port, --flood = send as fast as possible
```

**Key finding:** FTP transmits usernames and passwords in **plain text** — fully readable in Wireshark. This is why FTP is considered insecure and why protocols like SFTP/FTPS exist. The SYN flood filter showed hundreds of SYN packets in seconds — a clear visual signature of a DoS attempt.

---

## Step 5: Firewall Basics with iptables

### Theory
iptables is Linux's traditional firewall tool, controlling traffic through **chains**:
- **INPUT** — traffic coming into the machine
- **OUTPUT** — traffic leaving the machine
- **FORWARD** — traffic passing through (routing)

Rules are processed top to bottom; each rule can **ACCEPT** (allow), **DROP** (silently discard), or **REJECT** (discard + notify sender) matching traffic.

### Commands
```bash
# View current rules
sudo iptables -L -v -n

# Allow a specific port (e.g. SSH)
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Block a specific port (e.g. Telnet - insecure protocol)
sudo iptables -A INPUT -p tcp --dport 23 -j DROP

# Rate limiting - detect/block scan-like rapid connections
sudo iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m limit --limit 5/minute --limit-burst 5 -j ACCEPT
sudo iptables -A INPUT -p tcp -m conntrack --ctstate NEW -j DROP

# Reset all rules (cleanup)
sudo iptables -F
```

### Testing Blocked Ports
```bash
nc -nv -w 3 192.168.56.102 23     # times out = confirms DROP rule works
sudo nmap -sS -p 8080 192.168.56.102   # shows "filtered" = port hidden by firewall
```

**Key finding:** A DROP rule makes Nmap report a port as `filtered` instead of `open`/`closed` — this is the signature of a firewall silently blocking reconnaissance attempts, directly demonstrating how iptables defends against port scanning.

---

## Task 2 Summary — Key Takeaways

| Step | Tool | Core Skill |
|------|------|-----------|
| 1 | Whois, Nslookup, Nmap, Netcat | Gathering target info before/during contact |
| 2 | Nmap | Mapping open ports, services, versions, OS |
| 3 | OpenVAS/GVM | Automated vulnerability detection + severity rating |
| 4 | Wireshark, hping3 | Reading raw traffic, spotting plaintext creds, recognizing attack patterns |
| 5 | iptables | Building firewall rules to block/limit unwanted traffic |

**Overall concept:** Steps 1-2 = reconnaissance (finding what's there), Step 3 = assessment (finding what's vulnerable), Step 4 = analysis (seeing traffic in the raw), Step 5 = defense (blocking/mitigating). This mirrors the real-world pentest → hardening workflow.