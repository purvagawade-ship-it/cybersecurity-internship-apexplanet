# Linux Cheat Sheet
**Task 1 Deliverable | ApexPlanet Cybersecurity Internship**

Quick reference of Linux commands used during Task 1, grouped by category.

---

## Navigation

| Command | Description |
|---|---|
| `pwd` | Show current working directory |
| `ls` | List files/folders in current directory |
| `ls -l` | Detailed list (permissions, owner, size, date) |
| `ls -a` | Show hidden files (starting with `.`) |
| `cd <folder>` | Move into a folder |
| `cd ..` | Move up one directory level |
| `cd /` | Jump to root directory |
| `cd ~` | Jump to home directory |

## Permissions & Ownership

| Command | Description |
|---|---|
| `touch <file>` | Create an empty file |
| `chmod 755 <file>` | Set permissions: owner=rwx, group=r-x, others=r-x |
| `chmod +x <file>` | Add execute permission |
| `chown user:group <file>` | Change file owner and group |
| `ls -l <file>` | View current permissions/owner |

**Permission digits:** 4=read, 2=write, 1=execute (add them up per position: owner/group/others)

## Package Management

| Command | Description |
|---|---|
| `sudo apt update` | Refresh list of available packages |
| `sudo apt upgrade` | Upgrade installed packages to latest version |
| `sudo apt install <pkg> -y` | Install a package without confirmation prompt |
| `sudo apt remove <pkg>` | Remove a package |
| `apt list --installed` | List all installed packages |
| `dpkg -l` | List packages via low-level Debian tool |
| `dpkg -l \| grep <pkg>` | Check if a specific package is installed |

## Networking

| Command | Description |
|---|---|
| `ifconfig` | Show network interfaces and IP addresses |
| `ip addr show` | Show IP address and subnet (CIDR notation) |
| `ip route` | Show routing table / default gateway |
| `ping -c 4 <host>` | Test connectivity, send 4 packets |
| `netstat -tulnp` | Show active listening ports and processes |
| `ss -t -a` | Show TCP connections and their state |
| `traceroute <host>` | Show path (hops) packets take to a destination |
| `nslookup <domain>` | Resolve a domain name to an IP address |
| `dig <domain>` | Detailed DNS lookup (records, TTL) |
| `curl -I <url>` | Fetch only HTTP headers from a URL |
| `sudo tcpdump -c 5 -i eth0` | Capture 5 packets on interface eth0 |

## Cryptography (OpenSSL)

| Command | Description |
|---|---|
| `openssl enc -aes-256-cbc -salt -pbkdf2 -in <file> -out <file>.enc -k <pass>` | Encrypt a file with AES-256 |
| `openssl enc -d -aes-256-cbc -pbkdf2 -in <file>.enc -out <file> -k <pass>` | Decrypt an AES-256 encrypted file |
| `md5sum <file>` | Generate MD5 hash of a file |
| `sha256sum <file>` | Generate SHA256 hash of a file |
| `openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365 -nodes` | Generate self-signed SSL certificate + private key |
| `openssl x509 -in cert.pem -text -noout` | View certificate details |

## Recon & Scanning Tools

| Command | Description |
|---|---|
| `nmap <target>` | Basic scan - discover open ports |
| `nmap -sV <target>` | Detect service versions on open ports |
| `nmap -sS <target>` | TCP SYN (stealth) scan |
| `nmap -sU <target>` | UDP scan |
| `nmap -O <target>` | OS detection |
| `nc -v <target> <port>` | Connect to a port and grab service banner |
| `sudo wireshark` | Launch GUI packet capture tool |
| `burpsuite` | Launch web proxy/interception tool |

---

**Tip:** Replace `<target>`, `<file>`, `<pass>`, `<domain>` etc. with real values relevant to your lab (e.g. Metasploitable2's IP).