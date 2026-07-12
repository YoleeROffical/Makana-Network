# James

James is the primary performance-focused Proxmox node in the Makana Network cluster.

## System role

| Item | Value |
|---|---|
| Hostname | `james` |
| Proxmox address | `192.168.1.10` |
| CPU | Intel Core i7-6700 |
| Memory | 32 GB DDR4 |
| Main roles | Media processing, game servers, Docker workloads, future storage |
| Cluster | `prox-cluster` |
| Partner node | Stella |

## Workloads

| VMID | Name | Purpose |
|---:|---|---|
| 100 | Networking | The Networking LXC only moves to james incase Stella's status is offline/maintenance |
| 102 | Media | Jellyfin, ARM, media-processing services |
| 104 | Game-Servers | Crafty Controller and Minecraft servers |
| 105 | docker | Testing and sandbox workloads |

## Host-specific documentation

- [`ARM/README.md`](ARM/README.md) — automated optical-media workflow
- [`ARM/arm-rip-sr0-host.md`](ARM/arm-rip-sr0-host.md) — host-side trigger script
- [`ARM/udev-rule.md`](ARM/udev-rule.md) — optical-disc insertion rule
- [`ARM/lxc-gpu-passthrough.md`](ARM/lxc-gpu-passthrough.md) — optical drive and Intel iGPU passthrough into CT 102

## Notes

James detects optical-disc insertion with `udev`, waits for the drive to settle, and then invokes the worker script inside CT 102 with `pct exec`.

The Media LXC performs the actual MakeMKV rip, TMDb naming, Intel VA-API transcode, and final placement into the media library.
