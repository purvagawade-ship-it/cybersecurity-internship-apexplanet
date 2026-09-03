# Task 2 - Step 5: Firewall Basics with iptables

## Rules Demonstrated

| Rule | Command | Purpose |
|------|---------|---------|
| Allow SSH | `iptables -A INPUT -p tcp --dport 22 -j ACCEPT` | Explicitly permit admin access |
| Block Telnet | `iptables -A INPUT -p tcp --dport 23 -j DROP` | Block insecure, unencrypted protocol |
| Block port scan target | `iptables -A INPUT -p tcp --dport 8080 -j DROP` | Demonstrate hiding a port from Nmap |
| Rate limiting | `iptables -A INPUT -p tcp -m conntrack --ctstate NEW -m limit --limit 5/minute -j ACCEPT` + DROP fallback | Detect/block scan-like rapid connection attempts |

## Test Results
- Port 23 (Telnet): confirmed blocked via `nc` timeout test
- Port 8080: Nmap scan result changed from `open`/`closed` to `filtered` after rule applied — confirms firewall is hiding the port from reconnaissance
- Rate limiting: increased number of `filtered` results during fast Nmap scan, demonstrating scan-detection behavior

## Real-World Relevance
iptables (and its concepts) form the basis of Linux server hardening, used in cloud security groups, 
SOC monitoring, and compliance audits. Understanding chains (INPUT/OUTPUT/FORWARD) and actions 
(ACCEPT/DROP/REJECT) is foundational to network defense.