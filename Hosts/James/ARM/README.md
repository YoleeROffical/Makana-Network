# Automated Ripping Machine — James host

This directory documents the Proxmox-host side of the automated optical-media workflow.

## Workflow

```text
Disc inserted into /dev/sr0
        |
        v
udev change event
        |
        v
systemctl --no-block start arm-rip-sr0.service
        |
        v
/usr/local/bin/arm-rip-sr0-host.sh
        |
        +--> acquire kernel-managed flock
        +--> wait for the drive to settle
        +--> verify optical media is present
        +--> start CT 102 when necessary
        `--> pct exec 102 -- /usr/local/bin/arm-rip-transcode.sh
```

## Documentation

- [`udev-rule.md`](udev-rule.md) — optical-disc event rule
- [`arm-rip-sr0-service.md`](arm-rip-sr0-service.md) — systemd oneshot service
- [`arm-rip-sr0-host.md`](arm-rip-sr0-host.md) — host-side launcher and locking
- [`lxc-gpu-passthrough.md`](lxc-gpu-passthrough.md) — optical drive and Intel iGPU passthrough into CT 102

## Logs

Host-side trigger and launcher:

```bash
tail -f /tmp/arm-udev-host.log
```

Detailed worker log inside CT 102:

```bash
pct exec 102 -- tail -f /media/rips/logs/automatic-rip.log
```

## Lock status

```bash
flock -n /run/lock/makana-rip.lock \
  -c 'echo "Lock is free"' \
  || echo "Lock is currently held"
```

The lock file may remain on disk after completion. This is expected. The kernel lock, rather than the file's existence, determines whether a job is active.

## Troubleshooting

### Media LXC fails with `/dev/sr0 does not exist`

Check detection on James:

```bash
ls -l /dev/sr* /dev/sg* 2>/dev/null
lsscsi -g
```

The configured optical drive currently appears as:

```text
/dev/sr0
/dev/sg0
```

If `/dev/sr0` is absent, check SATA power, the SATA data cable, and the motherboard SATA port before changing Proxmox or LXC configuration.

### A manual test returns status 11

The worker may return a non-zero status when the service is manually started with no readable disc. This does not indicate a locking failure. Confirm the next no-media event is ignored and verify that `flock` reports the lock as free.
