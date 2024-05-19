
# Archlinux Install Script

Custom archlinux script for installation

This Script will install archlinux without using the official archinstall script.

It will format your 3 partitions, which u need to create manually.

It will also set the language and keymaps to german.


## Prerequisites

boot your archlinux iso, then enable multilib in /etc/pacman.conf.

type `nano /etc/pacman.conf` and change

```bash
  #[multilib]
  #Include = /etc/pacman.d/mirrorlist
```

to

```bash
  [multilib]
  Include = /etc/pacman.d/mirrorlist
```

`CTRL+O` to save and `CTRL+X` to close nano

list and partition your disk

create 3 partitions:

1 efi ~600M

1 swap ~4-8G

1 linux filesystem


```bash
  lsblk
  cfdisk /dev/sda
```
replace `/dev/sda` with your target disk.

u should have  3 partitions now like:

`/dev/sda1` for your efi

`/dev/sda2` for your swap

`/dev/sda3` for your filesystem
## Installation

With your 3 partitions created you can start the installation.

install.sh is only needed, if u put the script like me inside a ISO. I'm a bit lazy, i dont want to always make a new iso if i make a change in 1 line :)

run these commands and change loadkeys to your choice, mine is german.

```bash
  loadkeys de-latin1 
  curl https://raw.githubusercontent.com/padtrick/arch_install/main/current_install.sh -o current_install.sh
  chmod +x current_install.sh
  clear
  lsblk
  sh current_install.sh
```

just follow the few prompts and wait :)

## create your own Archiso

#Copy archiso files. Run in Konsole
```bash
mkdir -p ./archiso
cd ./archiso
cp -r /usr/share/archiso/configs/releng/* ./
```
#Add packages to packages.x86_64
```bash
git
python
python-setuptools
```

#Modify pacman.conf, remove # infront of these lines to enable multilib
```bash
[multilib]
Include = /etc/pacman.d/mirrorlist
```
#copy the archiso/airootfs folder from this github-repo into your archiso folder or create the files manually

#Create a skel .zprofile for autolaunch. Run in Konsole (archiso folder)
```bash
cat <<\EOF >> ./airootfs/root/.zprofile
[[ -z $DISPLAY && $XDG_VTNR -eq 1 ]] && sh -c "loadkeys de-latin1; cd /root; chmod +x /usr/local/bin/greeting; chmod +x /usr/local/bin/parthelp; chmod +x /usr/local/bin/wifihelp; chmod +x /usr/local/bin/wifihelp; chmod +x /usr/local/bin/install; /usr/local/bin/greeting"
EOF
```

#Build. Run in Konsole (archiso folder)
```bash
mkarchiso -v -w work/ -o out/ ./
```


## INFO

This script isn't perfect or finished.

Windows/Archlinux dualboot works, adding the instructions in howto setup it up, soon

it already works flawless if done manually :)
