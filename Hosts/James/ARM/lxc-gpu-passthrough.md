# CT 102 optical-drive and Intel iGPU passthrough

The Media LXC is unprivileged. It needs access to:

- `/dev/sr0` — optical block device
- `/dev/sg0` — SCSI generic device used by MakeMKV
- `/dev/dri/renderD128` — Intel render node used for VA-API encoding

## Host device ownership

Check on James:

```bash
ls -ln /dev/dri
ls -ln /dev/sr0 /dev/sg0
```

The render node in this setup uses host GID `993`.

## Subordinate group mapping

Add the render GID to `/etc/subgid` on James:

```bash
grep -qxF 'root:993:1' /etc/subgid || echo 'root:993:1' >> /etc/subgid
```

## CT 102 configuration

Stop the container before changing ID mappings:

```bash
pct stop 102
cp /etc/pve/lxc/102.conf /root/102.conf.backup
```

Relevant entries in `/etc/pve/lxc/102.conf`:

```ini
# Optical drive
dev0: /dev/sg0
dev1: /dev/sr0

# Intel DRM render node
lxc.cgroup2.devices.allow: c 226:128 rwm
lxc.mount.entry: /dev/dri dev/dri none bind,optional,create=dir

# Unprivileged LXC ID mappings.
# Container GID 993 maps directly to host GID 993.
lxc.idmap: u 0 100000 65536
lxc.idmap: g 0 100000 993
lxc.idmap: g 993 993 1
lxc.idmap: g 994 100994 64542
```

Start and verify:

```bash
pct start 102
pct status 102
pct exec 102 -- ls -ln /dev/dri
```

Expected render node:

```text
crw-rw---- ... 65534 993 ... renderD128
```

## Media-LXC group

Inside CT 102, GID `993` is currently named `kvm`:

```bash
getent group 993
```

Root was added to that group during testing:

```bash
usermod -aG kvm root
```

A direct VA-API test inside the LXC:

```bash
sg kvm -c 'LIBVA_DRIVER_NAME=i965 vainfo --display drm --device /dev/dri/renderD128'
```

Both `i965` and `iHD` can initialize on the Skylake iGPU, but the final application choices differ:

| Application | Driver | Reason |
|---|---|---|
| ARM FFmpeg pipeline | `iHD` | H.264 VA-API low-power CQP works with `-low_power 1` |
| Jellyfin | `i965` | Supports Jellyfin's VBR H.264 VA-API transcode command |

## Docker device mapping

ARM uses the render node directly:

```yaml
devices:
  - /dev/sr0:/dev/sr0
  - /dev/sg0:/dev/sg0
  - /dev/dri/renderD128:/dev/dri/renderD128

group_add:
  - "993"
```

Jellyfin maps all DRM nodes:

```yaml
devices:
  - /dev/dri:/dev/dri
```

The LinuxServer Jellyfin container also mounts a custom init script that adds its internal `abc` user to GID `993`.

## Recovery from a bad LXC mapping

If CT 102 fails to start after editing `lxc.idmap`, restore the backup:

```bash
cp /root/102.conf.backup /etc/pve/lxc/102.conf
```

Remove duplicate GUI-created DRM device entries such as:

```ini
dev2: /dev/dri/card1
dev3: /dev/dri/renderD128
```

when `/dev/dri` is already bind-mounted through `lxc.mount.entry`.
