
# A PERSONAL Arch Installation Guide


#### This is a personal guide so if you're lost and just found this guide from somewhere, I recommend you to read the official [`wiki`](https://wiki.archlinux.org/index.php/Installation_guide)! This guide will focus on systemd-boot and UEFI setups, so we will not use `grub`. 

Arch changed a lot of things in packaging like removing a lot of packages in base group. I will mostly forgot those yet again, so I need this guide for myself.

I'm using GPT and systemd-boot in my installation.  

### Set the keyboard layout
The default console keymap is US. Available layouts can be listed with:

```
$ ls /usr/share/kbd/keymaps/**/*.map.gz
```

To modify the layout, append a corresponding file name to loadkeys, omitting path and file extension. For example, to set a US keyboard layout:  

```
$ loadkeys us
```


### Update the system clock
Use timedatectl to ensure the system clock is accurate:

```
$ timedatectl set-ntp true
```

### Disk Partitioning
**Verify the boot mode**
If UEFI mode is enabled on an UEFI motherboard, Archiso will boot Arch Linux accordingly via systemd-boot. To verify this, list the efivars directory:  

```
$ ls /sys/firmware/efi/efivars
```

**Check the drives**
The most common main drive is **sda**.

```
$ lsblk
```

**Let’s clean up our main drive to create new partitions for our installation**  

```
$ gdisk /dev/sda
```

  Press <kbd>x</kbd> to enter **expert mode**. Then press <kbd>z</kbd> to *zap* our drive. Then hit <kbd>y</kbd> when prompted about wiping out GPT and blanking out MBR.  
  
**NOTE:**  
Remember that pressing <kbd>x</kbd> button will put you to *expert mode* and <kbd>z</kbd> will *zap* or wipe-out the drive. **Think carefully before pressing <kbd>Y</kbd> you can't undo this!**

**Now let’s create our partitions**  

```
$ cgdisk
```
Enter the device filename of your main drive, e.g. `/dev/sda`. Just press <kbd>Return</kbd> when warned about damaged GPT.

Now we should be presented with our main drive showing the partition number, partition size, partition type, and partition name. If you see list of partitions, delete all those first.

**Let’s create our boot partition**  

   + Hit New from the options at the bottom.
   + Just hit enter to select the default option for the first sector.
   + Now the partion size - Arch wiki recommends 200-300 MB for the boot + size. Let’s make it 500MiB or 1GB in case we need to add more OS to our machine. I’m gonna assign mine with 1024MiB. Hit enter.
   + Set GUID to **EF00**. Hit enter.
   + Set name to boot. Hit enter.
   + Now you should see the new partition in the partitions list with a partition type of EFI System and a partition name of boot. You will also notice there is 1007KB above the created partition. That is the MBR. Don’t worry about that and just leave it there.

**Create the swap partition**

   + Hit New again from the options at the bottom of partition list.
   + Just hit enter to select the default option for the first sector.
   + For the swap partition size, it is advisable to have 1.5 times the size of your RAM. I have 8GB of RAM so I’m gonna put 12GiB for the partition size. Hit enter.
   + Set GUID to **8200**. Hit enter.
   + Set name to swap. Hit enter.

**Create the root partition**

   + Hit New again.
   + Hit enter to select the default option for the first sector.
   + Hit enter again to input your root size.
   + Also hit enter for the GUID to select default.
   + Then set name of the partition to root.


**Create the home partition**

   + Hit New again.
   + Hit enter to select the default option for the first sector.
   + Hit enter again to use the remainder of the disk.
   + Also hit enter for the GUID to select default.
   + Then set name of the partition to home.

Lastly, hit **Write** at the bottom of the patitions list to *write the changes* to the disk. Type `yes` to *confirm* the write command. Now we are done partitioning the disk. Hit **Quit** *to exit cgdisk*.


### FORMATTING PARTITIONS

**Let’s see the new partition list**

```
$ lsblk
```

You should see something like this:

| NAME | MAJ:MIN | RM | SIZE | RO | TYPE | MOUNTPOINT |
| --- | --- | --- | --- | --- | --- | --- |
| sda | 8:0 | 0 | 477G | 0 | disk |   |
| sda1 | 8:1 | 0 | 1 | 0 | part | /boot |
| sda2 | 8:2 | 0 | 1 | 0 | part | [SWAP] |
| sda3 | 8:3 | 0 | 175G | 0 | part | / |
| sda4 | 8:4 | 0 | 300G | 0 | part | /home  |

**`sda`** is the hard drive
**`sda1`** is the boot partition
**`sda2`** is the swap partition
**`sda3`** is the home partition
**`sda4`** is the root partition

**Format the boot partition as FAT32**  

```
$ mkfs.fat -F32 /dev/sda1
```  

**Enable the swap partition**  

```
$ mkswap /dev/sda2  
$ swapon /dev/sda2
```  

**Format home and root partition as ext4**  

```
$ mkfs.ext4 /dev/sda3  
$ mkfs.ext4 /dev/sda4
```  

### Connect to internet

We need to make that sure we are connected to the internet to be able to install Arch Linux `base` and `linux` packages. Let’s see the names of our interfaces.

```
$ ip link
```

You should see something like this:
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp0s30: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff
3: wlp7s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DORMANT group default qlen 1000
    link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff permaddr 00:00:00:00:00:00
```

**Where:**
**`enp0s30`** is the wired interface
**`wlp7s0`** is the wireless interface

If you are on a wired connection, you can enable your wired interface by systemctl start dhcpcd@<interface>.  

```
$ systemctl start dhcpcd@enp3s0
```

If you are on a laptop, you can connect to a wireless access point using wifi-menu -o <wireless_interface>.

```
$ systemctl enable netctl  
$ wifi-menu wlp7s0
```  

Ping google to make sure we are online:

```
$ ping -c 3 google.com
``` 

If you receive Unknown host or Destination host unreachable response, means you are not online yet. Review your network configuration and redo the steps above.

After setting up the drive and connecting to the internet, we are now ready to install Arch Linux to our system.

First we need to mount the partitions that we created earlier to their appropriate mount points.

### Mounting partitions

Mount the **`root`** partition:  

```
$ mount /dev/sda3 /mnt
```  

**If `/mnt` doesn't exists create it.**

```
mkdir /mnt
```  

Mount the **`boot`** partition  

```
$ mkdir /mnt/boot  
$ mount /dev/sda1 /mnt/boot
```  

Mount the **`home`** partion  

```
$ mkdir /mnt/home
$ mount /dev/sda4 /mnt/home
``` 

We don’t need to mount **swap** since it is already enabled.  

### Installing the base and linux packages

Now let’s go ahead and install **`base`**, **`linux`** and **`base-devel`** packages into our system. The kernel is not included in the base group including other application like editors. Even the lovely *nano* doesn't survived the snap.  

So let's also install the packages detached from the base group.  

```
$ pacstrap /mnt base linux linux-firmware base-devel less logrotate man-db man-pages which dhcpcd inetutils jfsutils mdadm perl reiserfsprogs sysfsutils systemd-sysvcompat texinfo usbutils xfsprogs s-nail netctl nano iputils
```

It's your decision if you want to install the packages.  

Just hit enter/yes for all the prompts that come up. Wait for Arch to finish installing the base, linux and base-devel and other packages.  


### Generating the fstab

```
$ genfstab -U /mnt >> /mnt/etc/fstab
```  


### Chroot time

Now, chroot to the newly installed system  

```
$ arch-chroot /mnt /bin/bash
```

### Configure the time

A selection of timezones can be found under `/usr/share/zoneinfo/`. Since I am in the Philippines, I will be using `/usr/share/zoneinfo/Asia/Manila`. Select the appropriate timezone for your country:  

```
$ ln -s /usr/share/zoneinfo/Asia/Manila /etc/localtime
```

Run hwclock to generate `/etc/adjtime`:  
```
$ hwclock --systohc
```

### Localization

The Locale defines which language the system uses, and other regional considerations such as currency denomination, numerology, and character sets. Possible values are listed in `/etc/locale.gen`. Uncomment `en_US.UTF-8`, as well as other needed localisations.

**Uncomment** `en_US.UTF-8 UTF-8` and other needed locales in `/etc/locale.gen`, **save**, and generate them with:  
```
$ locale-gen
```

Create the locale.conf file, and set the LANG variable accordingly:  

```
$ echo 'LANG=en_US.UTF-8' > /etc/locale.conf
```

If you set the keyboard layout, make the changes persistent in vconsole.conf:  

```
echo 'KEYMAP=us' > /etc/vconsole.conf
```


### Network configuration

Create the hostname file:  

```
$ echo 'MYHOSTNAME' > /etc/hostname
```

Open/Create hosts file to add matching entries to hosts:  

```
nano /etc/hosts
```  

Add this:  

```
127.0.0.1    localhost  
::1          localhost  
127.0.1.1    MYHOSTNAME.localdomain	  MYHOSTNAME  
```

**NOTE:**  
If the system has a permanent IP address, it should be used instead of 127.0.1.1.

### Initramfs  
Creating a new initramfs is usually not required, because mkinitcpio was run on installation of the kernel package with pacstrap.
For LVM, system encryption or RAID, modify mkinitcpio.conf and recreate the initramfs image:  

```
$ mkinitcpio -p linux
```

### Wireless Connections
For wireless connections, install iw, wpa_supplicant, and (for wifi-menu) dialog. This is needed if you want to use a wi-fi connection after the next reboot!

```
$ pacman -S iw wpa_supplicant dialog
```

### Adding Repositories
Enable multilib and AUR repositories in `/etc/pacman.conf:`  
```
$ nano /etc/pacman.conf
```

Uncomment multilib (remove # from the beginning of the lines):  
```
[multilib]
Include = /etc/pacman.d/mirrorlist
```
Add the following lines at the end of `/etc/pacman.conf` to enable AUR repo:  

```
[archlinuxfr]
SigLevel = Never
Server = http://repo.archlinux.fr/$arch
```

**Pacman Easter Eggs**

While you're at it, **you can add colors and PAC-MAN in pacman!**

Open `/etc/pacman.conf`, then find `# Misc options` string then **below** that add:

```
Color
ILoveCandy
```

### Passwords and Users

Set the root password:  

```
$ passwd
```

Add a new user with username *MYUSERNAME*:  

```
$ useradd -m -g users -G wheel,storage,power,video,audio,rfkill -s /bin/bash MYUSERNAME
```  

Set password the password for user MYUSERNAME:  

```
$ passwd MYUSERNAME
```


### Add the new user to sudoers:

If you want a root power by using the command `sudo`, you should grant your self with that power.

```
$ EDITOR=nano visudo
```

Uncomment the line
```
# %wheel ALL=(ALL) ALL
```

### Install the boot loader:  

Yeah, this is wher we install the bootloader. No need for `grub2`.
```
$ bootctl install
```

Create a boot entry:  

```
$ nano /boot/loader/entries/arch.conf
```

Add these lines:

```
title Arch Linux  
linux /vmlinuz-linux  
initrd  /initramfs-linux.img  
options root=/dev/sda3 rw
```

Save and exit.

### Exit chroot and reboot:  
```
$ exit
$ reboot
```


## POST INSTALLATION
### My Personal Applications
**This is for my setup so don't follow it if you don't want to.**

| xorg |
| --- |
| xorg-server |
| xorg-xrdb |
| xorg-xinit |
| xorg-xbacklight |
| xorg-xrandr |
| xorg-xev |
| xorg-xdpyinfo |


| Video |
| --- |
| xf86-video-intel |
| gstreamer-vaapi |
| vulkan-intel |
| lib32-vulkan-intel | 
| vulkan-icd-loader | 
| lib32-vulkan-icd-loader |
| lib32-mesa |

| Audio |
| --- |
| alsa-utils |
| pulseaudio-alsa |
| pulseaudio-bluetooth |
| pulseaudio |
| pavucontrol |
| pulseeffects |

| Filesystem |
| --- |
| unrar |
| unzip |
| p7zip |
| gvfs-mtp |
| libmtp |
| ntfs-3g |
| android-udev |
| nemo |
| nemo-fileroller |
| nemo-preview |
| ffmpeg |
| ffmpegthumbnailer |


| Browser |
| --- | 
| firefox |  

| Terminal Emulators  |
| --- |
| kitty |
| xterm |

| Network managers |
| --- |
| networkmanager |
| network-manager-applet |
| dhclient |
| modemmanager |
| usb_modeswitch |
| mobile-broadband-provider-info |

| Network tools |
| --- |
| aircrack-ng |
| macchanger |

| Maintenance |
| --- |
| tlp |
| ufw |
| netdata |
| thinkfan |
| acpid |
| acpi_call |

| Font |
| --- |
| noto-fonts-emoji |

| Bluetooth |
| --- |
| bluez |
| bluez-utils |
| blueman |

| Editors |
| --- |
| vim |
| sub1ime-text |

| terminal eyecandies |
| --- |
| neofetch |

| tools |
| --- |
| imagemagick |
| dconf-editor |
| maim |
| mupdf |
| redshift |
| arandr |

| settings |
| --- |
| lxappearance |
| qt5ct |
| qt5-styleplugins |

| vbox |
| --- |
| virtualbox |
| virtualbox-host-modules-arch |
| virtualbox-guest-iso |
| linux-headers |
| virtualbox-ext-oracle |

| media |
| --- |
| vlc |
| mpd |
| mpc |
| ncmpcpp |
| perl-image-exiftool |
| feh |
| simplescreenrecorder |


| Media editors |
| --- |
| gimp |
| inkscape |

# Aur helper
Install `yay`, a *Yet another yogurt. Pacman wrapper and AUR helper written in go.*
```
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -sri
```

| more aur apps |
| --- |
| picom-tryone-git |
| cava-git |
| awesome-git |
| mugshot |
| flat-remix |
| flat-remix-gtk |
| pamac-aur |
| gtk3-nocsd-git |
| gtk3-mushrooms |
| rxvt-unicode-pixbuf |
| rofi-git |
| glxinfo |
| toilet |
| korla-icon-theme-git |
| pkhex-bin |
| oomox-git |
| ttf-vista-fonts |
