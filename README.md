````markdown
# 🧠 Pi-hole Installation & Troubleshooting on Raspberry Pi (Full Technical Guide)

A complete, step-by-step, and *field-tested* guide to install, configure, debug, and stabilize **Pi-hole** on a **Raspberry Pi** — based entirely on real-world issues and resolutions during a full deployment process.  

---

## 📋 Table of Contents

1. [Overview](#overview)  
2. [Environment & Requirements](#environment--requirements)  
3. [Installation Steps](#installation-steps)  
4. [Static IP Configuration](#static-ip-configuration)  
5. [Router DNS Configuration](#router-dns-configuration)  
6. [Admin Interface Issues](#admin-interface-issues)  
7. [Lighttpd Missing or Failed](#lighttpd-missing-or-failed)  
8. [SetupVars.conf Missing](#setupvarsconf-missing)  
9. [DNS Timeout / Query Refused Fix](#dns-timeout--query-refused-fix)  
10. [Testing Commands](#testing-commands)  
11. [Maintenance Commands](#maintenance-commands)  
12. [Concepts Recap](#concepts-recap)  
13. [Full Command Reference](#full-command-reference)  

---

## 🧾 Overview

**Pi-hole** is a local DNS server that blocks ads, trackers, and malicious domains *before* they ever reach your devices.  
In this guide, we’ll build, fix, and optimize Pi-hole on a **Raspberry Pi 1** connected via Ethernet.

---

## ⚙️ Environment & Requirements

| Component | Description |
|------------|--------------|
| **Device** | Raspberry Pi 1 (ARMv6) |
| **OS** | Raspberry Pi OS (Lite / Debian-based) |
| **Connection** | Ethernet (`eth0`) |
| **Static IP** | `192.168.0.48` |
| **Router Gateway** | `192.168.0.1` |
| **Upstream DNS** | Cloudflare `1.1.1.1` |
| **Web Server** | `lighttpd` |
| **DNS Engine** | `pihole-FTL` |

---

## 🧱 Installation Steps

### 1️⃣ Update the System
Always update before any installation:
```bash
sudo apt update && sudo apt upgrade -y
````

This ensures your system libraries and dependencies match the Pi-hole version.

---

### 2️⃣ Install Pi-hole

Run the official one-liner installer:

```bash
curl -sSL https://install.pi-hole.net | bash
```

During setup:

| Step         | Option              | Selected                    |
| ------------ | ------------------- | --------------------------- |
| Interface    | Network interface   | `eth0`                      |
| Static IP    | Address reservation | `192.168.0.48`              |
| Upstream DNS | Provider            | Cloudflare `1.1.1.1`        |
| Blocklists   | Ad sources          | Default (StevenBlack hosts) |

---

## 🧩 Static IP Configuration

To make sure Pi-hole’s IP never changes:

```bash
sudo nano /etc/dhcpcd.conf
```

Add:

```
interface eth0
static ip_address=192.168.0.48/24
static routers=192.168.0.1
static domain_name_servers=1.1.1.1 8.8.8.8
```

Then apply:

```bash
sudo service dhcpcd restart
```

---

## 🌐 Router DNS Configuration

In your router admin page (`192.168.0.1`), set:

```
Primary DNS: 192.168.0.48
Secondary DNS: 1.1.1.1
```

This makes all devices use Pi-hole as their main DNS resolver.

---

## ⚠️ Admin Interface Issues

### 1️⃣ Check if `lighttpd` is running:

```bash
sudo systemctl status lighttpd
```

If not active:

```bash
sudo systemctl start lighttpd
sudo systemctl enable lighttpd
```

---

### 2️⃣ Test local web interface:

```bash
curl http://localhost/admin
```

If it returns HTML → the web server is running correctly.

---

### 3️⃣ Check connectivity:

```bash
ping 192.168.0.48
```

---

### 4️⃣ Verify port 80:

```bash
sudo ss -tuln | grep :80
```

Expected output should show `LISTEN` on `0.0.0.0:80`.

---

### 5️⃣ Restart both web and DNS services:

```bash
sudo systemctl restart lighttpd
sudo systemctl restart pihole-FTL
```

Access again from browser:

```
http://192.168.0.48/admin
```

---

## ❌ Lighttpd Missing or Failed

If you get `lighttpd: command not found` or dashboard doesn’t start:

```bash
sudo apt update
sudo apt install lighttpd -y
sudo pihole -r
sudo systemctl status lighttpd
```

This re-installs and auto-enables the Pi-hole web server.

---

## ❌ SetupVars.conf Missing

If `/etc/pihole/setupVars.conf` is missing, recreate it:

```bash
sudo nano /etc/pihole/setupVars.conf
```

Paste:

```
PIHOLE_INTERFACE=eth0
IPV4_ADDRESS=192.168.0.48/24
IPV6_ADDRESS=
QUERY_LOGGING=true
INSTALL_WEB_SERVER=true
INSTALL_WEB_INTERFACE=true
LIGHTTPD_ENABLED=true
DNSMASQ_LISTENING=local
DNS_FQDN_REQUIRED=true
DNS_BOGUS_PRIV=true
DNSSEC=false
REV_SERVER=false
```

If using Wi-Fi:

```
PIHOLE_INTERFACE=wlan0
```

Then:

```bash
sudo systemctl restart pihole-FTL
sudo systemctl restart lighttpd
```

---

## 🧠 DNS Timeout / Query Refused Fix

If:

```
nslookup google.com 192.168.0.48
```

returns **Query Refused** or **Timeout**,
Pi-hole is restricted to “local” queries.

Allow all LAN clients:

```bash
sudo sed -i 's|DNSMASQ_LISTENING=.*|DNSMASQ_LISTENING=all|' /etc/pihole/setupVars.conf
sudo pihole reloaddns
```

Verify Pi-hole is listening globally:

```bash
sudo netstat -tulnp | grep :53
```

Expected output:

```
tcp   0   0 0.0.0.0:53   0.0.0.0:*   LISTEN   pihole-FTL
udp   0   0 0.0.0.0:53   0.0.0.0:*           pihole-FTL
```

---

## 🔍 Testing Commands

### Test local DNS resolution

```bash
nslookup google.com 127.0.0.1
```

### Test network DNS resolution

```bash
nslookup google.com 192.168.0.48
```

### Verify service status

```bash
sudo systemctl status pihole-FTL
sudo systemctl status lighttpd
```

### Check open ports

```bash
sudo ss -tuln | grep :80
sudo netstat -tulnp | grep :53
```

---

## 🧰 Maintenance Commands

| Purpose                     | Command                             |
| --------------------------- | ----------------------------------- |
| Update Pi-hole              | `pihole -up`                        |
| Update blocklists (gravity) | `pihole -g`                         |
| Reset admin password        | `pihole -a -p`                      |
| Repair Pi-hole              | `pihole -r`                         |
| Restart DNS                 | `sudo systemctl restart pihole-FTL` |
| Restart web server          | `sudo systemctl restart lighttpd`   |
| Flush logs                  | `pihole -f`                         |
| Live log view               | `pihole -t`                         |

---

## 🧩 Concepts Recap

| Concept               | Explanation                                                              |
| --------------------- | ------------------------------------------------------------------------ |
| **Pi-hole**           | A DNS sinkhole that blocks ads and trackers network-wide.                |
| **DNS (Port 53)**     | Translates human-readable domain names into IP addresses.                |
| **FTL**               | Pi-hole’s DNS engine – fast, cached, and lightweight.                    |
| **Lighttpd**          | Lightweight web server that hosts the Pi-hole admin interface.           |
| **Static IP**         | Prevents the Pi-hole address from changing (critical for DNS stability). |
| **setupVars.conf**    | Core configuration file for all Pi-hole parameters.                      |
| **DNSMASQ_LISTENING** | Defines who can query the DNS (local vs all LAN devices).                |

---

## 🧾 Full Command Reference

```bash
# Update System
sudo apt update && sudo apt upgrade -y

# Install Pi-hole
curl -sSL https://install.pi-hole.net | bash

# Verify services
sudo systemctl status pihole-FTL
sudo systemctl status lighttpd

# Restart services
sudo systemctl restart pihole-FTL
sudo systemctl restart lighttpd

# Fix missing lighttpd
sudo apt install lighttpd -y
sudo pihole -r

# Fix missing setupVars.conf
sudo nano /etc/pihole/setupVars.conf

# Allow all LAN devices to query DNS
sudo sed -i 's|DNSMASQ_LISTENING=.*|DNSMASQ_LISTENING=all|' /etc/pihole/setupVars.conf
sudo pihole reloaddns

# Test DNS resolution
nslookup google.com 127.0.0.1
nslookup google.com 192.168.0.48

# Check listening ports
sudo ss -tuln | grep :80
sudo netstat -tulnp | grep :53
```

---

## ✅ Final Result

After following this guide:

* ✅ Pi-hole is running as a **primary DNS resolver** on `192.168.0.48`.
* ✅ The web interface is accessible via `http://192.168.0.48/admin`.
* ✅ All DNS traffic is filtered, ads blocked, and analytics viewable in real time.
* ✅ The system is fully stable and persistent across reboots.

---

## 🧠 Author Notes

This guide documents a **real debugging session** with a Raspberry Pi 1.
Every issue, command, and resolution here was tested and verified in a live environment.

If this helped you — ⭐ star the repo, and feel free to contribute improvements or automation scripts for Pi-hole deployment.

---

### 💬 Contact

For questions or contributions:

```
github.com/thorestic
```
