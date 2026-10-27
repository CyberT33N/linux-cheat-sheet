# Archives

## Create archives

```bash
# recursive add all sub folders and files
zip -r archivename.zip directory_name

# recursive add all sub folders and files in current directory
zip -r archivename.zip .

# zip multiple directories into individual zip files
for i in */; do zip -r "${i%/}.zip" "$i"; done
```

## Extract archives

### Zip

```bash
unzip yourfile.zip
```

### Tar.gz

- [How to extract (unzip) a .tar.gz file (linuxize)](https://linuxize.com/post/how-to-extract-unzip-tar-gz-file/)

```bash
tar -xf yourfile.tar.gz
```