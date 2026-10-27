# Common Errors

## How to disable ksgrd_network_helper because of high cpu

- Disable columns Download & Upload from your KsysGuard Task Manager

## inotify cannot be used, reverting to polling: Too many open files / User limit of inotify instances reached or too many open files — inotify_add_watch -- failed: "No space left on device"

- [How to fix inotify cannot be used, reverting to polling: too many open files](https://medium.com/@ivanermilov/how-to-fix-inotify-cannot-be-used-reverting-to-polling-too-many-open-files-bb1c1437dbf)

```bash
# Method 1
sudo gedit /etc/sysctl.conf

# Add this to sysctl.conf
fs.inotify.max_user_watches=90000000
fs.inotify.max_user_instances = 25600000

# To take effect of changes
sudo sysctl -p
```

## The SUID sandbox helper binary was found, but is not configured correctly. Rather than run without sandboxing I'm aborting now. You need to make sure that /tmp/.mount_cursorkGhSPt/chrome-sandbox is owned by root and has mode 4755.

```shell
./cursor-0.41.2x86_64.AppImage --appimage-extract

sudo chown root:root squashfs-root/chrome-sandbox
sudo chmod 4755 squashfs-root/chrome-sandbox

./squashfs-root/AppRun
```