# In-home-Pi-hole-DNS-Sinkhole-Network-Wide-Ad-Tracker-Blocking
A documented walkthrough of setting up Pi-hole as a DNS sinkhole on an Ubuntu home lab server, blocking ads and trackers at the network level for all configured devices.


# 🛡️ Pi-hole DNS Sinkhole — Network-Wide Ad & Tracker Blocking

A documented walkthrough of setting up Pi-hole as a DNS sinkhole on an Ubuntu home lab server, blocking ads and trackers at the network level for all configured devices.

---

## 📋 Project Overview

| Detail | Info |
|---|---|
| **OS** | Ubuntu 24.04.4 LTS (bare metal) |
| **Hardware** | Lenovo IdeaPad 5 15ITL05 |
| **Server IP** | 10.0.0.226 (static) |
| **DNS Sinkhole** | Pi-hole (latest) |
| **Upstream DNS** | Cloudflare (1.1.1.1) |
| **Web Interface** | http://10.0.0.226/admin |

---

## 🛠️ Skills Demonstrated

- DNS architecture and sinkhole configuration
- Network-level ad and tracker blocking
- Linux service management (systemd, lighttpd)
- UFW firewall rule management (ports 53, 80)
- IPv6 DNS bypass identification and remediation
- Troubleshooting and root cause analysis
- Pi-hole dashboard and query log analysis

---

## 🔍 What is a DNS Sinkhole?

A DNS sinkhole intercepts DNS requests before they reach the internet. When a device tries to load an ad, tracker, or malicious domain, Pi-hole intercepts the DNS query and returns a blocked response — preventing the content from ever loading. This works at the network level, meaning it blocks ads across all apps and browsers on configured devices.

---

## 📁 Steps

### 1. Install Pi-hole
Ran the official Pi-hole installer on the Ubuntu server:

```bash
curl -sSL https://install.pi-hole.net | bash
```

**Installer configuration choices:**
- Upstream DNS: **Cloudflare (1.1.1.1)** — fast, privacy-focused
- Default blocklist: **Enabled**
- Web admin interface: **Enabled**
- Query logging: **Enabled**
- Privacy mode: **Show everything** (home lab environment)

---

### 2. Configure Web Dashboard Access
Pi-hole's FTL process uses port 80 for the web interface. UFW was blocking external access, so opened port 80:

```bash
sudo ufw allow 80/tcp
sudo ufw status verbose
```

Dashboard accessible at:
```
http://10.0.0.226/admin
```

Set admin password:
```bash
sudo pihole setpassword
```

---

### 3. Open DNS Port on Firewall
Pi-hole listens for DNS queries on port 53. UFW was blocking incoming DNS requests from client devices, causing timeouts. Opened port 53 for both TCP and UDP:

```bash
sudo ufw allow 53
sudo ufw status verbose
```

---

### 4. Configure Client Devices to Use Pi-hole

#### Windows PC
Configured DNS manually via Control Panel:

1. **Control Panel** → **Network and Sharing Center** → **Change adapter settings**
2. Right click WiFi adapter → **Properties**
3. **Internet Protocol Version 4 (TCP/IPv4)** → **Properties**
4. Set DNS:
   - Preferred DNS: `10.0.0.226`
   - Alternate DNS: `1.1.1.1`
5. Disabled **IPv6** to prevent Xfinity DNS bypass via IPv6

**Verification:**
```
nslookup google.com
Server: pi.hole
Address: 10.0.0.226
```

#### iPhone
1. **Settings** → **WiFi** → tap ⓘ next to network
2. **Configure IP** → set to **Automatic (DHCP)**
3. **Configure DNS** → set to **Manual**
4. Added `10.0.0.226` as primary DNS
5. Added `1.1.1.1` as backup DNS

---

## ⚠️ Key Technical Challenges

### Challenge 1 — ISP Router DNS Lock
The Xfinity/Comcast router does not allow custom DNS configuration. Router-level DNS changes were not possible, requiring manual DNS configuration on each device individually.

### Challenge 2 — IPv6 DNS Bypass
After setting IPv4 DNS to `10.0.0.226` on the Windows PC, `nslookup` still showed Xfinity's DNS server (`cdns01.comcast.net`). Investigation revealed Windows was using an **IPv6 DNS address** (`2001:558:feed::1`) provided by Xfinity, bypassing the IPv4 Pi-hole setting entirely.

**Solution:** Disabled IPv6 on the Windows network adapter, forcing all DNS traffic through IPv4 and Pi-hole.

### Challenge 3 — UFW Blocking DNS Requests
After pointing the Windows PC to `10.0.0.226`, DNS requests were timing out. Pi-hole was running and listening on port 53, but UFW was blocking incoming connections on that port from client devices.

**Solution:** Opened port 53 on UFW:
```bash
sudo ufw allow 53
```

---

## ✅ Outcome

- Pi-hole running and blocking ads/trackers on the network
- Web dashboard accessible at `http://10.0.0.226/admin`
- Windows PC and iPhone routing DNS through Pi-hole
- Live query log showing real-time DNS traffic from all configured devices
- Blocked requests visible in dashboard with domain and client information
- UFW firewall rules updated to allow DNS (53) and web interface (80)

---

## 🔒 Current UFW Firewall Rules

| Port | Protocol | Purpose |
|---|---|---|
| 2222 | TCP | SSH (hardened, non-default port) |
| 53 | TCP/UDP | DNS (Pi-hole) |
| 80 | TCP | Pi-hole web dashboard |

---

## 📊 Dashboard Features

- **Total Queries** — all DNS requests from configured devices
- **Queries Blocked %** — percentage of requests blocked
- **Top Blocked Domains** — most frequently blocked ad/tracker domains
- **Query Log** — live feed of every DNS request with client IP and status
- **Client Identification** — identify which device made each request by IP

---

## 🔗 Related

- [Pi-hole Documentation](https://docs.pi-hole.net/)
- [UFW Documentation](https://help.ubuntu.com/community/UFW)
- [Cloudflare DNS](https://1.1.1.1/)
- [StevenBlack Blocklist](https://github.com/StevenBlack/hosts)

---

## 🔗 Related Projects

- [Ubuntu Home Lab Server Setup](../ubuntu-homelab-server)
- [SSH Hardening on Ubuntu 24.04](../ubuntu-ssh-hardening)

---

*Part of my ongoing home lab build documenting hands-on cybersecurity and Linux administration skills.*
