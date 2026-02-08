# 🚀 Deep Dive: QUIC & HTTP/3 Protocol Analysis
### A Wireshark Case Study on Modern Web Traffic

## 📌 Project Overview
This project demonstrates a practical analysis of the **QUIC protocol (HTTP/3)** using Wireshark. Unlike traditional TCP+TLS connections, QUIC runs over UDP and encrypts transport metadata, making analysis challenging.

By injecting SSL Session Keys (`SSLKEYLOGFILE`) into **Firefox/Curl**, I successfully decrypted the traffic to visualize the full lifecycle of a QUIC connection: from the initial DNS negotiation to the decrypted HTTP/3 GET request.

**Tools Used:**
* **OS:** Arch Linux (CachyOS)
* **Network Analyzer:** Wireshark
* **Browser/Client:** Firefox (Private Mode) & Curl
* **Target:** Facebook.com (Early adopter of HTTP/3)

---

## 🔍 The "Perfect Flow" Analysis

### Phase 1: The Negotiation (DNS & ALPN)
Before sending any data, the browser must know if the server supports QUIC. It doesn't guess; it asks via DNS.

![Full Flow Timeline](screenshots/1_Full_Flow_Timeline.png)
*Figure 1: The complete timeline showing DNS Query → DNS Response → Immediate QUIC Initial.*

**Observation:**
1.  **The Question:** Client sends a DNS Query for `www.facebook.com` type `HTTPS` (Type 65).
2.  **The Answer:** The DNS server responds with a Service Parameter **`alpn="h3"`**.
3.  **The Decision:** This tells the browser: *"Don't use TCP. Connect directly via UDP on port 443."*

---

### Phase 2: The "Zero-RTT" Connection (UDP)
Traditional HTTPS requires a TCP 3-way handshake (SYN, SYN-ACK, ACK) followed by a TLS handshake. QUIC merges these.

![QUIC Handshake](screenshots/3_quic_handshake.png)

**Key Technical Findings:**
* **No TCP Flags:** There are no SYN/ACK packets.
* **Protocol:** Traffic switches immediately to **QUIC (UDP)** after the DNS response.
* **Connection ID (CID):** Unlike TCP ports, QUIC uses Connection IDs to maintain sessions even if the client's IP changes (e.g., switching from WiFi to Mobile Data).

---

### Phase 3: The Decryption (Breaking the Encryption)
Without SSL keys, Wireshark sees generic `QUIC Protected Payload`. By loading the session keys, I revealed the actual HTTP/3 structure.

![Decrypted GET Request](screenshots/4_decrypted_get.png)
*Figure 3: Decrypted Packet #319 showing the HTTP/3 GET Request.*

**Proof of Success:**
* **Protocol:** Displayed as `HTTP3` instead of encrypted UDP garbage.
* **Method:** We can clearly see the **`:method: GET`** header.
* **Path:** The browser requested `/security/hsts-pixel.gif`.
* **Authority:** The request was sent to `facebook.com`.

---

## 🛠️ How to Reproduce

If you want to replicate this analysis, follow these steps on a Linux machine:

**1. Kill existing browser instances:**
```bash
pkill firefox
