# `arm-rip-sr0.service`

systemd oneshot service used to run the ARM host launcher outside the restricted `udev` execution environment.

## Install location

```text
/etc/systemd/system/arm-rip-sr0.service
```

## Unit

```ini
[Unit]
Description=Makana automatic optical-disc ripping
ConditionPathExists=/dev/sr0

[Service]
Type=oneshot
ExecStart=/usr/local/bin/arm-rip-sr0-host.sh
TimeoutStartSec=infinity
```

## Installation

Run on James:

```bash
cat >/etc/systemd/system/arm-rip-sr0.service <<'EOF'
[Unit]
Description=Makana automatic optical-disc ripping
ConditionPathExists=/dev/sr0

[Service]
Type=oneshot
ExecStart=/usr/local/bin/arm-rip-sr0-host.sh
TimeoutStartSec=infinity
EOF

systemctl daemon-reload
systemd-analyze verify /etc/systemd/system/arm-rip-sr0.service
```

The service is static and should not be enabled. It is started by the optical-drive `udev` rule.

An idle service should show:

```text
Active: inactive (dead)
```

## Manual test

```bash
systemctl start --no-block arm-rip-sr0.service
systemctl status arm-rip-sr0.service --no-pager
```

Follow the host log:

```bash
tail -f /tmp/arm-udev-host.log
```

## Why systemd is required

`udev` rules should perform only short actions. The original rule launched the complete rip workflow directly, including a 30-second delay and a potentially hours-long `pct exec` operation.

Moving the workflow into systemd gives it a normal service lifetime and allows `flock` to be released reliably when the process exits.
