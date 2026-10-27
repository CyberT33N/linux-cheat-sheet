# I/O Limit For An Application Via Systemd User Service — Service Configuration

Part of the systemd I/O-limiting guide; see also [physical device detection](./physical-device-detection.md) and [activation and operations](./activation-and-operations.md).

## 3. Create the systemd user service file

Path:
`~/.config/systemd/user/cursor-throttled.service`

Content:

```ini
[Unit]
Description=Cursor editor with I/O limit and low priority
# Description of the service as it appears in the system.

[Service]
ExecStart=/home/userName/Applications/cursor/squashfs-root/usr/bin/cursor
# Starts the Cursor application.

IOReadBandwidthMax=249:3 115M
# Maximum read bandwidth on the physical device (Major:Minor 249:3).
# '1M' = 1 megabyte per second.
# The smaller the value, the stricter the throttling.
# Highest value = no limitation (omit the value).
# Lowest value = minimal allowed bandwidth (~1 byte).

IOWriteBandwidthMax=249:3 115M
# Same logic for the write bandwidth.

# Nice=-20
# CPU priority (nice value) from -20 (highest priority) to +19 (lowest priority).
# The highest priority value you can set is +19 → the process only runs when the CPU is completely free.
# The lowest value is -20 → the process runs preferentially and can take CPU from other processes.
# For resource conservation always set it as high as possible, i.e. +19.

# IOSchedulingClass=2
# I/O scheduler classes:
# 1 = Realtime – highest I/O priority
# 2 = Best Effort – default
# 3 = Idle – lowest priority, I/O only when nothing else is happening.
# For throttling this value must be set to 3 (Idle).

# IOSchedulingPriority=2
# Priority within the scheduler class:
# Only valid for classes 1 (Realtime) and 2 (Best Effort).
# Values from 0 (highest priority) to 7 (lowest priority).
# For class 3 (Idle) this value is ignored, but setting 7 does no harm.

Restart=on-failure
# Automatic restart of the process on crash.

# Further sensible options (not mandatory, but recommended):

# CPUQuota=20%
# Limit CPU utilization to 20% (alternative to Nice).
# Useful if you want absolute CPU limits in addition to priority.

# MemoryMax=500M
# Maximum RAM usage, here e.g. 500 megabytes.
# Helps with memory leaks and protects the system.

[Install]
WantedBy=default.target
# Starts the service automatically with the user session.
```