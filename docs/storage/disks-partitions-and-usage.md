# Drives, Partitions & Disk Usage

## Show drives

```bash
# method #1
lsblk

# method #2
df -h
```

## Create Partitions

```bash
parted dev/yourpartition

# create 3 partitions. The last one will as big as much space is available
mkpart primary 1MiB 513MiB
mkpart primary 513MiB 4609MiB
mkpart primary 4609MiB 100%
```

## Disk Space

### Check what files are using your disk space

```bash
sudo dnf install ncdu
sudo apt install ncdu

# check your full disk
sudo ncdu /
```