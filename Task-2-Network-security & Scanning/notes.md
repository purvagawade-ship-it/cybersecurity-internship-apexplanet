# Task 2 - Step 1: Reconnaissance

## Passive Recon
- **Whois**: Queried domain registration info for google.com (registrar, name servers, expiry).
- **Nslookup**: Resolved google.com to IP; performed reverse lookup on 8.8.8.8.
- **Google Dorking**: Used site:, filetype:, intitle: operators to find exposed files/directories on authorized test targets.
- **Shodan**: Searched for publicly exposed devices/services using Shodan search engine.

## Active Recon
- **Ping Sweep**: `nmap -sn 192.168.56.0/24` — discovered live hosts on Host-Only lab network.
- **Banner Grabbing**: `nc -nv <target-ip> <port>` — identified running services and versions (e.g., vsFTPd 2.3.4 on port 21).

## Key Learning
Passive recon = gather info without touching the target (legal against any public domain).
Active recon = lightly interacts with target, only done in the isolated lab (Metasploitable2), never against real-world systems without authorization.