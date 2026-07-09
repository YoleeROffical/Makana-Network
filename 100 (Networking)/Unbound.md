# Unbound Setup

Installed bare metal inside the **Networking LXC** on Stella (`192.168.1.32`), running alongside Pi-hole.

Unbound acts as a recursive DNS resolver sitting behind Pi-hole. Instead of forwarding queries to a third-party upstream like Cloudflare or Google, Unbound resolves them directly by querying the root DNS servers itself — no upstream provider sees the queries.

---

## Installation

```bash
sudo apt install unbound -y
```

---

## Configuration

Created the config file at `/etc/unbound/unbound.conf.d/pi-hole.conf`:

```conf
server:
    verbosity: 1
    interface: 127.0.0.1
    port: 5335
    do-ip4: yes
    do-udp: yes
    do-tcp: yes
    do-ip6: no

    # Proxmox LXC Bypass Fixes
    chroot: ""
    username: ""

    # Use Debian's static package-managed trust anchor
#    trust-anchor-file: "/usr/share/dns/root.key"

    # Buffer size for UDP receptors
    so-rcvbuf: 4m

    # Ensure privacy of local members
    private-address: 192.168.0.0/16
    private-address: 169.254.0.0/16
    private-address: 172.16.0.0/12
    private-address: 10.0.0.0/8
    private-address: fd00::/8
    private-address: fe80::/10
```

Unbound listens on `127.0.0.1:5335` — only accessible locally, Pi-hole forwards to it.

---

## Linking to Pi-hole

In Pi-hole's settings, upstream DNS was set to:

```
Custom 1: 127.0.0.1#5335
```

This routes all Pi-hole upstream queries to Unbound instead of any public DNS provider.

---

## Service

```bash
sudo systemctl enable unbound
sudo systemctl start unbound
```

Verify it's resolving correctly:

```bash
dig google.com @127.0.0.1 -p 5335
```

Should return a valid answer with `status: NOERROR`.

---

## Notes

- Unbound only listens on localhost — it's not exposed to the network directly, only Pi-hole talks to it
- `do-ip6: no` is set since the home network doesn't use IPv6 internally
- `prefetch: yes` improves performance by refreshing popular DNS records before they expire