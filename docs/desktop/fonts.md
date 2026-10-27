# Fonts

## Install

- You can download any font and isntall it via right click on it and then install or you can do it like this in bulk:

```shell
wget -O /tmp/myfont.ttf https://example.com/path/to/font.ttf
mkdir -p ~/.local/share/fonts
cp /tmp/myfont.ttf ~/.local/share/fonts/
fc-cache -fv
```