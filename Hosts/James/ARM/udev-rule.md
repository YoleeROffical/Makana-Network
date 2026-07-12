# Optical-disc udev rule

This rule starts the ARM host launcher when Linux reports a media-change event for `/dev/sr0`.

## Install location

```text
/etc/udev/rules.d/99-arm-sr0.rules
```

## Rule

```udev
ACTION=="change", KERNEL=="sr0", RUN+="/usr/local/bin/arm-rip-sr0-host.sh"
```

## Installation

Run on James:

```bash
cat >/etc/udev/rules.d/99-arm-sr0.rules <<'EOF'
ACTION=="change", KERNEL=="sr0", RUN+="/usr/local/bin/arm-rip-sr0-host.sh"
EOF

udevadm control --reload-rules
```

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

A single insertion can generate multiple `change` events. The host script's atomic lock prevents duplicate rip jobs.

The rule intentionally does not depend on `ID_CDROM_MEDIA=1`, because early optical-drive events may not include that property. The 30-second delay in the launcher gives the drive time to become ready.
