# Arch-Linux-Install By: William Reinhart
This is meant to showcase my successful attempt at installing arch linux with notes on the failures at the end

#Overview: this first part if the successful code that led to the completion, I will document all the code that I used in the order that I used it.

1. 
# System Setup

- **Virtualization Platform**: VMware Workstation
- **Boot Mode**: Legacy BIOS
- **Disk Type**: GPT
- **Disk Size**: 20 GB


2. 
# Creation and formatting the partitions and mounting them
cfdisk /dev/nvme0n1

# created a boot partition (p1) around 2MB and made a root partition (p2) around 20GB

# formatted the root partition to ext4
mkfs.ext4 /dev/nvme0n1p2
mount /dev/nvme0n1p2 /mnt

3.
# using pacstrap to install the needed programs
#-K copied the package managers keyring to the new system
pacstrap -K /mnt base linux linux-firmware vim nano

#Using fstab to generate a filesystem table, used to tell the system where to mount partitions at boot
genfstab -U /mnt >> /mnt/etc/fstab

# Change into chroot to configure the system as if it has already been booted
arch-chroot /mnt

4.

# Once in /mnt these programs are run to configure the system 
# Sets the clock
ln -sf /usr/share/zoneinfo/America/Tulsa /etc/localtime
hwclock --systohc

# Configures the language settings
nano /etc/locale.gen  # Uncomment en_US.UTF-8
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf

# Sets the hostname
echo "archvm" > /etc/hostname
nano /etc/hosts
# Add:
127.0.0.1   localhost
::1         localhost
127.0.1.1   archvm.localdomain archvm

#sets root password
passwd  

5. 
# Installation of grub and networkmanager
pacman -S grub networkmanager

# Installalation of grub to the disk 
grub-install --target=i386-pc /dev/nvme0n1

# This generates a grub configuration file, and -o specifies the output location for the file
grub-mkconfig -o /boot/grub/grub.cfg

# Then this enables network manager
systemctl enable NetworkManager

6.
# After everything was finished I exit and reboot the system
exit

umount -R /mnt

reboot

7. 
# after succesfully rebooting I create the users and the group while also giving the 2 users passwords and sudo permissions

seradd -m -G wheel -s /bin/bash william

passwd william

useradd -m -G wheel -s /bin/bash codi

passwd codi

pacman -S sudo

EDITOR=nano visudo  # Uncomment %wheel ALL=(ALL:ALL) ALL

8.
# I then install the prerequisites to the GUI by updating the system, I decided on going for lxqt

pacman -Syu

pacman -S xorg sddm lxqt

systemctl enable sddm


9.
# I then install the Zsh shell and swapped it to Zsh for both users

pacman -S zsh

chsh -s /bin/zsh william

chsh -s /bin/zsh codi

10. 
# Install, enable, and start SSH

pacman -S openssh

systemctl enable sshd

systemctl start sshd


11.
# Adding the terminal colors
nano /home/william/.zshrc

# added this to both users

alias ls='ls --color=auto'

alias ll='ls -la --color=auto'

alias gs='git status'

alias update='sudo pacman -Syu'

alias grep='grep --color=auto'

12.
# and finally I started sddm after rebooting to start the GUI
reboot

systemctl start sddm



## Issues I faced along the road.

#It took me a while to do, in the beginning I was struglleing with partitioning the disk while using the commands fdisk and gdisk. cfdisk just works better and shows the amount of storage available making it very convienient.
#After that i had issues with using pacstrap since I hadn't formatted my root partition by that time but figured it out eventually.
#The first time i tried to reboot the system to then start adding users, I had errors with potential corrupted blocks, this is because I slighly decreased the size of the root table to create the boot table so the size wasn't completely updated, so I tried running resize2fs and e2fsck and fsck to try to fix the issue, I eventually did fix it and began adding users but reducing the size of the root partition had its other problems.
#before I had a boot table, I only had a uefi table and a root table at 10GB each, but i didnt need the uefi table so i deleted it but issues arised because I had already formatted it as fat32 so when i tried to add more space to the root table to install lqxt I just couldn't
#So as any person does when they realise that it was unfixable, I just deleted that vm and made a new one learning from my misakes and creating the 2 partitions that I needed and I encountered little to no problems from there on.
#In conclusion: it was a pretty difficult install all in all since I've never done it before, but with the help of the arch linux wiki, chatgpt, and other internet sources I managed to pull through and complete it.
