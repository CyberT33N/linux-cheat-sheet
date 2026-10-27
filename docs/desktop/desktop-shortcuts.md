# Desktop Custom Shortcuts

## Create custom shortcut with termainl command

```shell
touch ~/Desktop/iron.desktop

sudo nano ~/Desktop/iron.desktop

#!/usr/bin/env xdg-open
[Desktop Entry]
Name=Iron [Sandboxed]
Exec=/bin/bash -c "firejail /usr/share/iron/chrome"
Type=Application
Terminal=true
Icon=/home/t33n/Downloads/chromium-logo.png
```