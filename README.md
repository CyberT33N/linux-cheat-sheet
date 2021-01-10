# linux-cheat-sheet
Linux Cheat Sheet with the most needed stuff..


<br><br>

# Show drives
```bash
# method #1
lsblk

# method #2
df -h

```

<br><br>

# Create Partitions
```bash
parted dev/yourpartition

# create 3 partitions. The last one will as big as much space is available
mkpart primary 1MiB 513MiB
mkpart primary 513MiB 4609MiB
mkpart primary 4609MiB 100%
```
