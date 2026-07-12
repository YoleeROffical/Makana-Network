# Stella

Stella is the primary services and storage-oriented Proxmox node in the Makana Network cluster.

## System role

| Item | Value |
|---|---|
| Hostname | `Stella` |
| Proxmox address | `192.168.1.11` |
| CPU | Intel Core i5-4590 |
| Memory | 12 GB DDR3 |
| Storage | System SSD and 2 TB media HDD |
| Main roles | Networking, primary services, torrent automation, media storage |
| Cluster | `prox-cluster` |
| Partner node | James |

## Workloads

| VMID | Name | Purpose |
|---:|---|---|
| 100 | Networking | NGINX Proxy Manager, Pi-hole, Unbound, CrowdSec, ddclient |
| 101 | Services | Nextcloud, Vaultwarden, Homepage, Portainer |
| 103 | Torrenting | qBittorrent, Sonarr, Radarr, Bazarr, Prowlarr, Profilarr |

## Storage

Stella currently hosts the main media storage. The storage is mounted into the containers that need it, including the Media LXC currently running on James.

Typical shared paths include:

```text
/media/content
/media/rips
```

The long-term plan is to move primary storage responsibilities to James after the HBA and SAS storage are deployed.

## Cluster role

Stella is the preferred node for the Networking LXC because of its current network path. James remains the secondary target for planned maintenance migration.

## Related documentation

Service-level documentation remains organized by VMID in the repository root:

- [`../../100 (Networking)/`](../../100%20(Networking)/)
- [`../../101 (Services)/`](../../101%20(Services)/)
- [`../../103 (Torrenting)/`](../../103%20(Torrenting)/)
