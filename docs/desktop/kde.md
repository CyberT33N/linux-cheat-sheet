# KDE Plasma

## Logout

```bash
loginctl terminate-user <username>
```

## How to disable "copy on select"?

```text
Go to taskbar down right and make right click on clipboard > Configure Clipboard > Ignore selection
```

## Broken login screen

```shell
loginctl unlock-session 4
```

## Restart plasma

```shell
kquitapp5 plasmashell
kstart5 plasmashell

# Restart plasma from tty
# Switch to tty2 by Press CTRL + ALT + F2 and hohld all buttons
systemctl restart sddm.service
# Then switch back to tty by Press CTRL + ALT + F1
```

## Taskbar not showing

- Right Click on desktop > Add Panel > Default Panel

## KDE Plasma Dolphin Bookmarks

```bash
/home/<USER>/.local/share/user-places.xbel
```