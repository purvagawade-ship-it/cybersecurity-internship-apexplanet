# Task 1 – Foundation and Environment Setup
**ApexPlanet Cybersecurity & Ethical Hacking Internship | Days 1–12**

---

## Step 1: Cybersecurity Basics

**CIA Triad:**
- **Confidentiality** – only authorized people can access data
- **Integrity** – data is accurate and unaltered
- **Availability** – systems/data are accessible when needed

**Threat Types explored:** Phishing, Malware, DDoS, SQL Injection, Brute Force, Ransomware

**Attack Vectors explored:** Social Engineering, Wireless Attacks, Insider Threats

---

## Step 2: Lab Environment Setup

**Tools installed:**
- VirtualBox (hypervisor)
- Kali Linux (attacker machine)
- Metasploitable2 (intentionally vulnerable target machine)

**Network configuration:**
- Configured **Host-Only Adapter** for both VMs

**Why Host-Only networking?**
Host-Only mode creates an isolated virtual network between the host PC and VMs only — no traffic can reach or come from the real internet. This lets me safely practice scanning and exploitation on Metasploitable2 without any risk of affecting outside systems or exposing vulnerable services to the internet.

📸 *Screenshots: VirtualBox showing both VMs running, Host-Only Network Manager settings.*

---

## Step 3: Linux Fundamentals

### File System Navigation
```bash
pwd                # show current directory - confirms where I am in the filesystem
ls                  # list files/folders in current directory
ls -l               # detailed list - shows permissions, owner, size, date
ls -a               # shows hidden files too (files starting with .)
cd Desktop          # move into Desktop folder
cd ..               # go up one directory level
cd /                # jump to root directory
cd ~                # jump back to home directory
```

### File & Directory Permissions
```bash
touch testfile.txt          # create an empty test file to practice on
ls -l testfile.txt          # check default permissions before changing anything
chmod 755 testfile.txt      # set permissions: owner=rwx, group=r-x, others=r-x
ls -l testfile.txt          # verify permissions changed correctly
chown root:root testfile.txt  # change file owner/group to root - controls who owns the file
ls -l testfile.txt          # verify ownership change
```

### Package Management
```bash
sudo apt update                    # refresh list of available packages from repositories
sudo apt list --installed | head   # check what's already installed on the system
sudo apt install curl -y           # install a tool (curl) - practicing package installation
dpkg -l | grep curl                # verify install using low-level package tool (dpkg)
```

### Networking Commands
```bash
ifconfig               # show network interfaces and IP addresses assigned to this machine
ping -c 4 8.8.8.8       # test connectivity/reachability to an external host (4 packets only)
netstat -tulnp          # show active listening ports and services - useful for spotting open services
traceroute google.com   # show the path (hops/routers) packets take to reach a destination
```

📸 *Screenshots: navigation commands run together, permissions before/after chmod & chown, apt/dpkg output, ifconfig + ping + netstat + traceroute output.*

---

## Step 4: Networking Basics

### OSI Model (7 Layers)
| Layer | Name | Function | Example |
|---|---|---|---|
| 7 | Application | User-facing services | HTTP, FTP, DNS |
| 6 | Presentation | Data format/encryption | SSL/TLS, JPEG |
| 5 | Session | Manages connections | Login sessions |
| 4 | Transport | End-to-end delivery | TCP, UDP |
| 3 | Network | Routing, addressing | IP, routers |
| 2 | Data Link | Node-to-node delivery | MAC address, switches |
| 1 | Physical | Raw bits over medium | Cables, Wi-Fi signals |

### TCP/IP Protocol Suite
```bash
ss -t -a                    # show all TCP connections and their state (LISTEN, ESTABLISHED etc.)
sudo tcpdump -c 5 -i eth0   # capture 5 live packets on eth0 - see TCP/IP in action at packet level
```

### DNS & HTTP/HTTPS
```bash
nslookup google.com         # resolve domain name to IP address - shows DNS in action
dig google.com              # more detailed DNS lookup - shows record type, TTL etc.
curl -I http://example.com  # fetch only headers over HTTP - shows unencrypted response
curl -I https://example.com # fetch only headers over HTTPS - shows encrypted (TLS) response
```

### IP Addressing, Subnetting, NAT
```bash
ip addr show   # show assigned IP address and subnet mask (CIDR notation) for this machine
ip route       # show default gateway - the point where NAT translates private IP to public IP
```

**Subnetting note:** `/24` = `255.255.255.0` = 254 usable host addresses.
**NAT note:** Translates private IPs (e.g. 192.168.1.x) to a public IP so multiple devices share one internet connection — performed at the default gateway.

📸 *Screenshots: ss/tcpdump output, nslookup+dig output, curl -I (HTTP vs HTTPS), ip addr + ip route.*

---

## Step 5: Cryptography Basics

### Symmetric Encryption (AES) with OpenSSL
```bash
echo "This is a secret message for my internship task" > secret.txt
# create a plain text file to encrypt

openssl enc -aes-256-cbc -salt -pbkdf2 -in secret.txt -out secret.enc -k mypassword123
# encrypt the file using AES-256 symmetric encryption with a password-derived key

cat secret.enc
# view encrypted file - confirms it's unreadable/garbled, proving encryption worked

openssl enc -d -aes-256-cbc -pbkdf2 -in secret.enc -out decrypted.txt -k mypassword123
# decrypt the file back using the same password (symmetric = same key both ways)

cat decrypted.txt
# verify decrypted content matches the original message
```

### Hashing (MD5 & SHA256)
```bash
echo "This is a secret message for my internship task" > hashtest.txt
# create a file to generate hashes from

md5sum hashtest.txt      # generate MD5 hash - older/weaker algorithm, shown for comparison
sha256sum hashtest.txt   # generate SHA256 hash - stronger, industry-standard integrity check
```
**Note:** Hashing is one-way (cannot be reversed); used for integrity checks, not confidentiality. SHA256 is more secure than MD5.

### Digital Certificates & SSL/TLS
```bash
openssl req -x509 -newkey rsa:2048 -keyout mykey.pem -out mycert.pem -days 365 -nodes -subj "/C=IN/ST=State/L=City/O=ApexPlanet/OU=Intern/CN=localhost"
# generate a self-signed SSL certificate + private key - simulates how HTTPS trust is set up

openssl x509 -in mycert.pem -text -noout
# view certificate details (issuer, validity, public key) to understand certificate structure
```

📸 *Screenshots: original vs encrypted vs decrypted file content, md5sum/sha256sum output, certificate generation + text details.*

---

## Step 6: Tool Familiarization

**Role of Metasploitable2:** An intentionally vulnerable Linux VM used as the "target" machine so tools below have something safe and legal to scan/capture/attack. Reachable only via the isolated Host-Only network (from Step 2).

Find target IP (run on Metasploitable2, login `msfadmin`/`msfadmin`):
```bash
ifconfig   # get Metasploitable2's IP address on the Host-Only network - needed for all scans below
```

### Nmap (on Kali, replace `<TARGET_IP>`)
```bash
nmap <TARGET_IP>        # basic scan - discover open ports/services on the target
nmap -sV <TARGET_IP>    # service version detection - identifies exact software versions (helps find known exploits)
```

### Wireshark (on Kali)
```bash
sudo wireshark   # launch packet capture tool to inspect raw network traffic
```
- Capture on Host-Only interface, filter: `ftp`  *(filters traffic to only show FTP packets)*
- In another terminal: `ftp <TARGET_IP>` *(generates FTP traffic to capture — login msfadmin/msfadmin, then `bye`)*
- Right-click packet → Follow → TCP Stream *(reassembles the conversation to reveal credentials sent in plain text)*

### Burp Suite (on Kali)
```bash
burpsuite   # launch web proxy tool to intercept and inspect HTTP requests/responses
```
- Proxy listener `127.0.0.1:8080` *(local port Burp listens on)*
- Firefox proxy set to match *(routes browser traffic through Burp so it can be intercepted)*
- Browse `http://<TARGET_IP>/` with Intercept ON *(pauses the request so I can inspect/modify it)*

### Netcat (on Kali)
```bash
nc -v <TARGET_IP> 21   # connect to FTP port - grabs service banner to confirm exact version without Nmap
nc -v <TARGET_IP> 22   # connect to SSH port - same banner-grabbing technique for SSH service
```

📸 *Screenshots: Nmap scan results, Wireshark FTP stream, Burp intercepted request, Netcat banner grabs.*

---

## Deliverables Checklist
- [ ] Lab Setup Report (screenshots of Kali, Metasploitable2, Wireshark capture)
- [ ] GitHub Repo with notes & Linux cheat-sheet
- [ ] 5-min video walkthrough of lab setup
