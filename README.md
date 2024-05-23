
# Archlinux Install Script

### Custom archlinux script for installation

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


or run the folloing command to change it

`sed -i "/\[multilib\]/,/Include/"'s/^#//' /etc/pacman.conf`


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

The file `get_current_installscript.sh` is only needed, if u put the script like me inside a ISO. I'm a bit lazy, i dont want to always make a new iso if i make a change in 1 line :)

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

## Create your own Archiso

Copy archiso files. Run in Konsole
```bash
mkdir -p ./archiso
cd ./archiso
cp -r /usr/share/archiso/configs/releng/* ./
```
Add packages to packages.x86_64 or copy from this github-repo
```bash
git
python
python-setuptools
```

Modify pacman.conf, remove # infront of these lines to enable multilib or copy from this github-repo
```bash
[multilib]
Include = /etc/pacman.d/mirrorlist
```
Copy the archiso/airootfs folder from this github-repo into your archiso folder or create the files manually

Create a skel .zprofile for autolaunch. Run in Konsole (archiso folder)
```bash
cat <<\EOF >> ./airootfs/root/.zprofile
[[ -z $DISPLAY && $XDG_VTNR -eq 1 ]] && sh -c "loadkeys de-latin1; cd /root; chmod +x /usr/local/bin/greeting; chmod +x /usr/local/bin/parthelp; chmod +x /usr/local/bin/wifihelp; chmod +x /usr/local/bin/wifihelp; chmod +x /usr/local/bin/install; /usr/local/bin/greeting"
EOF
```

Build. Run in Konsole (archiso folder)
```bash
mkarchiso -v -w work/ -o out/ ./
```

## Dualboot Archlinux & Windows 10/11

### Using the Custom Install Script

#### This example is for EFI - W10 & Archlinux on 1 Disk, if u want to install Windows and Archlinux on different disks, it should be something similar.

Install Windows 10 and shrink the Windows Partition (i have a 1TB NVME, i do a 50/50 split).

Boot up the Archlinux Iso and make sure you are connected to the internet. Use `wifihelp` to show some infos for wireless.

Enter `lsblk` to list Disks, after that run `cfdisk /dev/YOURDEVICE` (for me `cfdisk /dev/nvme0n1`)

#### Create at least 1x Partition of ~600M Type EFI Filesystem, 1 Swap 4GB or more Swap Partition and 1x Partition for the actual Archlinux installation.

If you run the custom `install` script, you can skip the formating the partitions.


#### After installation, we need to copy some win10 files to get dual working.

list your disks with `lsblk`

create a mountpoint

`mkdir /mnt/win10`

`mount /dev/nvme0n1p1 /mnt/win10` (change the to your disks)

go into your mountpoint

`cd /mnt/win10/EFI`

` ls` to list your directory and check if the folder `Microsoft` is present.

copy the Microsoft folder into your `/mnt/boot`

`cp -r /mnt/win10/EFI/Microsoft /mnt/boot/EFI`

check if its inside the correct folder `/mnt/boot` 

`cd /mnt/boot/EFI` and `ls`

Now you can Reboot

`reboot`



### Using the Official Install Script

#### This example is for EFI - W10 & Archlinux on 1 Disk, if u want to install Windows and Archlinux on different disks, it should be something similar.

Install Windows 10 and shrink the Windows Partition (i have a 1TB NVME, i do a 50/50 split).

Boot up the Archlinux Iso and make sure you are connected to the internet. Read the `icwtl` help on howto setup wireless.


#### If you want to use the official archinstall script, you need to mount the partitions manually (this part and also formatting seems broken in archinstall version 2.8.0).

`mount /dev/nvme0n1p6 /mnt`

`mkdir /mnt/boot`

`mount /dev/nvme0n1p5 /mnt/boot`

after this, start `archinstall`

Format those partitions (those commands are for my own disks, change to your needs)


#### If you run the custom `install` script, you can skip the formating.

`mkfs.vfat -F32 /dev/nvme0n1p5` Format EFI Partition

`mkswap /dev/nvme0n1p6` Format Swap Partition

`swapon /dev/nvme0n1p7` Activate Swap Partition

`mkfs.btrfs /dev/nvme0n1p7` Format Archlinux Partition

Run `install` to start the Archlinux Installation with the custom Script.


#### all you need to make sure, is that you setup the mount points during Disk Setup.

choose premounted configuration and type `/mnt` for root.

your efi partition should be `/boot`

your archlinux partition should be `/`


#### After installation, select YES to change the installation, we need to copy some win10 files.

list your disks with `lsblk`

create a mountpoint

`mkdir /mnt/win10`

`mount /dev/nvme0n1p1 /mnt/win10` (change the to your disks)

go into your mountpoint

`cd /mnt/win10/EFI`

`ls` to list your directory

copy the Microsoft folder into your `/boot`

`cp -r /mnt/win10/EFI/Microsoft /boot/EFI`

Now you can Exit & Reboot

`exit`
`reboot`




## INFO

This script isn't perfect or finished.

Currently Hyprland installation has some minor issues with gaming, i'm looking into it.

Could be the dot files, learning atm howto config Hyprland myself from scratch :)
