# `arm-rip-sr0-host.sh`

Host-side launcher for the automated optical-media workflow.

The launcher is started by `arm-rip-sr0.service`, rather than being executed directly by `udev`. This prevents `udev` from terminating the long-running rip process before cleanup can complete.

## Install location

```text
/usr/local/bin/arm-rip-sr0-host.sh
```

## Script

```bash
#!/bin/bash
set -u

LOG=/tmp/arm-udev-host.log
LOCK=/run/lock/makana-rip.lock

exec >>"$LOG" 2>&1

echo "$(date -Is) Automatic rip trigger received"

# Open the lock file on file descriptor 9.
exec 9>"$LOCK"

# The kernel automatically releases this lock when the process exits,
# including abnormal termination.
if ! flock -n 9; then
    echo "$(date -Is) Another rip is already running, exiting"
    exit 0
fi

echo "$(date -Is) Rip lock acquired"

# Give the optical drive and udev database time to settle.
sleep 30

if [[ ! -b /dev/sr0 ]]; then
    echo "$(date -Is) ERROR: /dev/sr0 does not exist"
    exit 1
fi

# Ignore tray-open and media-removal change events.
if ! udevadm info --query=property --name=/dev/sr0 2>/dev/null \
    | grep -q '^ID_CDROM_MEDIA=1$'; then
    echo "$(date -Is) No optical media detected; ignoring event"
    exit 0
fi

if ! pct status 102 | grep -q '^status: running$'; then
    echo "$(date -Is) CT 102 is stopped; starting it"

    if ! pct start 102; then
        echo "$(date -Is) ERROR: Could not start CT 102"
        exit 1
    fi
fi

echo "$(date -Is) Starting worker inside CT 102"

pct exec 102 -- /usr/local/bin/arm-rip-transcode.sh
RESULT=$?

echo "$(date -Is) Automatic rip finished with status $RESULT"
exit "$RESULT"
```

## Installation

Run on James:

```bash
install -m 0755 /dev/stdin /usr/local/bin/arm-rip-sr0-host.sh <<'EOF'
#!/bin/bash
set -u

LOG=/tmp/arm-udev-host.log
LOCK=/run/lock/makana-rip.lock

exec >>"$LOG" 2>&1

echo "$(date -Is) Automatic rip trigger received"

exec 9>"$LOCK"

if ! flock -n 9; then
    echo "$(date -Is) Another rip is already running, exiting"
    exit 0
fi

echo "$(date -Is) Rip lock acquired"

sleep 30

if [[ ! -b /dev/sr0 ]]; then
    echo "$(date -Is) ERROR: /dev/sr0 does not exist"
    exit 1
fi

if ! udevadm info --query=property --name=/dev/sr0 2>/dev/null \
    | grep -q '^ID_CDROM_MEDIA=1$'; then
    echo "$(date -Is) No optical media detected; ignoring event"
    exit 0
fi

if ! pct status 102 | grep -q '^status: running$'; then
    echo "$(date -Is) CT 102 is stopped; starting it"

    if ! pct start 102; then
        echo "$(date -Is) ERROR: Could not start CT 102"
        exit 1
    fi
fi

echo "$(date -Is) Starting worker inside CT 102"

pct exec 102 -- /usr/local/bin/arm-rip-transcode.sh
RESULT=$?

echo "$(date -Is) Automatic rip finished with status $RESULT"
exit "$RESULT"
EOF

bash -n /usr/local/bin/arm-rip-sr0-host.sh
```

A successful `bash -n` check prints no output.

## Lock behavior

The old implementation created `/tmp/makana-rip.lock` as a directory and removed it through a Bash `EXIT` trap. Because the script was launched directly by `udev`, it could be terminated before the trap executed, leaving a stale lock behind.

The current implementation uses:

```text
/run/lock/makana-rip.lock
```

The file may remain after a job finishes. That is normal. The file's existence does not indicate that a rip is active; only the kernel-managed `flock` matters.

Check the lock:

```bash
flock -n /run/lock/makana-rip.lock \
  -c 'echo "Lock is free"' \
  || echo "Lock is currently held"
```

## Logs

Host launcher:

```bash
tail -f /tmp/arm-udev-host.log
```

Detailed worker log:

```bash
pct exec 102 -- tail -f /media/rips/logs/automatic-rip.log
```
