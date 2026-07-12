# `arm-rip-sr0-host.sh`

Host-side launcher for the automated optical-media workflow.

## Install location

```text
/usr/local/bin/arm-rip-sr0-host.sh
```

## Script

```bash
#!/bin/bash
set -u

LOCK=/tmp/makana-rip.lock
LOG=/tmp/arm-udev-host.log

if ! mkdir "$LOCK" 2>/dev/null; then
    echo "$(date) Rip already running, exiting" >>"$LOG"
    exit 0
fi

trap 'rmdir "$LOCK" 2>/dev/null || true' EXIT

echo "$(date) Automatic rip trigger fired" >>"$LOG"

# Give the optical drive time to finish detecting the disc.
sleep 30

pct exec 102 -- /usr/local/bin/arm-rip-transcode.sh >>"$LOG" 2>&1
RESULT=$?

echo "$(date) Automatic rip finished with status $RESULT" >>"$LOG"
exit "$RESULT"
```

## Installation

Run on James:

```bash
cat >/usr/local/bin/arm-rip-sr0-host.sh <<'EOF'
#!/bin/bash
set -u

LOCK=/tmp/makana-rip.lock
LOG=/tmp/arm-udev-host.log

if ! mkdir "$LOCK" 2>/dev/null; then
    echo "$(date) Rip already running, exiting" >>"$LOG"
    exit 0
fi

trap 'rmdir "$LOCK" 2>/dev/null || true' EXIT

echo "$(date) Automatic rip trigger fired" >>"$LOG"

# Give the optical drive time to finish detecting the disc.
sleep 30

pct exec 102 -- /usr/local/bin/arm-rip-transcode.sh >>"$LOG" 2>&1
RESULT=$?

echo "$(date) Automatic rip finished with status $RESULT" >>"$LOG"
exit "$RESULT"
EOF

chmod 755 /usr/local/bin/arm-rip-sr0-host.sh
bash -n /usr/local/bin/arm-rip-sr0-host.sh
```

A successful `bash -n` check prints no output.

## Design

The lock uses `mkdir` because directory creation is atomic. Multiple `udev` change events can occur for one disc insertion; only the first invocation obtains the lock.

The worker script lives inside CT 102 at:

```text
/usr/local/bin/arm-rip-transcode.sh
```

The host script does not rip or transcode directly.
