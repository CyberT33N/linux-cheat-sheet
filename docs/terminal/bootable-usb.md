# Bootable USB

## Ubuntu/Kubuntu

### Terminal

```shell
# You can find the partion name via partion manager
sudo dd if=input.iso of=/dev/sdx
```

### GUI

- [Video guide](https://www.youtube.com/watch?v=PurlSJCCuQQ)

1. Open Disks
2. Choose USB
3. Click Settings Icon -> Restore Partition Image
4. Choose .iso file
5. Click "Start Restoring"
6. Click "Restore"