<div align="center">

# 🏠 Makana Network

**A fully self-hosted homelab built for privacy, resilience, and zero third-party dependencies.**

*Two Proxmox nodes · One Oracle VPS · Self-managed DNS · Self-hosted VPN · Self-hosted Mail*

---

[![Proxmox](https://img.shields.io/badge/Proxmox-E57000?style=flat&logo=proxmox&logoColor=white)](https://proxmox.com)
[![NetBird](https://img.shields.io/badge/NetBird-Mesh%20VPN-orange?style=flat)](https://netbird.io)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)](https://docker.com)
[![Let's Encrypt](https://img.shields.io/badge/TLS-Let's%20Encrypt-003A70?style=flat)](https://letsencrypt.org)

</div>

---

## 📖 Overview

Makana Network is a personal homelab running entirely on self-owned hardware and self-managed services. No Cloudflare, no Tailscale, no DuckDNS — everything from DNS to VPN to email is owned and operated in-house.

The setup consists of two on-premise Proxmox nodes and one Oracle Cloud VPS, all connected via a self-hosted NetBird WireGuard mesh. Services run inside Docker LXCs on Proxmox, organized by function.

---

## 🖥️ Hardware

| Node | Hostname | Role | CPU | RAM | Storage |
|------|----------|------|-----|-----|---------|
| Proxmox Node 1 | **James** | Primary node · Game servers · Future TrueNAS | Intel i7-6700 | 32GB DDR4 | SSD |
| Proxmox Node 2 | **Stella** | Secondary node · All current services | Intel i5-4590 | 12GB DDR3 | 1x SSD | 1x HDD |
| Oracle Cloud VPS | **Fred** | Mail server · NetBird management | AMD (Always Free) | 1GB + 8GB swap | 44GB |

> **ℹ️ Note:** James is the intended primary node. Once the HBA and SAS drives arrive, James will host TrueNAS Scale and become the main storage and compute node. Stella hosts all services in the interim.

---

## 🌐 Network

| Setting | Value |
|---------|-------|
| **Local subnet** | `192.168.1.0/24` |
| **Mesh VPN** | NetBird (self-hosted on Fred) |
| **Mesh subnet** | `100.100.0.0/16` |
| **DNS** | Pi-hole + Unbound (Networking LXC, Stella) |
| **Reverse proxy** | NGINX Proxy Manager |
| **Domain** | `yoleer.eu` (Namecheap) |
| **Dynamic DNS** | ddclient → Namecheap API |
| **ISP** | Broadband2 Sweden — 1000/1000 Mbps |

---

## 🗂️ Nodes

### 🟠 James — `192.168.1.10`
> Intended primary node. Currently running game servers only, pending storage hardware arrival.

| LXC | IP | Services |
|-----|----|---------|
| Game-Servers | `192.168.1.34` | Crafty Controller · Minecraft servers · Velocity proxy |

---

### 🟣 Stella — `192.168.1.11`
> Secondary node. Currently hosting all primary services while James is being prepared.

| LXC | IP | Services |
|-----|----|---------|
| Torrenting | `192.168.1.30` | qBittorrent · Sonarr · Radarr · Bazarr · Prowlarr · Profilarr |
| Media | `192.168.1.31` | Jellyfin · Jellyseerr · Wizarr |
| Networking | `192.168.1.32` | NGINX Proxy Manager · Pi-hole · Unbound · Crowdsec · ddclient |
| Services | `192.168.1.33` | Nextcloud · Vaultwarden · Homepage · Portainer |

---

### 🔵 Fred — Oracle Cloud VPS
> Always-on cloud node. Public-facing mail and NetBird coordination server.

| Service | Purpose |
|---------|---------|
| Maddy | Self-hosted SMTP/IMAP mail server |
| NetBird stack | Zitadel · Management · Signal · Relay · Caddy |

---

## 🔒 Mesh VPN

All nodes connected via **self-hosted NetBird** using encrypted WireGuard tunnels. No third-party VPN provider.

| Peer | Hostname | Type |
|------|----------|------|
| James | `james.netbird.selfhosted` | Home node |
| Stella | `stella.netbird.selfhosted` | Home node |
| Fred | `fred.netbird.selfhosted` | Cloud VPS |
| Mark-1 | `mark-1.netbird.selfhosted` | Desktop (CachyOS) |
| Mark-2 | `mark-2.netbird.selfhosted` | Laptop (CachyOS) |
| Tonys26 | `tonys26.netbird.selfhosted` | Android phone |

Subnet route `192.168.1.0/24` is advertised by both James and Stella for redundant home LAN access from any peer.

---

## 🧭 DNS Architecture

```
Device query
    └── Pi-hole (192.168.1.32:53)
            ├── Block lists (ad/tracker blocking)
            ├── Local DNS records (split-horizon for yoleer.eu → internal NPM IP)
            └── Unbound (recursive resolver → root DNS servers)
```

**Split-horizon DNS** prevents NAT hairpin issues — home devices resolve `jelly.yoleer.eu` etc directly to the internal NPM IP rather than the public home IP.

**Unbound** performs recursive resolution directly against root servers — no upstream DNS provider sees queries.

---

## 🌍 Public Services

All services exposed via **NGINX Proxy Manager** with **Let's Encrypt TLS**. Router only forwards ports `80`, `443`, `25565`, and `25566`.

| Subdomain | Service |
|-----------|---------|
| `jelly.yoleer.eu` | Jellyfin |
| `cloud.yoleer.eu` | Nextcloud |
| `vault.yoleer.eu` | Vaultwarden |
| `crafty.yoleer.eu` | Crafty Controller |
| `seerr.yoleer.eu` | Jellyseerr |
| `wizarr.yoleer.eu` | Wizarr |
| `netbird.yoleer.eu` | NetBird Dashboard |
| `mx.yoleer.eu` | Maddy Mail Server |

---

## 📧 Email

Self-hosted mail using **Maddy** on Fred — Google Free.

```
Inbound:   Internet → mx.yoleer.eu (Fred:25) → Maddy → IMAP
Outbound:  Maddy → SMTP2GO relay (port 587) → Internet
Access:    Thunderbird over NetBird mesh (no public IMAP/SMTP exposure)
```

> Outbound relay via SMTP2GO is required since both home ISP and Oracle Cloud block outbound port 25.

✅ SPF · DKIM · DMARC all configured · **10/10 on mail-tester.com**

---

## 🛡️ Security

| Layer | Implementation |
|-------|---------------|
| Intrusion detection | Crowdsec + iptables bouncer (Networking LXC) |
| Firewall | Minimal open ports — only 80, 443, 25565, 25566 |
| Inter-node traffic | Encrypted WireGuard via NetBird mesh |
| TLS | Let's Encrypt via NPM for all public services |
| DNS privacy | Unbound recursive resolver — no upstream provider |
| Dynamic IP | ddclient → Namecheap API (no DuckDNS dependency) |
| Proxying | No Cloudflare — fully self-managed |

---

## 🗺️ Planned

### ⚡ Short Term
- [ ] Uptime Kuma on Fred for external monitoring + phone alerts
- [ ] Automated Maddy mail backups

### 💾 Storage & HA *(pending hardware arrival)*
- [ ] Dell H310 HBA (LSI 9211-8i, IT Mode)
- [ ] 3x HP 8TB SAS HDDs — ZFS RAIDZ pool on James
- [ ] TrueNAS Scale VM on James with NFS shares
- [ ] Ceph cluster (James + Stella, Fred as quorum witness)
- [ ] Proxmox HA — live LXC migration between nodes
- [ ] Proxmox Backup Server → TrueNAS pool

### 🔮 Future
- [ ] HA for Pi-Hole between Stella & James for redundancy
- [ ] Dual-node HA mail once Ceph shared storage is in place
- [ ] Dovecot replication for redundant IMAP
- [ ] Homepage widgets (Proxmox, Jellyfin, Pi-hole API integration)
- [ ] Expand Minecraft server capacity for larger player count

---

<div align="center">

*Built and maintained by Tony/YoleeR*

</div>
