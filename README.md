# Archlinux Install Script (currently only EFI supported)

### Custom Arch Linux script for installation

This Script will install Arch Linux without using the official `archinstall` script.

It will format your 3 partitions, which you need to create manually.

It will also set the language and keymaps to German.

## Supported Configurations

- **Desktop Environments**: Hyprland, KDE, Gnome, XFCE
   - KDE works great
   - Hyprland works but can have some issues with gaming (working on it)
   - Gnome & XFCE aren't tested
- **Driver Presets**: Nvidia proprietary, AMD (Mesa), Intel (Mesa, incl. Intel Arc), Hyper-V
- **Audio Servers**: Pipewire & Pulseaudio

## Prerequisites

Boot your Arch Linux iso, then enable multilib in /etc/pacman.conf.

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

or run the following command to change it

`sed -i "/\[multilib\]/,/Include/"'s/^#//' /etc/pacman.conf`

#### List and Partition Your Disk

Create 3 partitions:

1 EFI ~600M

1 Swap ~4-8G

1 Linux filesystem

```bash
 lsblk
 cfdisk /dev/sda
```

replace `/dev/sda` with your target disk.

u should have 3 partitions now like:

`/dev/sda1` for your efi

`/dev/sda2` for your swap

#### If you run the custom `install` script, you can skip the formatting.

`/dev/sda3` for your filesystem

## Installation

With your 3 partitions created, you can start the installation.

The file `get_current_installscript.sh` is only needed, if you put the script inside an ISO. I'm a bit lazy, I don't want to always make a new ISO if I make a change in 1 line :)

Run these commands and change loadkeys to your choice, mine is German.

```bash
 loadkeys de-latin1
 curl https://raw.githubusercontent.com/padtrick/arch_install/main/current_install.sh -o current_install.sh
 chmod +x current_install.sh
 clear
 lsblk
 sh current_install.sh
```

just follow the few prompts and wait :)

## Dualboot Archlinux & Windows 10/11

### Using the Custom Install Script

#### This example is for EFI - W10 & Archlinux on 1 Disk, if u want to install Windows and Archlinux on different disks, it should be something similar.

Install Windows 10 and shrink the Windows Partition (I have a 1TB NVME, I will do a 50/50 split).

Boot up the Arch Linux ISO and make sure you are connected to the internet. Use `wifihelp` to show some information for wireless connections.

Enter `lsblk` to list Disks, after that run `cfdisk /dev/YOURDEVICE` (for me `cfdisk /dev/nvme0n1`)

#### Create at least 1x Partition of ~600M Type EFI Filesystem, 1 Swap 4GB or more Swap Partition, and 1x Partition for the actual Archlinux installation.

If you run the custom `install` script, you can skip the formatting of the partitions.

#### After installation, don't reboot. We need to copy some Windows 10 files to get dual boot working.

list your disks with `lsblk`

Create a mount point

`mkdir /mnt/win10`

`mount /dev/nvme0n1p1 /mnt/win10` (change the to your disks)

Go into your mount point

`cd /mnt/win10/EFI`

` ls` to list your directory and check if the folder `Microsoft` is present.

copy the Microsoft folder into your `/mnt/boot`

`cp -r /mnt/win10/EFI/Microsoft /mnt/boot/EFI`

Check if it's inside the correct folder `/mnt/boot`

`cd /mnt/boot/EFI` and `ls`

Now you can reboot

`reboot`

### Using the Official Install Script

#### This example is for EFI - W10 & Archlinux on 1 Disk, if u want to install Windows and Archlinux on different disks, it should be something similar.

Install Windows 10 and shrink the Windows Partition (I have a 1TB NVME, I do a 50/50 split).

Boot up the Archlinux Iso and make sure you are connected to the internet. Read the `icwtl` help on how to set up wireless connections.

Create and format those 3 partitions (those commands are for my disks, change to your needs)

`mkfs.vfat -F32 /dev/nvme0n1p5` Format EFI Partition

`mkswap /dev/nvme0n1p6` Format Swap Partition

`swapon /dev/nvme0n1p7` Activate Swap Partition

`mkfs.btrfs /dev/nvme0n1p7` Format Archlinux Partition

#### If you want to use the official archinstall script, you need to mount the partitions manually (this part and also formatting seems broken in archinstall version 2.8.0).

`mount /dev/nvme0n1p6 /mnt` (change to your disk)

`mkdir /mnt/boot`

`mount /dev/nvme0n1p5 /mnt/boot`

after this, start `archinstall`

#### All you need to make sure is that you set up the mount points during Disk Setup.

Choose premounted configuration and type `/mnt` for root.

Your EFI partition should be mounted at `/boot`

Your Arch Linux partition should be mounted at `/`

#### After installation, select YES to change the installation, we need to copy some Windows 10 files.

List your disks with `lsblk`

Create a mount point

`mkdir /mnt/win10`

`mount /dev/nvme0n1p1 /mnt/win10` (change the to your disks)

go into your mount point

`cd /mnt/win10/EFI`

`ls` to list your directory

copy the Microsoft folder into your `/boot`

`cp -r /mnt/win10/EFI/Microsoft /boot/EFI`

Now you can Exit & Reboot

`exit`

`reboot`

## Activate Dualboot after the Installation is already finished and you have rebooted

list your disks with `lsblk`

Create a mount point

`mkdir /mnt/win10`

`mount /dev/nvme0n1p1 /mnt/win10` (change the to your disks)

Go into your mount point

`cd /mnt/win10/EFI`

` ls` to list your directory and check if the folder `Microsoft` is present.

copy the Microsoft folder into your `/boot`

`cp -r /mnt/win10/EFI/Microsoft /boot/EFI`

Check if it is inside the correct folder `/boot`

`cd /boot/EFI` and `ls`

Now you can Reboot and check if it's working

`reboot`

## Create your own Archiso

Copy archiso files. Run in Konsole

```bash
mkdir -p ./archiso
cd ./archiso
cp -r /usr/share/archiso/configs/releng/* ./
```

Add packages to packages.x86_64 or copy from this GitHub repo

```bash
git
python
python-setuptools
```

Modify pacman.conf, remove # in front of these lines to enable multilib or copy from this GitHub repo

```bash
[multilib]
Include = /etc/pacman.d/mirrorlist
```

Copy the archiso/airootfs folder from this GitHub-repo into your archiso folder or create the files manually

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

## INFO

This script isn't perfect or finished.

Looking into getting Grub supported as well.

Currently, the Hyprland installation has some minor issues with gaming, and I'm looking into it.

It could be the dotfiles, but I am learning at the moment how to config Hyprland myself from scratch :)
