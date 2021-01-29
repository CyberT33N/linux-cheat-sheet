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












<br><br>
____________________________________________
____________________________________________
<br><br>

# Disk Space

<br><br>

## Check what files are using your disk space
```bash
sudo dnf install ncdu
sudo apt install ncdu

# check your full disk
sudo ncdu /
```



































<br><br>
____________________________________________
____________________________________________
<br><br>

# IP

<br><br>

## Get adapter ip adresses
```bash
ifconfig
```




























<br><br>
____________________________________________
____________________________________________
<br><br>

# .sh scripts

<br><br>

## Layout
- When you use **#!/bin/bash** you can execute .sh files without **bash script.sh**! So you can use directly **script.sh**
```bash
#!/bin/bash
mkdir gitlab
cd gitlab
```






















<br><br>
____________________________________________
____________________________________________
<br><br>

# extract archives

<br><br>

## zip
```bash
unzip yourfile.zip
```

<br><br>

## tar.gz (https://linuxize.com/post/how-to-extract-unzip-tar-gz-file/)
```bash
tar -xf yourfile.tar.gz
```
























<br><br>
____________________________________________
____________________________________________
<br><br>

# Ubuntu

<br><br>


## benchmark
- https://www.youtube.com/watch?v=dD4-8NVonVM


<br><br>


## Install .deb files
```bash
# will also install all needed dependencies
sudo apt install *****.deb

# will only install the file
sudo dpkg -i ****.deb
```

<br><br>

## Install .run files
```bash
sudo chmod +x /path/to/file.run
sudo /path/to/file.run
```

















<br><br>

## PPA (Personal Package Archive)

<br><br>

#### remove PPA
```bash
sudo add-apt-repository --remove ppa:whatever/ppa

# As example
sudo add-apt-repository --remove http://ppa.launchpad.net/michael-gruz/canon-trunk/ubuntu
```






































