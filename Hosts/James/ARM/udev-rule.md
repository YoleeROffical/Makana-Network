# Optical-disc udev rule

This rule starts a short-lived systemd request when Linux reports a media-change event for `/dev/sr0`.

`udev` does not run the ripping script directly. Long-running commands launched directly from `udev` may be terminated by the device manager, which previously caused stale lock directories.

## Install location

```text
/etc/udev/rules.d/99-arm-sr0.rules
```

## Rule

```udev
ACTION=="change", KERNEL=="sr0", RUN+="/usr/bin/systemctl --no-block start arm-rip-sr0.service"
```

## Installation

Run on James:

```bash
cat >/etc/udev/rules.d/99-arm-sr0.rules <<'EOF'
ACTION=="change", KERNEL=="sr0", RUN+="/usr/bin/systemctl --no-block start arm-rip-sr0.service"
EOF

udevadm control --reload-rules
```

Do not run `udevadm trigger` while a disc is inserted, because it may start a rip immediately.

## Verify events

```bash
udevadm monitor --udev --property
```

Insert a disc and look for values similar to:

```text
ACTION=change
DEVNAME=/dev/sr0
ID_CDROM_MEDIA=1
ID_CDROM_MEDIA_DVD=1
ID_CDROM_MEDIA_STATE=complete
```

## Notes

A single insertion can generate multiple `change` events. systemd will not start a second instance while the oneshot service is active, and the host launcher also uses a non-blocking `flock`.

The rule intentionally does not require `ID_CDROM_MEDIA=1`, because early optical-drive events may omit that property. The launcher waits 30 seconds and then verifies that media is actually present.
