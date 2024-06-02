
# Arch Linux: Installation guide
In this guide, I will show you the full Arch Linux installation process starting from your Windows Desktop and ending on your Arch Linux Desktop Environment.

This guide covers as many things as I could, explaining everything I could. Installation both with and without a USB Drive **is covered**, installation with a CD/DVD **is covered**, **Virtualization is also covered**. Installing in network boot **is not yet covered**.

# Getting into an Arch Live Environment
There are many ways to boot a Live Environment. The recommended way is using a USB Drive for actual computers, and booting the ISO directly in case of Virtual Machines.
## Downloading the ISO file
Visit the [Download](https://archlinux.org/download/) page and, depending on how you want to boot, acquire the ISO file or a netboot image
### BitTorrent Download (recommended)
Install a BitTorrent client such as [qBittorrent](https://www.qbittorrent.org/download) and open the magnet link on the download website. The ISO should start downloading after you open the magnet link with your BitTorrent client.
### Direct download from mirror
Find a mirror for your country and download the ISO directly from there, you will not need a BitTorrent client for this.
## Making a bootable media
If you are installing onto a Virtual Machine, you can skip to Booting the Live Environment.
### Using a USB Drive
**!! Warning !! All data on the USB Drive will be wiped. Backup anything you need first!**
Download and Install the KDE ISO Image Writer and use it to flash the ISO file onto your chosen USB Drive.
### Using an Optical Disk
### Using Localboot (not recommended)
Using local boot to install Arch is not recommended because it is a dangerous, long, and painful process, and if done wrong you could get to the point where you have to use another computer to make a bootable media on a USB Drive. If you don't have a USB Drive or a CD/DVD Disk, I would still recommend trying network booting first if your computer supports it. If you don't have any other option and have the time and nerves to do this, good luck.
#### Partitioning
## Booting the Live Environment
### Booting from a USB Drive or Optical Disk
### Booting a virtual machine from ISO
### Booting locally (again not recommended)
# Installing Arch Linux
## Setting a keyboard layout
*If you want to use the default keyboard layout (English US) then you can skip this step*<br>
You can list the available keyboard layouts by using this command:
```bash
localectl list-keymaps
```
You can then select the keyboard layout by running this command, replacing `<keymap>` with the layout you want to use:
```bash
loadkeys <keymap>
```
for example, you would change your keyboard layout to a Czech one by running `loadkeys cz-qwertz`
## Changing the console font
You can change the font that the console uses in this section. First, if you want to know what keyboard layouts are there, you can check out this list or run this command:
```bash
ls /usr/share/kbd/consolefonts/
```
Then, you want to choose a font by running this command, replacing `<font>` for the font you want to use
```bash
setfont <font>
```
## Verify what boot mode did you boot in
Run this command to see what boot mode did you boot in.
```bash
cat /sys/firmware/efi/fw_platform_size
```
If the command returns `64`, you are booted in UEFI and have 64-bit x64 UEFI
If the command returns `32`, you are booted in UEFI and have 32-bit IA32 UEF
If the command returns that the file doesn't exist, you are booted in BIOS (or CSM) mode

If your computer supports UEFI, it is strongly recommended to use that instead of legacy BIOS. If it doesn't, don't worry, it's no problem.

If your computer hasn't booted in the mode you desired, check your motherboard's or computer's manual (depends if you have a custom build or a pre-built desktop/laptop)

## Connecting to the internet
You will need an internet connection to install Arch Linux, since all the packages get downloaded from the internet.
### Ethernet
Plug in your Ethernet cable
### WiFi
First, if you do not know your wireless device name, list all Wi-Fi devices:
```bash
iwctl device list
```
If the device is turned off, turn it on, replacing `<name>` with the name of your device:
```bash
iwctl device <name> set-property Powered on
```
If the device's adapter is turned off, turn it on, replacing `<adapter>` with the adapter of your device:
```bash
iwctl adapter <adapter> set-property Powered on
```
Then, to scan for networks, use this command, again replacing `<name>` with the name of your device:
```bash
iwctl station <name> scan
```
This will not output anything, it will just scan for networks. To actually see what networks it found, use this command:
```bash
iwctl station <name> get-networks
```
Finally, to connect to a network that **is password protected**, use this command replacing `<password>` with the password and `<SSID>` with the name of the network:
```bash
iwctl --passphrase <password> station <name> connect <SSID>
```
To connect to a network that is **not password protected**, use this command, replacing `<SSID>` with the name of the network:
```bash
iwctl station <name> connect <SSID>
```
### Verifying connection
You can now verify that you are connected to the internet by pinging any page, such as google.com:
```bash
ping google.com
```
If you see bytes being sent through, you are connected to the internet and you can cancel the ping by using the Ctrl + C shortcut, and continue on.

## Partitioning the disks
Please remember that if you are booted in UEFI then you should use the UEFI partitioning guide, if you are booted in Legacy BIOS, you should use this BIOS partitioning guide.
### Normal installation with UEFI
First, list all the disks and partitions with lsblk:
```bash
lsblk
```
Now, you will see a list of disks. You want to find the disk you want to install Arch Linux to. You can probably find it by size, but if you have multiple disks and you are not sure which one is the right one, I recommend removing all the other disks except the one you want to install to during installation. Now, you want to remember the name of your disk, it will probably be something like `sda` or `nvme0n1` or something similar.

Now, you want to start a partitioning tool on the disk that you want to install your system to by using this command, replacing `<diskname>` with the name of your disk from the previous step:
```bash
fdisk /dev/<diskname>
```

Now, before doing any actual partitioning, we want to wipe the disk clean by removing any existing partitions. You can do this by using the `d` command in the fdisk console and then selecting the partition we want to remove. Since we don't care about what partitions we delete, we just want a clean slate, you don't have to select which partition to remove and just press enter. Repeat the `d` command until the output says `No partition is defined yet!`

Now, since we are in UEFI mode, we have to make three partitions. An EFI partition that the system will use to boot, the Swap partition (this is not needed but recommended), and the root partition where the whole system will be stored. You can also make a separate home partition for all your files and documents, but it is not needed and your home directory can be stored in your root partition.

#### Making the EFI partition (required)
First, we have to make a new partition. We can do this by using the `n` command:
`n`
Now, you will be asked for the partition type. The partition type should be primary so you can input `p` or just leave it blank and press enter since primary type is the default option.
Next, you will be asked for the partition number. The EFI partition should be the first partition, so you can just input `1` or leave it blank and press enter because 1 is the default option
Now you will be asked for the first sector, I recommend just leaving it blank and pressing enter because the default is usually the best choice for the first sector.
Finally, you will be asked for the last sector. The recommended size of the EFI partition is 1 GiB, so when you want to add 1 GiB from the first sector as the last sector, you just say `+1GiB` and press enter

Now, we have the partition made, but it is of the wrong type. To change the type, use the `t` command:
`t`
Then, when it asks for the type, you want to list all types by inserting `L`
Now, all the partition types will get listed. Now, you should search for `EFI (FAT-12/16/32)` or `EFI System` and type in the prefix before it and press enter

#### Making the Swap partition (recommended)
*If you don't want to make a Swap partition, skip ahead to "Making the Root partition"*
First, we have to make a new partition. We can do this by using the `n` command:
`n`
Now, you will be asked for the partition type. The partition type should be primary so you can input `p` or just leave it blank and press enter since primary type is the default option.
Next, you will be asked for the partition number. The Swap partition should be the second partition, so you can just input `2` or leave it blank and press enter because 2 will be the default option
Now you will be asked for the first sector, I recommend just leaving it blank and pressing enter because the default is usually the best choice for the first sector.
Finally, you will be asked for the last sector. For most computers, the size of 4 GiB for the Swap partition is enough, so when you want to add 4 GiB from the first sector as the last sector, you just say `+4GiB` and press enter

Now, we have the partition made, but it is of the wrong type again. To change the type, use the `t` command:
`t`
When it asks you for the partition number, select `2`
Then, when it asks for the type, you want to list all types by inserting `L`
Now, all the partition types will get listed. Now, you should search for `Linux swap / Solaris` or `Linux swap` and type in the prefix that's displayed before it and press enter

#### Making the Root partition (required)
*Please note that from this point, the setup process might differ if you haven't made a Swap partition.*
First, we have to make a new partition. We can do this by using the `n` command:
`n`
Now, you will be asked for the partition type. The partition type should be primary so you can input `p` or just leave it blank and press enter since primary type is the default option.
Next, you will be asked for the partition number. The Root partition will be the third partition since we have two other ones already, so you can just input `3` or leave it blank and press enter because 3 will be the default option
Now you will be asked for the first sector, I recommend just leaving it blank and pressing enter because the default is usually the best choice for the first sector.
Finally, you will be asked for the last sector. If you don't want to make a home partition as well, you can just press enter and the remaining space will be used. If you want to also have a home partition, consider what size do you want your home partition to be and type in `-<amount>GiB` replacing `<amount>` with the amount of GiB to leave for the home partition.

Now, we have the partition made, but it is of the wrong type again. To change the type, use the `t` command:
`t`
When it asks you for the partition number, select `3` since the root partition is the third partition
Then, when it asks for the type, you want to list all types by inserting `L`
Now, all the partition types will get listed. Now, you should search for `Linux` with the `83` prefix or `Linux root (x86-64)` with the `23` prefix and type in the prefix and press enter

If you don't want to make a home partition, skip ahead to the "Finishing partitioning" step, If you do want to make a home partition, don't skip and go to the next topic
#### Making a home partition (optional)
#### Finishing partitioning
You have now successfully made all your partitions, but the changes haven't yet been written. You can write the changes that you have made by using the `w` command. If you feel that you messed up at any point in the process you can use the `q` command to quit without saving changes and do this whole part again.

### Normal installation with BIOS
First, list all the disks and partitions with lsblk:
```bash
lsblk
```
Now, you will see a list of disks. You want to find the disk you want to install Arch Linux to. You can probably find it by size, but if you have multiple disks and you are not sure which one is the right one, I recommend removing all the other disks except the one you want to install to during installation. Now, you want to remember the name of your disk, it will probably be something like `sda` or `nvme0n1` or something similar.

Now, you want to start a partitioning tool on the disk that you want to install your system to by using this command, replacing `<diskname>` with the name of your disk from the previous step:
```bash
fdisk /dev/<diskname>
```

Now, before doing any actual partitioning, we want to wipe the disk clean by removing any existing partitions. You can do this by using the `d` command in the fdisk console and then selecting the partition we want to remove. Since we don't care about what partitions we delete, we just want a clean slate, you don't have to select which partition to remove and just press enter. Repeat the `d` command until the output says `No partition is defined yet!`

Now, since we are in BIOS mode, we have to make two partitions. A Swap partition (this is not needed but recommended), and the root partition where the whole system will be stored. You can also make a separate home partition for all your files and documents, but it is not needed and your home directory can be stored in your root partition.

#### Making the Swap partition (recommended)
*If you don't want to make a Swap partition, skip ahead to "Making the Root partition"*
First, we have to make a new partition. We can do this by using the `n` command:
`n`
Now, you will be asked for the partition type. The partition type should be primary so you can input `p` or just leave it blank and press enter since primary type is the default option.
Next, you will be asked for the partition number. The Swap partition should be the first partition, so you can just input `1` or leave it blank and press enter because 1 will be the default option
Now you will be asked for the first sector, I recommend just leaving it blank and pressing enter because the default is usually the best choice for the first sector.
Finally, you will be asked for the last sector. For most computers, the size of 4 GiB for the Swap partition is enough, so when you want to add 4 GiB from the first sector as the last sector, you just say `+4GiB` and press enter

Now, we have the partition made, but it is of the wrong type. To change the type, use the `t` command:
`t`
When it asks for the type, you want to list all types by inserting `L`
Now, all the partition types will get listed. Now, you should search for `Linux swap / Solaris` or `` and type in the prefix that's displayed before it and press enter

#### Making the Root partition (required)
*Please note that from this point, the setup process might differ if you haven't made a Swap partition.*
First, we have to make a new partition. We can do this by using the `n` command:
`n`
Now, you will be asked for the partition type. The partition type should be primary so you can input `p` or just leave it blank and press enter since primary type is the default option.
Next, you will be asked for the partition number. The Root partition will be the second partition since we have one other already, so you can just input `2` or leave it blank and press enter because 2 will be the default option
Now you will be asked for the first sector, I recommend just leaving it blank and pressing enter because the default is usually the best choice for the first sector.
Finally, you will be asked for the last sector. If you don't want to make a home partition as well, you can just press enter and the remaining space will be used. If you want to also have a home partition, consider what size do you want your home partition to be and type in `-<amount>GiB` replacing `<amount>` with the amount of GiB to leave for the home partition.

Now, we have the partition made, but it is of the wrong type. To change the type, use the `t` command:
`t`
When it asks you for the partition number, select `2` since the root partition is the second partition
Then, when it asks for the type, you want to list all types by inserting `L`
Now, all the partition types will get listed. Now, you should search for `Linux` with the `83` prefix or `Linux x64/x86` with the `xx` prefix and type in the prefix and press enter

If you don't want to make a home partition, skip ahead to the "Finishing partitioning" step, If you do want to make a home partition, don't skip and go to the next topic
#### Making a home partition (optional)
#### Finishing partitioning
You have now successfully made all your partitions, but the changes haven't yet been written. You can write the changes that you have made by using the `w` command. If you feel that you messed up at any point in the process you can use the `q` command to quit without saving changes and do this whole part again.
### Localboot
## Formatting the partitions
We have all the partitions made, but haven't formatted them yet. First, run `lsblk` again to see how the partitions got made.
```bash
lsblk
```
You should see branches coming from your disk showing the partitions. If you don't see your partitions, you might've done something wrong in the partitioning process.

Now, you want to actually remember the names of the partitions themselves and not the whole disk, and which partition corresponds to what. The EFI partition (UEFI only) will be the one that says `1G` for its size, the Swap partition (if you have made one) will say `4G` for its size, and etc.

### Formatting the EFI partition (UEFI only)
You can format the EFI partition to fat32 by using this command, replacing `<efipartition>` with the name of your EFI partition:
```bash
mkfs.fat -F 32 /dev/<efipartition>
```
### Formatting the Swap partition
You can initialize the swap partition by using this command, replacing `<swappartition>` with the name of your swap partition:
```bash
mkswap /dev/<swappartition>
```
### Formatting the Root partition
You can format the root partition to ext4 by using this command, replacing `<rootpartition>` with the name of your root partition:
```bash
mkfs.ext4 /dev/<rootpartition>
```
### Formatting the Home partition
You can format the home partition to ext4 by using this command, replacing `<homepartition>` with the name of your home partition:
```bash
mkfs.ext4 /dev/<homepartition>
```

## Mount the partitions
We have the partitions created and formatted, but now we have to actually mount them. This is a pretty simple process.
### Mounting the Root partition
The root partition has to be mounted first, and it can be mounted by using this command, replacing `<root>` with the name of your root partition:
```bash
mount /dev/<root> /mnt
```
### Mounting the EFI partition (UEFI only)
If you have an EFI partition, it can be mounted with this command, replacing `<efi>` with the name of your EFI partition:
```bash
mount --mkdir /dev/<efi> /mnt/boot
```
### Mounting the Home partition (If you made one)
If you have a home partition, it can be mounted with this command, replacing `<home>` with the name of your home partition:
```bash
mount --mkdir /dev/<home> /mnt/home
```

### Mounting the Swap partition
If you have a swap partition, it can be mounted with this command, replacing `<swap>` with the name of your swap partition:
```bash
swapon /dev/<swap>
```

## Installing the system itself
Everything is done, we can now get to the installation. First, we have to strap the base package, linux kernel and firmware to our root partition since the partitions are still empty and cannot do anything right now. We will use pacstrap for this.
```bash
pacstrap -K /mnt base linux linux-firmware
```
pacstrap downloads these packages, and then straps them onto our mountpoint `/mnt`

## Configuration of the system
Our Arch Linux system is now installed, but it isn't configured yet and if we went to boot it right now, it wouldn't.
### Generating the fstab
The fstab is essentially the file that tells our system what partitions should be mounted at which mountpoints every time during startup. This configuration is currently saved in our live session, so we just need to generate a file for the system by using this command:
 ```bash
 genfstab -U /mnt >> /mnt/etc/fstab
 ```

 ### Change root into the new system
 We now need to do things in our new system, and we can do that by changing the root to our new system so that it's just like we were on the system. We can do this with this command:
 ```bash
 arch-chroot /mnt
 ```
 
### Installing a text editor
Right now we need to install a text editor to proceed. For this guide, I will be using nano as my text editor, and you can install it with this command:
```bash
pacman -S nano
```
### Persistent keyboard layout
The keyboard layout that you have set will reset to the default (English US) once you reboot your machine. If you want to change your keyboard layout and for it to stay even after reboots, use this command to make a `vconsole.conf` file in the `/etc/` directory, and open it with nano:
```bash
nano /etc/vconsole.conf
```
nano will now open the file it has just made. You can change the keyboard layout persistently by typing this into the text file, replacing `<layout>` with the keyboard layout you want to use:
```
KEYMAP=<layout>
```
When you're done, press Ctrl + S to save, and Ctrl + X to exit.

### Network configuration
Create a `/etc/hostname` file and add the hostname entry to this file. Hostname is basically the name of your computer on the network.

In my case, I’ll set the hostname as `arch`, because it makes sense. You can choose whatever you want by replacing `<hostname>` with the hostname you want to use:
```bash
echo <hostname> > /etc/hostname
```

Next, let's create the hosts file, and edit it to put some configuration into it. You can do this with this command:
```bash
nano /etc/hosts
```
Next, input these lines into the file, replacing `<hostname>` with your hostname again:
```
127.0.0.1 localhost
::1 localhost
127.0.1.1 <hostname>
```

### Root password
Now, you want to create a password for the root account. The root account has unlimited privileges, and if you don't set a password for it now, you won't be able to access it unless you come back to the live session sometime and set the password. Please note that this is not the password for your user account, but for the root account.
You can set a password for the root account by using this command:
```bash
passwd
```
You will then be asked for the password twice.

### Installing a bootloader
The bootloader is used to boot the system, and without it the system won't even be recognized by your computer. There are many bootloaders out there, but I will be using GRUB. There are many parameters useful to know, so I'm gonna try to cover most of them.
#### GRUB on UEFI
For UEFI, GRUB has options to make a system portable, to support secure boot, and more things.

**Installing the packages**
We first need to install the grub package as well as the efibootmgr package. You can do this with this command:
```bash
pacman -S grub efibootmgr
```
**Mounting again**
Next, we need to mount the EFI partition again to a different location, we can do this by using this command, replacing `<efi>` with the name of your EFI partition:
```bash
mount --mkdir /dev/<efi> /boot/efi
```

**Installing GRUB with the least parameters**
The least you can provide to grub when installing to UEFI is the EFI directory and the target type.
We can install GRUB with these parameters using this command:
```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi
```

**Installing GRUB with a different name for the UEFI entry**
By default, the GRUB bootloader is gonna appear as "GRUB" in UEFI. If you want to pick a different name, you can also provide the `--bootloader-id="<name>"` argument, replacing `<name>` with the name you want to see in UEFI. For example, if I wanted to see "Arch Linux" in UEFI, I would type this command to install GRUB:
```bash
grub-install --target=x86_64-efi --bootloader-id="Arch Linux" --efi-directory=/boot/efi
```

**Installing GRUB with secure boot support**
By default, GRUB will not boot on devices that have secure boot enabled. You can enable secure boot support by passing `--modules="tpm" --disable-shim-lock` to the command. For example, if I wanted to enable secure boot support, I would use this command to install GRUB:
```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --modules="tpm" --disable-shim-lock
```
#### GRUB on BIOS
For BIOS, GRUB can really only by installed one way

**Installing the packages**
We first need to install the grub package. You can do this with this command:
```bash
pacman -S grub
```

**Installing GRUB**
To install GRUB on BIOS systems, you need to provide the name of your whole disk, not just any partition. To install GRUB on BIOS systems, use this command, replacing `<diskname>` with the name of the disk where you have your system installed
```bash
grub-install --target=i386-pc /dev/<diskname>
```

#### Generating a GRUB Configuration
After installing GRUB, you need to generate its configuration. You can do that by using this command:
```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

### Create additional user and change privileges
You should not boot into a system, which has only a root user account. This way, every change you make will happen without any authentication required, and you might end up messing up your system.
Of course, you can still choose to do it, but it is not the recommended solution for a stable and secure experience.
So you should make an additional user account, that will get its privileges using sudo
#### Installing sudo
You can install sudo with this command:
```bash
pacman -S sudo
```
#### Creating a new user
You can create a new user with this command, replacing `<username>` with the username you want your user to have:
```bash
useradd <username>
```
#### Setting a password for the user
You can set the password for this user by using this command, replacing `<username>` with the username of your user:
```bash
passwd <username>
```
You will be asked for your password twice.
#### Adding permissions
Now, we need to add this user to groups that enable specific permissions. This should be pretty self-explanatory, the wheel group makes the user a superuser (superusers can execute any command)
```bash
usermod -aG wheel,audio,video,storage <username>
```
Now, we need to edit the visudo file to enable the members of the wheel group to execute any command, after their password is given. We will first open the visudo file using this command:
```bash
EDITOR=nano visudo
```

Now, we need to scroll down through this until we find the lines that say:
```
## Uncomment to allow members of group wheel to execute any command
# %wheel ALL=(ALL:ALL) ALL
```
Now, we need to remove the `#` symbol at the `# %wheel ALL=(ALL:ALL) ALL` line so that it looks like this:
```
## Uncomment to allow members of group wheel to execute any command
%wheel ALL=(ALL:ALL) ALL
```
After you've done this, press Ctrl + S to save, and Ctrl + X to exit.

### Making the network work
To make the network work, we will need some packages. First, we need to install networkmanager with dhcpcd for networking, and the iwd package for WiFi:
```bash
pacman -S networkmanager dhcpcd iwd
```
Next, you need to enable the networkmanager service with this command:
```bash
systemctl enable NetworkManager
```

# You are done! (...if you didn't localboot)
You are done! The system is installed and ready for its first boot, with all the tools needed to continue. To finish the process, first exit the chroot with this command:
```bash
exit
```
Now finally, reboot the computer with this command:
```bash
reboot
```
Once your computer turns off, you can remove your bootable media and you will be booted into Arch Linux.
