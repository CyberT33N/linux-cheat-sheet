# Unmount Busy VeraCrypt Volumes

## Unmount busy target

Force-unmount a single busy mount target:

```shell
sudo umount -f <mountpoint>
```

## Scripted: dismount all mounted VeraCrypt volumes

```shell
#!/bin/bash

# Find all mounted VeraCrypt volumes
veracrypt_volumes=$(mount | grep 'veracrypt' | awk '{print $1}')

# Check whether VeraCrypt volumes were found
if [ -z "$veracrypt_volumes" ]; then
    echo "No mounted VeraCrypt volumes found."
else
    # Dismount every found VeraCrypt volume
    for volume in $veracrypt_volumes; do
        echo "Trying to dismount $volume..."
        sudo umount -f "$volume"
        if [ $? -eq 0 ]; then
            echo "$volume dismounted successfully."
        else
            echo "Error while dismounting $volume."
        fi
    done
fi
```