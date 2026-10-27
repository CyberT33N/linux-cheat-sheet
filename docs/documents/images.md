# Images

## Change quality

```shell
sudo apt install imagemagick
# cd into diretory with the images
mogrify -quality 75% *
```

## Resize image

```shell
sudo apt install imagemagick
# cd into diretory with the images
mogrify -resize 50% -format jpg *.png

# To specifc size - This will stretch the image
mogrify -resize 1024x1024! *

# Without stretching
mogrify -resize 1024x1024 -background white -gravity center -extent 1024x1024 *
```

## Convert

### Png to webp

```shell
sudo apt install webp
cwebp -q 60 Cute-Baby-Girl.png -o Cute-Baby-Girl.webp
```