# linux-cheat-sheet
Linux Cheat Sheet with the most needed stuff..






# umount

## Unmount busy target
-  sudo umount -f 
```shell
#!/bin/bash

# Finde alle gemounteten VeraCrypt-Volumes
veracrypt_volumes=$(mount | grep 'veracrypt' | awk '{print $1}')

# Überprüfe, ob VeraCrypt-Volumes gefunden wurden
if [ -z "$veracrypt_volumes" ]; then
    echo "Keine gemounteten VeraCrypt-Volumes gefunden."
else
    # Demontiere jedes gefundene VeraCrypt-Volume
    for volume in $veracrypt_volumes; do
        echo "Versuche, $volume zu demontieren..."
        sudo umount -f "$volume"
        if [ $? -eq 0 ]; then
            echo "$volume erfolgreich demontiert."
        else
            echo "Fehler beim Demontieren von $volume."
        fi
    done
fi
```

v2 (check in electron.js):
```javascript

// Function to cleanup VeraCrypt mounts and device mapper entries
async function cleanupVeraCrypt() {
    if (cleanupPromise) return cleanupPromise // Return existing cleanup promise if one is in progress
    if (isCleaningUp) return Promise.resolve() // Shouldn't happen, but just in case
    
    isCleaningUp = true
    cleanupPromise = (async() => {
        try {
            // First try to dismount all VeraCrypt volumes
            execSync('veracrypt --text --dismount --non-interactive', { stdio: 'pipe' })
            console.log('Successfully dismounted all VeraCrypt containers')

            // Check for stale device mapper entries
            const mapperLs = execSync('ls -l /dev/mapper', { stdio: 'pipe' }).toString()
            const veracryptEntries = mapperLs
                .split('\n')
                .filter(line => line.includes('veracrypt'))
                .map(line => {
                    // Split by spaces but keep multiple spaces as one separator
                    const parts = line.trim().split(/\s+/)
                    // The name is the third-to-last part before " -> ../dm-X"
                    return parts[parts.length - 3]
                })
                .filter(entry => entry && entry.startsWith('veracrypt'))

            if (veracryptEntries.length > 0) {
                console.log('Found stale VeraCrypt device mapper entries:', veracryptEntries)
                
                // Show native dialog box
                await dialog.showMessageBox(mainWindow, {
                    type: 'warning',
                    title: '⚠️ Administrator Rights Required',
                    message: '🔐 Administrator privileges are required to clean up VeraCrypt entries',
                    buttons: ['Continue'],
                    defaultId: 0,
                    noLink: true
                })

                // Combine all cleanup commands into a single pkexec call
                const cleanupCommands = veracryptEntries.map(entry => {
                    return [
                        `dmsetup remove "${entry}" 2>/dev/null || dmsetup remove -f "${entry}" 2>/dev/null || true`,
                        `umount -f "/dev/mapper/${entry}" 2>/dev/null || true`,
                        `dmsetup remove -f "${entry}" 2>/dev/null || true`
                    ].join(' && ')
                }).join(' && ')

                if (cleanupCommands) {
                    try {
                        execSync(`pkexec /bin/bash -c "${cleanupCommands}"`, { stdio: 'pipe' })
                        console.log('Successfully cleaned up stale device mapper entries')
                    } catch (err) {
                        console.error('Failed to remove device mapper entries:', err.message)
                    }
                }
            }
        } catch (error) {
            console.error('Error during VeraCrypt cleanup:', error.message)
        } finally {
            isCleaningUp = false
            cleanupPromise = null
        }
    })()
    
    return cleanupPromise
}
```




<br><br>
_____
_____
<br><br>

















# Empty Trash
```shell
sudo rm -rf ~/.local/share/Trash/*
```


<br><br>
--- 
<br><br>




# Audio



## Pulseeffect
- Open Pulseffects APP
  - Settings -> Audio -> Put every Playback Stream to Pulseeffect. Make sure at the top at Playback devices Pulseeffects app is 100% volume and you selected your Soundcard
    - Playback Stream Pulseeffect goes to your Soundcard
      - Open PulseAudio Volume Control at go to Playback Tab
        - Pulse Effects Playback Stream goes to your soundcard. All other goes to Pulseffects(app)
         - Always use PulseAudio Volume Control from now on instead of regular Audio Settings
  

<br><br>

## Bluetooth Mic not detected

1. Install Pulse Audio Volume Control
```
sudo apt-get install pavucontrol
```

2. Go to tab `Configuration`and set profile `High Fidelity Payback`. Then at Input devices should be able to select your Bluetooth mic input





<br><br>

## Input Mic not detected

1. Install Pulse Audio Volume Control
```
sudo apt-get install pavucontrol
```

2. Then at Input devices should be able to select your Bluetooth mic input








<br><br>
<br><br>

### Sound not working anymore after sign-out

#### Option 2
- Turn off default audio adapter and then on again and then start again pulseeffect

#### Option 2
- Go to Pulse Audio Volume Control by search it via menu bar. Then change audio output for application. If this not work try killall pulseaudio and then sign-out again and try again to change audio output.

<br><br>

### Restart pulseaudio
```
pulseaudio -k && pulseaudio --start
```










<br><br>
<br><br>
___________________________________________________
___________________________________________________
<br><br>
<br><br>


# Certs

<br><br>

## Openssl

<br><br>

### Download cert from address
```shell
openssl s_client -showcerts -connect minio.local.com:443 -servername minio.local.com < /dev/null 2>/dev/null | openssl x509 -outform PEM > minio.local.com.crt
```





















<br><br>
<br><br>
___________________________________________________
___________________________________________________
<br><br>
<br><br>


# PDF

<br><br>

## Convert multiple files to pdf
```shell
sudo apt install imagemagick
convert image1.jpg image2.png text.txt PDFfile.pdf outputFileName.pdf
```











<br><br>
<br><br>
___________________________________________________
___________________________________________________
<br><br>
<br><br>

# Images

<br><br>

## Change quality
```shell
sudo apt install imagemagick
# cd into diretory with the images
mogrify -quality 75% *
```

<br><br>

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


<br><br>

## Convert

<br><br>

### png to webp
```
sudo apt install webp
cwebp -q 60 Cute-Baby-Girl.png -o Cute-Baby-Girl.webp
```

















<br><br>
<br><br>

___________________________________________________
___________________________________________________


<br><br>
<br><br>



# Fonts

## Install
- You can download any font and isntall it via right click on it and then install or you can do it like this in bulk:
```shell
wget -O /tmp/myfont.ttf https://example.com/path/to/font.ttf
mkdir -p ~/.local/share/fonts
cp /tmp/myfont.ttf ~/.local/share/fonts/
fc-cache -fv
```














<br><br>
<br><br>

___________________________________________________
___________________________________________________


<br><br>
<br><br>


# Terminal

## Buffer Size

### Konsole
- If you have started a shell session in Konsole, do the following:

Konsole > Settings > Edit current profile > Scrolling > Scrollback
















<br><br>
<br><br>

___________________________________________________
___________________________________________________


<br><br>
<br><br>

## Bootable USB

<br><br>

### ubuntu/Kubuntu

#### Terminal
```shell
# You can find the partion name via partion manager
sudo dd if=input.iso of=/dev/sdx
```

#### GUI
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

# Terminal Managment



## screen
- https://wiki.ubuntuusers.de/Screen/
```shell
sudo apt-get install screen 
```

### Wichtige Befehle

- Die drei wichtigsten Befehle bzw. Tastenkombinationen, die man kennen sollte, sind:
    - Strg + A , gefolgt von C zum Erstellen eines neuen Fensters (siehe auch Unterschied zwischen Sitzung und Fenster)
    - Strg + A , gefolgt von SPACE zum Wechseln zwischen den einzelnen Fenstern einer Sitzung
    - Strg + A , gefolgt von D zum Trennen (detach) der Verbindung zur aktuellen Sitzung, die Sitzung läuft dann im Hintergrund weiter




















































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
<br><br>

## broken login screen
```shell
loginctl unlock-session 4
```

<br><br>

## restart plasma
```shell
kquitapp5 plasmashell
kstart5 plasmashell

# Restart plasma from tty
# Switch to tty2 by Press CTRL + ALT + F2 and hohld all buttons
systemctl restart sddm.service
# Then switch back to tty by Press CTRL + ALT + F1
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

## Pipe logs into file that you do not see stdout
```bash
port-forward.sh > /dev/null & echo 'Do other stuff where you do not want to see stdout of port-forward.sh'
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
<br><br>

____________________________________________
____________________________________________

<br><br>
<br><br>



# Symlink
- You can create symlinks for files and folders
  - If you decide to symlink a folder then the folder for your destination must not exists because it will be automatically created

- You can use absolute or relative paths. However for git projects you should always use relative paths that other developers can cooperate with your symlinks inside the project
  - It is very important that you cd inside the the destination directory where you want to create the symlink for your folder / file


<br><br>
<br><br>


## Create symlink to another file to use it.
```bash
ln -s ../project/.eslintrc.json ./.eslintrc.json
```

<br><br>

## Create a link to another folder to use it.
```bash
# In this case
# ln -s FolderWhereYouCentralFiles FolderWhereYouCreateSymlink
ln -s ~/Projects/ai/checkpoints/Stable-diffusion ~/Projects/stable-diffusion-webui/models
ls -l ~/Projects/stable-diffusion-webui/models/Stable-diffusion
```

<br><br>

## Show existing symlinks
```bash
ls -la /path/here
```

<br><br>

## remove symlink
```shell
rm -i /pfad/zum/zielordner
```

<br><br>

## create symlincy recursive for all folders expect of specific folder
```shell
# example 1
MIGRATIONS_PATH="/home/dennis/Projects/migrations"

CUSTOM_PROJECT="test-10400"
CUSTOM_PROJECT_PATH="data/$CUSTOM_PROJECT"

INITIAL_PROJECT="initial-project"
INITIAL_PROJECT_PATH="data/$INITIAL_PROJECT"

EXLUDED_FOLDER="Apple"

create_symlinks() {
    for folder in ../$INITIAL_PROJECT/*; do
       if [[ ! "$folder" =~ "$EXLUDED_FOLDER" ]]; then
           ENTITY_NAME="$(basename -- "$folder")"
           echo "ENTITY_NAME: $ENTITY_NAME"
   
           SOURCE="../$INITIAL_PROJECT/$ENTITY_NAME"
           echo "SOURCE: $SOURCE"
     
           DESTINATION="./$ENTITY_NAME"
           echo "DESTINATION: $DESTINATION"  
       
           ln -s "$SOURCE" "$DESTINATION"
       fi
    done
}

# Navigate to the data directory
cd $MIGRATIONS_PATH

# Remove existing directory
rm -rf "$CUSTOM_PROJECT_PATH"

# Create the new client folder
mkdir $CUSTOM_PROJECT_PATH

# Create symlinks
(cd $CUSTOM_PROJECT_PATH; create_symlinks)

# Copy the excluded folder to use it without a symlink
cp -r "$INITIAL_PROJECT_PATH/$EXLUDED_FOLDER" "$CUSTOM_PROJECT_PATH/$EXLUDED_FOLDER"

# Verify that everything worked
ls -l "$CUSTOM_PROJECT_PATH/SchemaEntities"


















# example 2
mkdir -p ~/Dropbox
cd ~
for file in *; do if [[ ! "$file" =~ Video|Dropbox ]]; then ln -s "$HOME/$file" "Dropbox/$file"; fi; done













# example 3
cd ~/lib/migrations/data

# create new client folder
mkdir "test-10400"

# cd into this folder because we will iterate of every file
cd "initial-project"

for file in *; do if [[ ! "$file" =~ cs.Apple ]]; then ln -s "$file" "../test-10400/$file"; fi; done

# Copy the excluded folder - So we use it without symlink
cp -r "cs.Apple" "../test-10400/cs.Apple"
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

## user

<br><br>

### Get details about user processes
```shell
ps -u userHere -f
```




<br><br>
<br><br>


## process

### Get details about process tree
```shell
pstree -p | grep mongosh
```






<br><br>
<br><br>

## Logs
- If you want to find logs after you system crashed try to search for `-- Boot`


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
- 




















































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

# Emulation

## Wine

### Ubuntu 23.04

#### Install
```shell
sudo dpkg --add-architecture i386 

sudo mkdir -pm755 /etc/apt/keyrings
sudo wget -O /etc/apt/keyrings/winehq-archive.key https://dl.winehq.org/wine-builds/winehq.key

sudo wget -NP /etc/apt/sources.list.d/ https://dl.winehq.org/wine-builds/ubuntu/dists/lunar/winehq-lunar.sources

sudo apt install --install-recommends winehq-stable
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

# Corsair

## Corsair iCue
- You can use VMware which does not use any virtualisiation and then run Windows
- You can also boot into Windows PE live










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





