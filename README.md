
# 🧠 Pi-hole Installation & Troubleshooting on Raspberry Pi (Full Technical Guide)

A complete, step-by-step, and *field-tested* guide to install, configure, debug, and stabilize **Pi-hole** on a **Raspberry Pi** — based entirely on real-world issues and resolutions during a full deployment process.  

---

## 📋 Table of Contents

1. [Overview](#Overview)  
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
14. [Common Errors & Fix Summary](#common-errors--fix-summary)

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
```bash
sudo apt update && sudo apt upgrade -y
````

---

### 2️⃣ Install Pi-hole

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

```bash
sudo nano /etc/dhcpcd.conf
```

```
interface eth0
static ip_address=192.168.0.48/24
static routers=192.168.0.1
static domain_name_servers=1.1.1.1 8.8.8.8
```

```bash
sudo service dhcpcd restart
```

---

## 🌐 Router DNS Configuration

```
Primary DNS: 192.168.0.48
Secondary DNS: 1.1.1.1
```

---

## ⚠️ Admin Interface Issues

```bash
sudo systemctl status lighttpd
sudo systemctl start lighttpd
sudo systemctl enable lighttpd
curl http://localhost/admin
ping 192.168.0.48
sudo ss -tuln | grep :80
sudo systemctl restart lighttpd
sudo systemctl restart pihole-FTL
```

Access:

```
http://192.168.0.48/admin
```

---

## ❌ Lighttpd Missing or Failed

```bash
sudo apt update
sudo apt install lighttpd -y
sudo pihole -r
sudo systemctl status lighttpd
```

---

## ❌ SetupVars.conf Missing

```bash
sudo nano /etc/pihole/setupVars.conf
```

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

If Wi-Fi:

```
PIHOLE_INTERFACE=wlan0
```

```bash
sudo systemctl restart pihole-FTL
sudo systemctl restart lighttpd
```

---

## 🧠 DNS Timeout / Query Refused Fix

```bash
sudo sed -i 's|DNSMASQ_LISTENING=.*|DNSMASQ_LISTENING=all|' /etc/pihole/setupVars.conf
sudo pihole reloaddns
sudo netstat -tulnp | grep :53
```

Expected:

```
tcp   0   0 0.0.0.0:53   0.0.0.0:*   LISTEN   pihole-FTL
udp   0   0 0.0.0.0:53   0.0.0.0:*           pihole-FTL
```

---

## 🔍 Testing Commands

```bash
nslookup google.com 127.0.0.1
nslookup google.com 192.168.0.48
sudo systemctl status pihole-FTL
sudo systemctl status lighttpd
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

| Concept               | Explanation                                   |
| --------------------- | --------------------------------------------- |
| **Pi-hole**           | Local DNS sinkhole to block ads and trackers. |
| **DNS (Port 53)**     | Resolves domains to IP addresses.             |
| **FTL**               | Pi-hole’s high-performance DNS engine.        |
| **Lighttpd**          | Lightweight web server for Pi-hole dashboard. |
| **Static IP**         | Ensures DNS address consistency.              |
| **setupVars.conf**    | Core Pi-hole configuration file.              |
| **DNSMASQ_LISTENING** | Defines who can query DNS (`local` or `all`). |

---

## 🧾 Full Command Reference

```bash
sudo apt update && sudo apt upgrade -y
curl -sSL https://install.pi-hole.net | bash
sudo systemctl status pihole-FTL
sudo systemctl status lighttpd
sudo systemctl restart pihole-FTL
sudo systemctl restart lighttpd
sudo apt install lighttpd -y
sudo pihole -r
sudo nano /etc/pihole/setupVars.conf
sudo sed -i 's|DNSMASQ_LISTENING=.*|DNSMASQ_LISTENING=all|' /etc/pihole/setupVars.conf
sudo pihole reloaddns
nslookup google.com 127.0.0.1
nslookup google.com 192.168.0.48
sudo ss -tuln | grep :80
sudo netstat -tulnp | grep :53
```

---

## 📉 Common Errors & Fix Summary

| Error / Issue                                      | Cause                            | Fix                                                                                     |                      |                       |                                                        |
| -------------------------------------------------- | -------------------------------- | --------------------------------------------------------------------------------------- | -------------------- | --------------------- | ------------------------------------------------------ |
| `curl: command not found`                          | curl not installed               | `sudo apt install curl -y`                                                              |                      |                       |                                                        |
| `lighttpd: command not found`                      | web server missing               | `sudo apt install lighttpd -y`                                                          |                      |                       |                                                        |
| Dashboard not opening (`http://<IP>/admin`)        | lighttpd not running             | `sudo systemctl start lighttpd`                                                         |                      |                       |                                                        |
| `pihole-FTL` failed to start                       | FTL crashed or missing config    | `sudo systemctl restart pihole-FTL`                                                     |                      |                       |                                                        |
| `nslookup google.com 192.168.0.48 → Query refused` | Pi-hole listening only locally   | `sudo sed -i 's                                                                         | DNSMASQ_LISTENING=.* | DNSMASQ_LISTENING=all | ' /etc/pihole/setupVars.conf && sudo pihole reloaddns` |
| `Timeout` during DNS queries                       | wrong IP or service down         | Restart Pi-hole and check `sudo netstat -tulnp                                          | grep :53`            |                       |                                                        |
| `/etc/pihole/setupVars.conf` missing               | config deleted or corrupted      | recreate manually (see above section)                                                   |                      |                       |                                                        |
| `lighttpd: failed to start`                        | bad configuration                | `sudo lighttpd -t -f /etc/lighttpd/lighttpd.conf` to test syntax, then restart          |                      |                       |                                                        |
| `curl http://localhost/admin` returns nothing      | web server down                  | `sudo systemctl restart lighttpd`                                                       |                      |                       |                                                        |
| `DNS not resolving`                                | Pi-hole service stopped          | `sudo systemctl status pihole-FTL` then `sudo systemctl restart pihole-FTL`             |                      |                       |                                                        |
| Can't reach Pi-hole from another device            | Firewall / interface restriction | `DNSMASQ_LISTENING=all` and check router DNS set to Pi IP                               |                      |                       |                                                        |
| `403 Forbidden` on dashboard                       | Permission or FTL cache issue    | `sudo systemctl restart pihole-FTL && sudo systemctl restart lighttpd`                  |                      |                       |                                                        |
| IP changed after reboot                            | DHCP conflict                    | Reserve Pi IP in router or make static in `/etc/dhcpcd.conf`                            |                      |                       |                                                        |
| `sudo: pihole: command not found`                  | PATH or install issue            | Re-run installer: `curl -sSL [https://install.pi-hole.net](https://install.pi-hole.net) |                      |                       |                                                        

---

### 🧠 Maintainer

**GitHub:** [thorerstic](https://github.com/thorerstic)

```
```
