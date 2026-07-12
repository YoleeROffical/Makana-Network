<div align="center">

# 🏠 Makana Network

*A production-inspired homelab built to develop practical skills in Linux administration, virtualization, networking, security, and self-hosting.*

Makana Network powers my personal infrastructure using a clustered Proxmox environment, self-hosted networking services, reverse proxies, VPN, email, media stack, and automated workflows.

This repository serves as both operational documentation and a record of the architectural decisions behind the environment.

*Clustered Proxmox nodes · One Oracle VPS · Self-managed DNS · Self-hosted VPN · Self-hosted Mail*

---

[![Proxmox](https://img.shields.io/badge/Proxmox-E57000?style=flat&logo=proxmox&logoColor=white)](https://proxmox.com)
[![NetBird](https://img.shields.io/badge/NetBird-Mesh%20VPN-orange?style=flat)](https://netbird.io)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)](https://docker.com)
[![Let's Encrypt](https://img.shields.io/badge/TLS-Let's%20Encrypt-003A70?style=flat)](https://letsencrypt.org)

</div>
---

## 📊 Infrastructure Statistics

| Resource | Count |
|----------|------:|
| Physical Servers | 2 |
| Cloud VPS | 1 |
| Linux Containers | 6 |
| Docker Containers | 20+ |
| Public Services | 8 |
| Domains | 1 |
| Reverse Proxies | 1 |

---

## 🚀 Project at a Glance

- 🖥️ 2-node Proxmox VE cluster
- 📦 6 production Linux Containers
- 🐳 20+ Docker services
- 🌐 Self-hosted DNS, VPN & Email
- 🔒 Reverse proxy with TLS & CrowdSec
- ☁️ Hybrid on-prem + Oracle Cloud VPS
- 📚 Complete infrastructure documentation

---

## 📖 Overview

Makana Network is a personal homelab running entirely on self-owned hardware and Oracle VPS. I aim to minimize reliance on third-party infrastructure wherever practical while self-hosting core services such as DNS, VPN and email.

The setup consists of two **clustered** on-premise Proxmox nodes (James + Stella) and one Oracle Cloud VPS (Fred), all connected via a self-hosted NetBird WireGuard mesh. James and Stella share a single Proxmox cluster with quorum device support from Fred, allowing unified management and live LXC migration between nodes. Services run inside Docker LXCs, organized by function.

---

## 🖥️ Hardware

| Node | Hostname | Role | CPU | RAM | Storage |
|------|----------|------|-----|-----|---------|
| Proxmox Node 1 | **James** | Cluster node · Game servers · Future TrueNAS | Intel i7-6700 | 32GB DDR4 | SSD |
| Proxmox Node 2 | **Stella** | Cluster node · Primary services · 2TB media HDD | Intel i5-4590 | 12GB DDR3 | 1x SSD, 1x 2TB HDD |
| Oracle Cloud VPS | **Fred** | Mail server · NetBird management · Cluster QDevice | AMD (Always Free) | 1GB + 8GB swap | 44GB |

> **ℹ️ Note:** James is the intended primary/performance node. Once the HBA and SAS drives arrive, James will host TrueNAS Scale and become the main storage and compute node. Stella hosts most services in the interim, with James currently running game servers and standing by as a live migration/failover target.

---

## 🧩 Proxmox Cluster

James and Stella are joined into a single Proxmox cluster (`prox-cluster`), managed from either node's web UI (`https://192.168.1.10:8006` or `https://192.168.1.11:8006`).

| Setting | Value |
|---------|-------|
| **Cluster name** | `prox-cluster` |
| **Corosync link** | Direct LAN (`192.168.1.10` ↔ `192.168.1.11`) — not routed over NetBird |
| **Quorum** | 2 node votes + 1 QDevice vote (Fred) = 3 total |
| **QDevice host** | Fred, via `corosync-qnetd` on port `5403` (opened in both Oracle Security List and local iptables) |
| **Storage per node** | Local `local-lvm` (LVM-thin) — no shared/network storage yet |
| **Migration type** | Offline (`pct migrate <vmid> <node> --restart`) — disk copied over direct LAN |

The QDevice on Fred acts as a tie-breaking third vote, so the cluster retains quorum (and stays writable) if either James or Stella goes offline individually — a plain 2-node cluster would otherwise lose quorum and become read-only if one node dropped.

> **Note:** True unplanned-failure HA (auto-restart on power loss) isn't yet possible since storage is local-only — see [Planned: Storage & HA](#-storage--ha-pending-hardware-arrival) below.

---

## 🎯 Why this project exists

Makana Network started as a way to learn Linux and self-hosting, but has grown into the infrastructure that powers my personal services and continues to serve as my learning platform for systems administration and networking.

The goal is to understand how infrastructure is designed, deployed, secured, documented and maintained.

Every major change is documented in this repository to provide both operational documentation and a historical record of architectural decisions.
---

## 🏗️ Architecture

The environment follows a layered design.

Internet
        ↓
Router
        ↓
Networking LXC
        ↓
Reverse Proxy
        ↓
Service LXCs
        ↓
Docker Containers

Core networking services are isolated from application workloads, with infrastructure split across dedicated Linux Containers to reduce service coupling and simplify maintenance.

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
| **ISP** | Broadband2 Sweden — 1000/1000 Mbps (~800-940Mbps real-world) |

**Planned network upgrade:** replacing the current switch with a 2.5GbE model and connecting James directly to it (rather than through the AP-router uplink), removing the AP-router as a bottleneck for James ↔ Stella traffic. The AP-router will then serve purely as a Wi-Fi access point.

---

## 🗂️ Nodes & Containers

Container VMIDs are unified cluster-wide (100–105) and organized numerically by function, regardless of which physical node currently hosts them.

### 🟠 James — `192.168.1.10`
> Cluster node. Currently running game servers, standing by as capacity for future services and migration target.

| VMID | LXC | IP | Services |
|------|-----|----|---------|
| 104 | Game-Servers | `192.168.1.34` | Crafty Controller · Minecraft servers · Velocity proxy |
| 105 | docker (test) | DHCP | Test/sandbox container |

---

### 🟣 Stella — `192.168.1.11`
> Cluster node. Currently hosting all primary services, plus the 2TB media HDD (`/mnt/pve/media`, mounted into Torrenting and Media containers).

| VMID | LXC | IP | Services |
|------|-----|----|---------|
| 100 | Networking | `192.168.1.32` | NGINX Proxy Manager · Pi-hole · Unbound · Crowdsec · ddclient — **HA-managed** |
| 101 | Services | `192.168.1.33` | Nextcloud · Vaultwarden · Homepage · Portainer |
| 102 | Media | `192.168.1.31` | Jellyfin · Jellyseerr · Wizarr |
| 103 | Torrenting | `192.168.1.30` | qBittorrent · Sonarr · Radarr · Bazarr · Prowlarr · Profilarr |

---

### 🔵 Fred — Oracle Cloud VPS (`82.70.56.40` public / `100.100.127.163` NetBird)
> Always-on cloud node. Public-facing mail, NetBird coordination, and cluster quorum witness.

| Service | Purpose |
|---------|---------|
| Maddy | Self-hosted SMTP/IMAP mail server |
| NetBird stack | Zitadel · Management · Signal · Relay · Caddy |
| corosync-qnetd | Proxmox cluster QDevice (tie-breaking quorum vote for James/Stella) |

---

## 🛡️ High Availability

The **Networking** LXC (100 — Pi-hole, NPM, Unbound, DNS) is configured as an HA-managed resource, since a full network/DNS outage on this container takes down internet access for the whole house.

| Setting | Value |
|---------|-------|
| **HA resource** | `ct:100` |
| **Affinity rule** | `networking-affinity` — Stella priority 2, James priority 1 (prefers Stella for faster network path; James is behind the AP-router bottleneck) |
| **Failback** | Enabled (default) — automatically returns to Stella once Stella rejoins after maintenance |
| **Failover type** | Planned/graceful only, via node maintenance mode — **not** automatic on sudden power loss (requires local-only storage to be copied live, which needs the source node to still be responsive) |

**Planned maintenance workflow** (e.g. before rebooting Stella):
```bash
ha-manager crm-command node-maintenance enable Stella
# wait for ha-manager status to show ct:100 relocated + started on james
reboot
# once back up:
ha-manager crm-command node-maintenance disable Stella
# ct:100 auto-migrates back to Stella due to failback + priority
```

**Manual migration** (non-HA containers, e.g. moving Services back to Stella):
```bash
# run FROM the node the container currently lives on
pct migrate 101 Stella --restart
```

> True zero-downtime failover isn't achievable yet for any container, since LXC live migration (CRIU-based) still involves a brief freeze, and Docker's internal networking/namespaces make it unreliable anyway — restart-style migration (~2 minutes for an 8GB container over direct LAN) is the practical baseline until shared/replicated storage is in place.

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

## 💿 ARM

### The Media workload includes a custom automated ripping workflow:

Disc insertion on James
        |
        v
udev host trigger
        |
        v
CT 102 worker
        |
        +--> MakeMKV rip
        +--> TMDb title lookup
        +--> Intel VA-API H.264 transcode
        +--> Embedded audio and subtitles preserved
        +--> Final Radarr/Jellyfin folder naming
        `--> Copy into /media/content/movies as UID:GID 1000:1000
```

The Intel i7-6700 uses two different VA-API drivers for compatibility:

| Workload | Driver | Mode |
|---|---|---|
| ARM transcoding | `iHD` | H.264 low-power CQP |
| Jellyfin transcoding | `i965` | H.264 VBR |

Detailed host documentation is under [`Hosts/James/ARM/`](Hosts/James/ARM/).

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

## 🔒 Security

| Layer | Implementation |
|-------|---------------|
| Intrusion detection | Crowdsec + iptables bouncer (Networking LXC) |
| Firewall | Minimal open ports — 80, 443, 25565, 25566 (home), 5403 (Fred, cluster QDevice only) |
| Inter-node traffic | Direct LAN for cluster/migration traffic; encrypted WireGuard (NetBird) for remote management and Fred communication |
| TLS | Let's Encrypt via NPM for all public services |
| DNS privacy | Unbound recursive resolver — no upstream provider |
| Dynamic IP | ddclient → Namecheap API (no DuckDNS dependency) |
| Proxying | No Cloudflare — fully self-managed |
| SSH | Key-based auth only on Fred (no password auth) |

---

## 🗺️ Planned

### ⚡ Short Term
- [ ] Uptime Kuma on Fred for external monitoring + phone alerts
- [ ] Automated Maddy mail backups
- [ ] 2.5GbE switch upgrade — direct James connection, AP-router demoted to Wi-Fi-only

### 💾 Storage & HA *(pending hardware arrival)*
- [ ] Dell H310 HBA (LSI 9211-8i, IT Mode)
- [ ] 3x HP 8TB SAS HDDs — ZFS RAIDZ pool on James
- [ ] TrueNAS Scale VM on James with NFS shares
- [ ] Ceph cluster or ZFS replication (James + Stella) for true unplanned-failure HA
- [ ] Proxmox Backup Server → TrueNAS pool

### 🔮 Future
- [ ] Extend HA to Media/Torrenting once shared/replicated storage exists
- [ ] Dual-node HA mail once shared storage is in place
- [ ] Dovecot replication for redundant IMAP
- [ ] Homepage widgets (Proxmox, Jellyfin, Pi-hole API integration)
- [ ] Expand Minecraft server capacity for larger player count

---

## 📖 Technical Experience Gained

Building Makana Network has given me hands-on experience with:

- Linux system administration
- Networking and DNS
- Virtualization with Proxmox
- High availability concepts
- Reverse proxying
- VPN infrastructure
- Self-hosted email
- Security hardening
- Infrastructure documentation
- Troubleshooting production-like systems

---

<div align="center">

*Built and maintained by Tony/YoleeR*

</div>
