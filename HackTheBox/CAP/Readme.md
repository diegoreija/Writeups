# CAP — HackTheBox

**Difficulty:** Easy
**OS:** Linux
**Date:** June 2026

## Summary

Cap is an Easy-rated Linux machine on HackTheBox. The name is a hint at the two core concepts involved: **PCAP** network captures and **Linux capabilities**. A web application exposes network traffic captures through an IDOR vulnerability, leaking FTP credentials in cleartext. These credentials grant SSH access, and privilege escalation is achieved through a misconfigured Python capability.

## Reconnaissance

### Connectivity check

```bash
ping 10.129.30.28
```

A response with TTL 63 confirms the target is a Linux host (Windows machines typically respond with TTL 127 or 128).

![Descripción de la imagen](./screenshots/01)
### Port scanning

```bash
nmap -sV -sS 10.129.30.28
```

**Flags used:**

| Flag | Purpose |
|---|---|
| `-sV` | Detects the service version running on each open port |
| `-sS` | TCP SYN scan — faster and stealthier than a full TCP connect scan |

[Insertar aquí la imagen original - nmap scan]

**Results:**

| Port | Service | Version |
|---|---|---|
| 21 | FTP | vsftpd 3.0.3 |
| 22 | SSH | OpenSSH 8.2p1 (Ubuntu) |
| 80 | HTTP | Gunicorn (Python WSGI server) |

> **Note:** FTP transmits all data — including credentials — in cleartext, and sometimes allows anonymous login. This makes it a service worth revisiting once more information is available.

## Web Enumeration and IDOR Vulnerability

Browsing to port 80 reveals a **Security Dashboard** belonging to a user named `nathan`.

[Insertar aquí la imagen original - dashboard]

Within the sidebar, a **"Security Snapshot"** section generates network reports. Accessing a report loads a URL ending in `/data/1`, which contains no relevant data.

Manually changing the ID to `/data/0` exposes a previous report that should not be accessible — an **Insecure Direct Object Reference (IDOR)** vulnerability. This report reveals a packet capture containing 72 recorded packets.

[Insertar aquí la imagen original - IDOR /data/0]

## Traffic Analysis and Credential Extraction

The exposed report allows downloading a `.pcap` file, opened with **Wireshark** for inspection.

[Insertar aquí la imagen original - Wireshark overview]

Filtering the capture for FTP traffic and following the TCP stream reveals the full authentication exchange in cleartext:
