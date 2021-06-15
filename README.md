# linux-cheat-sheet
Linux Cheat Sheet with the most needed stuff..






















<br><br>
<br><br>

_________________________________________________________________


<br><br>
<br><br>

# KDE

<br><br>

## KDE Plasma Dolphin Bookmarks
```bash
/home/<USER>/.local/share/user-places.xbel
```






















































<br><br>
<br><br>

_________________________________________________________________


<br><br>
<br><br>

# Burning CD/DVD
- https://docs.fedoraproject.org/en-US/Fedora/20/html-single/Burning_ISO_images_to_disc/index.html

<br><br>

## GUI

<br><br>

#### Brasero
- ISO to DVD
  - Launch Brasero.
  - Click Burn image.
  - Click Click here to select a disc image and browse to the ISO image file you downloaded.
  - Insert a blank disc, then click the Burn button. Brasero burns the image file to the disc.
```bash
# fedora
dnf install -y install brasero
```





























































<br><br>
<br><br>

_________________________________________________________________


<br><br>
<br><br>

# Terminal

<br><br>

## Open script in new terminal window and do not close it when its done
```bash
xterm -e "bash /home/username/Projects/start.sh;bash"
```














































































<br><br>
<br><br>

_________________________________________________________________


<br><br>
<br><br>

# cd


## Go to parrent directory
```bash
cd ..
```

<br><br>

## Go to last directory
```bash
cd -
```


<br><br>

## Go to home
```bash
cd
```


<br><br>

## Change to current directory of folder where the script file located
```bash
cd "$(dirname "$0")"
pwd
printf "\nWe will display now the current directory used:"
echo "$(dirname "$0")"
```















































































<br><br>
<br><br>

_________________________________________________________________


<br><br>
<br><br>

# cat


## combine content of 2 files to new file
```bash
cat test1.txt test2.txt > test3.txt
```













































































<br><br>
<br><br>

_________________________________________________________________


<br><br>
<br><br>

# pipe
- Pipe the results of any command to a new file
```bash
ls > result.txt
```

## add new text to file
```bash
echo hi >> result.txt
```









































<br><br>
<br><br>

_________________________________________________________________


<br><br>
<br><br>

# Files


## Check all files in current directory
```bash
ls

# show hidden files
la -a

# detailed informations
ls -al
```
























































<br><br>
<br><br>

_________________________________________________________________


<br><br>
<br><br>


Change Ownership
```bash
# -R means recursive
sudo chown -R username folder
```



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

# source
- source is a shell built-in command which is used to read and execute the content of a file(generally set of commands), passed as an argument in the current shell script. The command after taking the content of the specified files passes it to the TCL interpreter as a text script which then gets executed. If any arguments are supplied, they become the positional parameters when filename is executed. Otherwise, the positional parameters remain unchanged. The entries in $PATH are used to find the directory containing FILENAME, however if the file is not present in $PATH it will search the file in the current directory. The source command has no option and the argument is the file only. 
```bash
# coolscript.sh
NAME="LENA"

# otherscript.sh
source coolscript.sh
echo $NAME
```







































<br><br>
____________________________________________
____________________________________________
<br><br>

# Processes

<br><br>

## kill process
```bash
# To kill process in Linux with PID:
kill pid

# To kill process in Linux with application name:
killall appname
```

<br><br>

## force kill process
```bash
# To kill process in Linux with PID:
kill -9 pid

# To kill process in Linux with application name:
killall -9 appname
```


<br><br>

## kill all processes on specific port
```bash
sudo kill -9 $(sudo lsof -t -i:30500)
```















































































<br><br>
____________________________________________
____________________________________________
<br><br>

# Network

<br><br>

## Check what processes uses the specific port
```bash
lsof -i :30500
```












dejs/












<br><br>
____________________________________________
____________________________________________
<br><br>

# Symlink
- Create a link to another file to use it.
```bash
ln -s ../project/.eslintrc.json ./.eslintrc.json
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

# System

<br><br>

## Logs


<br><br>

#### check logs realtime
```bash
sudo journalctl -f
```


<br><br>

#### get system logs to find problems as example when system crash
```bash
# -1 last restart | -2 last restart before 2 days
journalctl -o short-precise -k -b -2

# Search logs that only contains "error"
journalctl -o short-precise -k -b -2 | grep -i "error"
```



















































<br><br>
____________________________________________
____________________________________________
<br><br>


# archives

## create archives
```bash
# recursive add all sub folders and files
zip -r archivename.zip directory_name

# recursive add all sub folders and files in current directory
zip -r archivename.zip .

# zip multiple directories into individual zip files
for i in */; do zip -r "${i%/}.zip" "$i"; done
```



<br><br>

## extract archives

<br><br>

#### zip
```bash
unzip yourfile.zip
```

<br><br>

#### tar.gz (https://linuxize.com/post/how-to-extract-unzip-tar-gz-file/)
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



## Known Problems

<br><br>

#### Random freezes
- system settings/ Display and Monitor/ Compositor switching from OpenGL to XRender seems to work.

<br><br>

#### Black screen after installation (https://askubuntu.com/questions/1085807/black-screen-after-installation-of-ubuntu-18-04)
- mediately after the BIOS/UEFI splash screen during boot, with BIOS, quickly press and hold the Shift key, which will bring up a GNU GRUB menu screen. With UEFI press (perhaps several times) the Esc key to get to the GNU GRUB menu screen. Sometimes the manufacturer's splash screen is a part of the Windows bootloader, so when you power up the machine it goes straight to the GNU GRUB menu screen, and then pressing Shift is unnecessary.
  - Press e to enter editing mode
    - Immediately after this string replace ro quiet splash by nomodeset quiet splash. This change is only temporary — it will just be used once and GRUB won't remember it in the future. Press Ctrl+X or F10 to boot with the nomodeset option that was added. If you make a mistake, press Esc to go back to the previous screen.

<br><br>

      - How to permanently set kernel boot options on an installed OS?
        ```bash
        sudo gedit /etc/default/grub
        GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nomodeset"
        sudo update-grub
        ```







<br><br><br><br>


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




















































<br><br>
____________________________________________
____________________________________________
<br><br>

# Fedora

<br><br>

## show if package with specific name exist
```bash
sudo dnf list | grep "dragondisk"
```


<br><br>

## ~/bin
- In this folder everything is global. This means you can use everything global in your terminal


























































































<br><br>
____________________________________________
____________________________________________
<br><br>

# cronjob

<br><br>

## list all cronjobs
```bash
crontab -l
```


<br><br>

## create new cronjob
```bash
crontab -e
1 * * * * /home/usernamehere/Projects/start.sh
```

