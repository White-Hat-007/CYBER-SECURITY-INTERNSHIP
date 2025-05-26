# 🚨 Reconnaissance & Vulnerability Assessment


This task involved performing reconnaissance and vulnerability assessment on a target Linux Virtual Machine (VM) using tools like **Nmap** and **Wireshark** to uncover active services, analyze traffic, and assess security risks.


Phase 1: Target Identification

The process began by identifying the internal IP address of the Linux VM using the `ifconfig` command. This provided the local IP configuration necessary to begin scanning within the local subnet. The detected IP address became the central focus for all scanning and analysis activities.


🛰Phase 2: Stealth Scan with Nmap

To detect active services and open ports, I used Nmap with a stealth SYN scan using the following command:

```bash
nmap -sS 10.0.2.15/24
```
-->
    Flag Explained:
      	-sS: Used to perform a TCP SYN scan, also known as a half-open scan. It is one of the most popular and stealthy scanning techniques in cyber security. 
             Faster and less likely to be logged by host-based firewalls compared to full connect scans.
             

Phase 3: Results Interpretation

### Network Scan Results

| Host IP     | Open Ports       | Identified Services                           |
|-------------|------------------|-----------------------------------------------|
| 10.0.2.2    | 135, 445, 5357   | msrpc, microsoft-ds (SMB), wsdapi (Web Services) |
| 10.0.2.3    | 53               | domain → Domain Name System (DNS)             |
| 10.0.2.15   | None (All closed)| —                                             |



Phase 4: Packet Capture with Wireshark

Simultaneously, I ran Wireshark to analyze live packet traffic for any anomalous or unencrypted communications. This provided insight into protocol behaviors, and validated the open services discovered by Nmap.

Service Overview:

  •	Port 135 (msrpc) – Used by Microsoft Remote Procedure Call services.
  
  •	Port 445 (microsoft-ds) – Commonly associated with SMB, used for file sharing; often exploited in lateral movement attacks.
  
  •	Port 5357 (wsdapi) – Web Services for Devices (WSDAPI); often seen in printer discovery and IoT environments. It can be a lesser-known attack surface if 
                         exposed.
                         
  •	Port 53 (domain) – DNS service; could be exploited for DNS cache poisoning or tunneling.
  
  
Host: 10.0.2.15 communicating with:

  •	37.221.196.71 on:
 
  ->
          	Port 443/tcp – HTTPS (TLSv1.2 observed)
           
  ->
          	Source ports from 10.0.2.15 include: 43690, 43696
          
  •	37.33.194.x on:

  
  ->
          	Protocol: OpenVPN – Suggesting VPN traffic (UDP likely, though not specified here)
          

### Observed Network Services

| Port  | Protocol   | Service           | Observed On     |
|-------|------------|-------------------|------------------|
| 135   | TCP        | MS RPC            | 10.0.2.2         |
| 445   | TCP        | Microsoft-DS      | 10.0.2.2         |
| 53    | TCP        | DNS               | 10.0.2.3         |
| 443   | TCP        | HTTPS (TLSv1.2)   | 37.221.196.71    |
| —     | UDP/TCP    | OpenVPN (P_DATA)  | 37.33.194.x      |



Phase 5: Research and Security Risks

Each open port was analyzed against known vulnerabilities and exposures:

[] Port 135/tcp – MS RPC (Microsoft Remote Procedure Call)

•	Service: msrpc

•	Purpose: Used by Microsoft systems for DCOM (Distributed Component Object Model) and various RPC services.

•	Risks:

->	Commonly targeted for remote code execution.

->	Used in Blaster worm and other RPC-based attacks.

->	Often open on internal networks, but should never be exposed to the internet.

•	Recommendation: Block at the perimeter firewall and monitor for unauthorized use.

[] Port 445/tcp – Microsoft-DS (Direct SMB over TCP)

•	Service: microsoft-ds

•	Purpose: File sharing, printer sharing, and other services over SMB.
    
•	Risks:

->	Vulnerable to EternalBlue (used in WannaCry and NotPetya ransomware).

->	SMB is a major lateral movement vector in networks.

->	Exposing this port externally is a critical security risk.

•	Recommendation: Disable SMB if not needed; otherwise, limit access to trusted IPs and keep systems patched.

[] Port 53/tcp – DNS (Domain Name System)

•	Service: domain

•	Purpose: Resolves domain names to IP addresses.

•	Risks:

->	Can be used for DNS tunneling (data exfiltration).

->	Vulnerable to cache poisoning, amplification attacks, or DoS if misconfigured.

->	If running your own DNS server, it must be hardened and monitored.

•	Recommendation: Restrict recursion, rate-limit queries, and use DNSSEC if available.

Outcomes & Learning

•	The scan revealed potential security risks due to exposed SMB and WSD services on the network.

•	Services running on non-standard or obscure ports (e.g., 5357) require closer scrutiny, especially in IoT-heavy environments.

•	The use of stealth scanning (SYN scan) and packet analysis (Wireshark) proved effective in identifying hosts and assessing vulnerabilities with minimal 
  intrusion.

Tools Used

•	Nmap (Network Mapper) – Port scanning and host discovery.

•	Wireshark – Packet-level network traffic analysis.

Additionally:

•	Manual Research – Services and its risks and mitigations, CVE lookups.

