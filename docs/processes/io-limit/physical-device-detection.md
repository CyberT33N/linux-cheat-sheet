# I/O Limit For An Application Via Systemd User Service — Find The Physical Block Device

Part of the systemd I/O-limiting guide; see also [service configuration](./systemd-service-configuration.md) and [activation and operations](./activation-and-operations.md).

## 1. Standard: find the physical block device

If you have no encrypted partition, find the device of your app like this:

```bash
# Adjust the path to the executable file
APP_PATH="/path/to/the/app"

# Determine the device
DEVICE=$(df "$APP_PATH" | tail -1 | awk '{print $1}')

# Determine the Major:Minor ID
DEVICE_ID=$(lsblk -no MAJ:MIN "$DEVICE")

echo "Physical device: $DEVICE_ID"
```

## 2. For LUKS/dm-crypt encrypted partitions (e.g. /dev/mapper/cryptdata)

Because systemd I/O limits only act on physical devices, you have to find the **underlying real block device**.

```bash
# First determine the mount device for your app
APP_PATH="/path/to/the/app"
MOUNT_DEVICE=$(df "$APP_PATH" | tail -1 | awk '{print $1}')

# That could be /dev/mapper/cryptdata – find the physical device beneath it
lsblk -o NAME,MAJ:MIN,TYPE,MOUNTPOINT -r | grep "$MOUNT_DEVICE" -A 5
```

You get a tree structure. Example:

```text
nvme0n1p3 249:3 part
cryptdata 242:0 crypt /home
```

**Important:**

* `cryptdata` is virtual (dm-crypt).
* The underlying physical device is `nvme0n` with Major\:Minor `249:3`.

So always use the **lower physical device (partition)**, not the virtual one.