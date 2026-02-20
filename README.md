# DNS Security Deep-Dive: From MITM Spoofing to DNSSEC & Redundancy

[![Framework: Kathara](https://img.shields.io/badge/Framework-Kathara-orange)](https://www.kathara.org/)
[![Tool: BIND9](https://img.shields.io/badge/Service-BIND9-blue)](https://www.isc.org/bind/)
[![Tool: Scapy](https://img.shields.io/badge/Tool-Scapy-brightgreen)](https://scapy.net/)

This repository contains a comprehensive three-stage laboratory developed for the **Computer Networks Security** course. It provides a complete hands-on experience on DNS vulnerabilities, cryptographic defenses (DNSSEC), and enterprise-level infrastructure redundancy.

> **Note**: This README is designed to provide all the technical details contained in the official laboratory slides, making it a standalone guide for the entire project.

---

## 🔬 Milestone 1 & 2: Basic Lab & DNSSEC Defense
### Overview
In the initial stages, the lab focuses on the core vulnerability of the DNS protocol and its resolution through DNSSEC. The network includes a University LAN, an external resolver, and an attacker's "Evil" network.

#### Network Topology (Basic)
![Basic Topology](images/topology_basic.png)

### 1. DNS Spoofing & Phishing (Offensive)
- **The Threat**: Lack of origin authentication in standard DNS.
- **The Attack**: Router `r2` uses a Scapy-based script (`r2_attack.py`) to intercept DNS queries. It sends a forged response pointing to the `evil` server.
- **Phishing Mechanism**: The `evil` server hosts a cloned student portal. Once credentials are entered, the PHP backend logs them and performs a **Seamless Auto-POST** to the real university site to remain undetected.

### 2. DNSSEC Implementation (Defensive)
- **The Mitigation**: Establishing a **Chain of Trust** (Root -> .it -> .uniroma3.it).
- **Validation**: The resolver (`pc4`) is configured to verify signatures. When the attacker tries to inject a fake record, the signature verification fails, and the resolver returns a `SERVFAIL` to the client.

---

## 🏢 Milestone 3: Full Redundancy & Enterprise Setup
### Overview
The final stage evolves the network into a highly available enterprise infrastructure. It introduces backup DNS authorities and redundant physical paths.

#### Network Topology (Redundant)
![Redundant Topology](images/topology_redundancy.png)

### Key Features of the Redundant Setup:
- **Master/Slave DNS**: 
    - `dnsroot2` acts as a backup for the Root.
    - `dnsuni2` acts as a slave for the University domain.
- **Zone Transfers (AXFR)**: Implementation of automated synchronization between Master and Slave nodes using `NOTIFY` and `AXFR` protocols.
- **Backbone Failover**: A secondary backbone path (LAN G) and a third router (`r3`) ensure that the network remains functional even if a primary link or router fails.

---

## 🚀 Step-by-Step Execution Guide

### 1. Start the Lab
```bash
cd 03-full-redundancy
kathara lstart
2. Launch the Attack (Scenario 1 & 2)
Inside the r2 terminal:
code
Bash
python3 r2_attack.py
Browse to www.uniroma3.it from pc3 (localhost:3001).
Without DNSSEC: You will be redirected to the phishing page.
With DNSSEC: The browser will show a "Server Not Found" error, protecting the user.
3. Verify Redundancy (Scenario 3)
Check if the Slave DNS has successfully received the zone from the Master:
code
Bash
# On dnsuni2 terminal
ls /var/cache/bind/db.uniroma3.transfered
⚙️ Technical Deep-Dive
The Scapy Attack Logic
The script on r2 reverse-engineers the incoming DNS packet to forge a perfect replica with a malicious payload:
code
Python
# Create the response by stealing the real server's identity
ip_layer = IP(src=pkt[IP].dst, dst=pkt[IP].src)
udp_layer = UDP(sport=pkt[UDP].dport, dport=pkt[UDP].sport)
dns_layer = DNS(id=pkt[DNS].id, qr=1, aa=1, rdata=EVIL_IP)
Automated DNSSEC Signing
The lab uses a dynamic logic in .startup files to build the Chain of Trust automatically:
code
Bash
# dnsit.startup waits for the child (uniroma3) to be ready, then requests its DS record
while ! dig @110.0.0.10 uniroma3.it DNSKEY | grep -q "257 3 13"; do sleep 2; done
dig @110.0.0.10 uniroma3.it DNSKEY | dnssec-dsfromkey -f - uniroma3.it >> /etc/bind/db.it
🛠 Prerequisites & Tools
Kathara Framework: Network virtualization.
BIND9: Authoritative and Recursive DNS services.
Apache2 + PHP: Web hosting and credential harvesting.
Wireshark: Real-time traffic analysis (accessible at localhost:3000).
📝 Credits
Course: Computer Networks Security - Roma Tre University.
Original Slides: D. Villa.
Framework: Kathara (Computer Networks Research Group).
