# Screen Lock

## Lock Screen and then sleep after

- Should work with every linux

```bash
xdg-screensaver lock
sleep 5.0
systemctl suspend
```

## Add custom stuff to screen lock like sounds or commands

- Should work with every linux

```text
1. Go to System Settings → Application and System Notifications → Manage Notifications.
2. Select Screen Saver as the Event Source
3. Locate and select the Screen Locked
```