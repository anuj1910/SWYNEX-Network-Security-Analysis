# SWYNEX Network Security Analysis (Task 2)

**Intern:** Anuj Agrawal  
**Domain:** Cloud & Emerging Technology (Cyber Security)  
**Environment:** Controlled Local Kali Linux Lab (`tcpdump` & Wireshark)  
**Date:** November 2026  

---

## 1. Executive Summary
This repository documents a controlled network traffic analysis performed in a local lab environment using `tcpdump` and Wireshark. The objective was to identify observed network protocols, flag suspicious or high-risk communication patterns—such as cleartext HTTP authentication and SYN probing bursts—and recommend actionable defensive controls.

## 2. Scope & Methodology
* **Capture Source:** Controlled local Kali Linux lab packet capture (`captures/lab_traffic.pcap`)
* **Tools Used:** `tcpdump`, Wireshark (`http` and `tcp.flags.syn==1 && tcp.flags.ack==0` display filters, Protocol Hierarchy)
* **Methodology:** Protocol enumeration, filter-based inspection of cleartext and connection-setup traffic, risk evaluation, and defensive recommendation mapping.

---

## 3. Observed Protocols & Traffic Summary

| Protocol / Layer | Role in Capture | Key Observation |
| :--- | :--- | :--- |
| **DNS (UDP/TCP 53)** | Domain name resolution | Resolves hostnames prior to outbound connections |
| **HTTP (TCP 80)** | Unencrypted web traffic | Exposes request headers and payloads in cleartext |
| **TCP (SYN / ACK / RST)** | Transport session control | Shows connection handshakes, probes, or resets |
| **TLS / HTTPS (TCP 443)** | Encrypted transport | Protects payload confidentiality compared to plain HTTP |

---

## 4. Findings, Evidence & Recommended Mitigations

### Finding 01: Cleartext HTTP Traffic & Credential Exposure
* **Affected Traffic / Component:** Unencrypted HTTP requests/responses (`http://`)
* **Risk:** **High** — Plaintext HTTP transmits headers, URLs, and authentication data (such as HTTP Basic Auth) without encryption, allowing anyone on the network path or local interface to read sensitive data.
* **Evidence:** Filtered HTTP traffic in Wireshark (`http`) showing plaintext request headers and credentials.
  ![Wireshark HTTP Analysis](evidence/02_cleartext_http_auth.jpg)
* **Recommended Mitigation:**
  1. Enforce HTTPS/TLS across web services and redirect all HTTP traffic to HTTPS.
  2. Avoid sending credentials or session tokens over unencrypted channels.

---

### Finding 02: TCP SYN Probing / Connection Attempt Burst
* **Affected Traffic / Component:** TCP control plane (`tcp.flags.syn==1 && tcp.flags.ack==0`)
* **Risk:** **Medium** — Rapid or repeated SYN packets without completed handshakes can indicate reconnaissance, port scanning, or abnormal connection churn that warrants monitoring and rate limiting.
* **Evidence:** Filtered TCP SYN packets in Wireshark (`tcp.flags.syn==1 && tcp.flags.ack==0`).
  ![Wireshark SYN Pattern](evidence/03_syn_scan_pattern.jpg)
* **Recommended Mitigation:**
  1. Monitor unusual SYN rates or port-sweep behavior in IDS/IPS or firewall logs.
  2. Restrict exposed ports using host/network firewall rules and enable SYN flood protections where appropriate.

---

### Finding 03: Protocol Mix & Unencrypted Protocol Visibility
* **Affected Traffic / Component:** Overall network traffic mix (Protocol Hierarchy)
* **Risk:** **Low / Medium** — Mixed protocol usage and legacy/unencrypted protocols increase visibility gaps and attack surface if legacy services remain enabled.
* **Evidence:** Wireshark **Statistics > Protocol Hierarchy** view.
  ![Protocol Hierarchy](evidence/01_protocol_hierarchy.jpg)
* **Recommended Mitigation:**
  1. Baseline normal protocol distributions for your environment so anomalies stand out.
  2. Disable unnecessary legacy or cleartext protocols.

---

## 5. Authorization Disclaimer
All packet capture and traffic analysis activities were conducted solely in a controlled, authorized lab environment.
