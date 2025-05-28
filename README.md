# 🔍 Reconnaissance & Vulnerability Assessment Report

This task involved performing reconnaissance and vulnerability assessment on a target **Linux Virtual Machine (VM)** using tools like **Nmap** and **Wireshark** to uncover active services, analyze traffic, and assess potential security risks.

---

## 🧭 Phase 1: Target Identification

The process began by identifying the **internal IP address** of the Linux VM using:

```bash
ifconfig
```

This revealed the local IP configuration necessary to initiate subnet scanning. The detected IP (`10.0.2.15`) became the focal point for all subsequent analysis.

---

## 🛰️ Phase 2: Stealth Scan with Nmap

To detect active services and open ports, a **stealth SYN scan** was conducted using:

```bash
nmap -sS 10.0.2.15/24
```

### 🔍 Flag Breakdown:
- `-sS`: Performs a **TCP SYN scan** (also known as a *half-open scan*).  
  - Faster and stealthier than full-connect scans.  
  - Less likely to trigger alarms on host-based firewalls.

---

## 📊 Phase 3: Scan Results Interpretation

### 🧠 Network Scan Summary

| Host IP     | Open Ports       | Identified Services                             |
|-------------|------------------|--------------------------------------------------|
| 10.0.2.2    | 135, 445, 5357   | msrpc, microsoft-ds (SMB), wsdapi (Web Services) |
| 10.0.2.3    | 53               | domain → Domain Name System (DNS)               |
| 10.0.2.15   | None (All closed)| —                                                |

---

## 🌐 Phase 4: Packet Capture with Wireshark

**Wireshark** was used in parallel to observe **live network traffic**. This provided a deeper understanding of communication patterns and confirmed services discovered via Nmap.

### 🔎 Service Analysis

- **Port 135 (msrpc):** Microsoft RPC services.  
- **Port 445 (microsoft-ds):** SMB (used for file/printer sharing); often exploited.  
- **Port 5357 (wsdapi):** Web Services for Devices (e.g., printers, IoT) – a lesser-known attack vector.  
- **Port 53 (domain):** DNS – may be vulnerable to cache poisoning or tunneling.  

### 🌐 External Communication Observed

#### From Host `10.0.2.15`:

- **To:** `37.221.196.71`  
  - **Port:** `443/tcp` (HTTPS using TLSv1.2)  
  - **Source Ports:** `43690`, `43696`  

- **To:** `37.33.194.x`  
  - **Protocol:** **OpenVPN** (likely UDP) – suggests encrypted VPN traffic  

---

## 🧾 Observed Network Services

| Port  | Protocol | Service         | Observed On      |
|-------|----------|------------------|------------------|
| 135   | TCP      | MS RPC           | 10.0.2.2         |
| 445   | TCP      | Microsoft-DS (SMB)| 10.0.2.2        |
| 53    | TCP      | DNS              | 10.0.2.3         |
| 443   | TCP      | HTTPS (TLSv1.2)  | 37.221.196.71    |
| —     | UDP/TCP  | OpenVPN (encrypted) | 37.33.194.x  |

---

## 🛡️ Phase 5: Security Risk Analysis

### 🔒 Port 135/tcp – MS RPC

- **Purpose:** DCOM and RPC services in Windows environments.  
- **Risks:**  
  - Targeted in **Blaster worm**, **remote code execution** vulnerabilities.  
  - Should never be publicly exposed.  
- **Recommendation:** Block at the **perimeter firewall** and monitor.

---

### 🔒 Port 445/tcp – Microsoft-DS (SMB)

- **Purpose:** File and printer sharing via **SMB protocol**.  
- **Risks:**  
  - **EternalBlue** exploit (WannaCry, NotPetya ransomware).  
  - Common vector for **lateral movement**.  
- **Recommendation:**  
  - **Disable SMB** if not required.  
  - Restrict access to trusted IPs and **apply security patches**.

---

### 🔒 Port 53/tcp – DNS

- **Purpose:** Resolves domain names to IPs.  
- **Risks:**  
  - Susceptible to **DNS tunneling**, **cache poisoning**, **DoS amplification**.  
- **Recommendation:**  
  - Harden DNS servers, enable **rate-limiting**, restrict recursion, and use **DNSSEC**.

---

## 📘 Outcomes & Key Learnings

- Identified potential risks from **exposed SMB and WSD** services.
- Highlighted the need for monitoring **obscure ports** (e.g., 5357 – often ignored).
- Demonstrated the power of combining **Nmap (stealth scanning)** and **Wireshark (live analysis)** for in-depth reconnaissance.
- Validated the effectiveness of passive and active scanning for real-world vulnerability assessment.

---

## 🧰 Tools Used

| Tool       | Purpose                                |
|------------|----------------------------------------|
| **Nmap**   | Port scanning and host discovery       |
| **Wireshark** | Deep packet inspection and traffic analysis |
| **Manual Research** | Interpreting services, vulnerabilities (CVE lookups), and security recommendations |