# Pi-hole Setup

Installed bare metal inside the **Networking LXC** on Stella (`192.168.1.32`).

Pi-hole was initially deployed as a Docker container but caused port 53 binding conflicts with Docker's internal DNS resolver and systemd-resolved. Moving to bare metal resolved all conflicts.

---

## Installation

```bash
curl -sSL https://install.pi-hole.net | bash
```

During setup, upstream DNS was set to `127.0.0.1#5335` (Unbound — see [unbound.md](unbound.md)).

Web UI runs on port `8081` to avoid conflict with NPM which owns port `80`.

---

## Router DHCP

Router DHCP hands out Pi-hole as the network-wide DNS server:

- **DNS Server 1:** `192.168.1.32` (Pi-hole)
- **DNS Server 2:** Is empty since routers sent out both dns servers and your device will just use the fastest one.
- **"Advertise router's IP in addition to user-specified DNS":** Disabled

---

## LXC DNS

Every LXC on James and Stella has DNS explicitly set to Pi-hole via Proxmox, since LXCs inherit the host DNS by default and the Proxmox hosts use NetBird's DNS which caused container resolution failures:

```bash
pct set <container_id> --nameserver 192.168.1.32 --searchdomain ""
```

Pi-hole's own LXC uses `9.9.9.9` as its upstream since it can't use itself.

---

## Split-horizon DNS

Local DNS records added in Pi-hole so home devices resolve `yoleer.eu` subdomains directly to NPM's internal IP (`192.168.1.32`) instead of the public home IP — this prevents NAT hairpin issues where requests would round-trip through the router and time out or load slowly.

| Domain | Resolves to |
|--------|------------|
| `yoleer.eu` | `192.168.1.32` |
| `cloud.yoleer.eu` | `192.168.1.32` |
| `crafty.yoleer.eu` | `192.168.1.32` |
| `jelly.yoleer.eu` | `192.168.1.32` |
| `seerr.yoleer.eu` | `192.168.1.32` |
| `wizarr.yoleer.eu` | `192.168.1.32` |
| `mc.yoleer.eu` | `192.168.1.34` |
| `vault.yoleer.eu` | `192.168.1.32` |