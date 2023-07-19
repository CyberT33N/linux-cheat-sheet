# linux-cheat-sheet
Linux Cheat Sheet with the most needed stuff..






# Audio

<br><br>

## Sound not working anymore after sign-out
- Go to Pulse Audio Volume Control by search it via menu bar. Then change audio output for application. If this not work try killall pulseaudio and then sign-out again and try again to change audio output.

<br><br>

## Restart pulseaudio
```
pulseaudio -k && pulseaudio --start
```















<br><br>
<br><br>

___________________________________________________
___________________________________________________


<br><br>
<br><br>

## Bootable USB

<br><br>

### ubuntu/Kubuntu
- https://www.youtube.com/watch?v=PurlSJCCuQQ
1. Open Disks
2. Choose USB
3. Click Settings Icon -> Restore Partition Image
4. Choose .iso file
5. Click "Start Restoring"
6. Click "Restore"























<br><br>
<br><br>

___________________________________________________
___________________________________________________


<br><br>
<br><br>


# Reset User Password

<br><br>

## Ubuntu/Kubuntu

<br><br>

https://askubuntu.com/questions/24006/how-do-i-reset-a-lost-administrative-password












<br><br>
<br><br>

___________________________________________________
___________________________________________________


<br><br>
<br><br>

# Priority

<br><br>

## Set priority to process
```
# nice -%LEVEL% your-app
# Where %LEVEL% is a value of “niceness” from -19 (maximal priority, minimal niceness) to 20 (minimal priority). For “niceness” less than zero you have to use root rights. For -19 niceness use:

sudo nice --19 your-app
```




































<br><br>
<br><br>

___________________________________________________
___________________________________________________


<br><br>
<br><br>

# Copy

<br><br>

## Copy folder with progress bar
```
rsync -Pa source destination
```





































<br><br>
<br><br>

___________________________________________________
___________________________________________________





<br><br>
<br><br>


# KDE

<br><br>

## Logout
```bash
loginctl terminate-user <username>
```
<br><br>

## How to disable "copy on select"?
```bash
Go to taskbar down right and make right click on clipboard > Configure Clipboard > Ignore selection
```





<br><br>

## Taskbar not showing
- Right Click on desktop > Add Panel > Default Panel



















<br><br>
<br><br>

___________________________________________________
___________________________________________________





<br><br>
<br><br>


# Grep

<br><br>

To search a file for a text string, use the following command syntax:
- grep string filename

For example, let’s search our document.txt text document for the string “example.”
- grep example document.txt

<br><br>



















<br><br>
<br><br>


# Screen Lock

<br><br>


## Lock Screen and then sleep after
- Should work with every linux
```bash
xdg-screensaver lock
sleep 5.0
systemctl suspend
```

<br><br>



## Add custom stuff to screen lock like sounds or commands
- Should work with every linux
```bash
1. Go to System Settings → Application and System Notifications → Manage Notifications.
2. Select Screen Saver as the Event Source
3. Locate and select the Screen Locked
```

























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
xterm -geometry 93x31+100+350 -xrm 'XTerm.vt100.allowTitleOps: false' -T Minikube -hold -e $SHELL -c "bash ./setup-port-foward.sh" &

xterm -geometry 93x31+100+750 -xrm 'XTerm.vt100.allowTitleOps: false' -T Minikube -hold -e $SHELL -c "bash ./setup-port-foward.sh" &
```












































<br><br>
<br><br>

_________________________________________________________________


<br><br>
<br><br>

# BTRFS

<br><br>

## Disable btrfs-cleaner
```bash
sudo btrfs quota disable /
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
printf "\nWe will display now the current directory used:" && pwd
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


- Show existing symlinks
```bash
ls -la /path/here
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

#### check logs specific time
```bash
journalctl --since "2021-11-30 15:00" --until "2021-11-30 17:00" > ~/log.txt
```
<br><br>

#### check logs 10 Minutes ago
```bash
journalctl --since "10min ago" > ~/log.txt
```
<br><br>

#### check logs specific time from NetworkManager service
```bash
journalctl -u NetworkManager.service --since "2021-11-30 15:00" --until "2021-11-30 17:00" > ~/log.txt
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

## Themes
- https://itsfoss.com/install-themes-ubuntu/
- https://www.gnome-look.org
- https://www.pling.com/p/1238824/

```
mkdir ~/.themes
mkdir ~/.icons

sudo apt install gnome-tweaks gnome-shell-extensions

reboot
```

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
- If your content of the bin folder is not available in your terminal add to your .bashrc or .zshrc
```javascript
export PATH="$HOME/bin:$PATH"
```
























































































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






<br><br>

## remove cronjob
```bash
// Text editor inside of terminal will open. Here you can remove your cronjobs
crontab -e
```














































<br><br>
____________________________________________
____________________________________________
<br><br>

# Nvidia Driver
- You can find a list of nvidia driver here and check there if your graphiccard is supported
  - https://wiki.ubuntuusers.de/Grafikkarten/Nvidia/nvidia/


<br><br>

## Gtx 1050ti
```bash
sudo apt-get purge 'nvidia*'
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt-get update
sudo apt-get install nvidia-driver-525 nvidia-settings
sudo reboot
```



## Suspend not working anymore
- If you suspend and it directly wake up after it then try:
```
sudo systemctl stop nvidia-suspend.service
sudo systemctl stop nvidia-hibernate.service
sudo systemctl stop nvidia-resume.service

sudo systemctl disable nvidia-suspend.service
sudo systemctl disable nvidia-hibernate.service
sudo systemctl disable nvidia-resume.service

sudo rm /lib/systemd/system-sleep/nvidia
```




















<br><br>
____________________________________________
____________________________________________
<br><br>




# Appearance / Design

<br><br>

## Desktop

<br><br>

### Shortcut 

<br><br>

#### Create custom shortcut with termainl command
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















<br><br>
____________________________________________
____________________________________________
<br><br>

# FAQ / Error



<br><br>

## How to disable ksgrd_network_helper because of high cpu
- Disable columns Download & Upload from your KsysGuard Task Manager
 
 
 
<br><br>
 
## inotify cannot be used, reverting to polling: Too many open files / User limit of inotify instances reached or too many open files
## inotify_add_watch -- failed: "No space left on device"
- https://medium.com/@ivanermilov/how-to-fix-inotify-cannot-be-used-reverting-to-polling-too-many-open-files-bb1c1437dbf
```bash
# Method 1
sudo gedit /etc/sysctl.conf

# Add this to sysctl.conf
fs.inotify.max_user_watches=90000000
fs.inotify.max_user_instances = 25600000

# To take effect of changes
sudo sysctl -p
```





