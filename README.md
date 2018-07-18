# Make-Raspberry-Pi-3-Boot-From-USB    (REFERENCED from www.makeuseof.com)
DIY How to Make Raspberry Pi 3 Boot From USB

How to Make Raspberry Pi 3 Boot From USB
Get Started: Install Raspbian and Add New Files

sudo apt-get update
sudo BRANCH=next rpi-update

This update delivers the two files into the /boot directory. With the files downloaded, proceed to
enable the USB boot mode with:

echo program_usb_boot_mode=1 | sudo tee -a /boot/config.txt

This command adds the program_usb_boot_mode=1 instruction to the end of the config.txt file.

You’ll need to reboot the Pi once this is done.
Next step is to check that the OTP — one-time programmable memory — has been changed. Check
this with:

vcgencmd otp_dump | grep 17:

If the result is representative of the address 0x3020000a (such as 17:3020000a) then all is good so far.
At this stage, should you wish to remove the program_usb_boot_mode=1 line from the config.txt file,
you can. The Pi is now USB boot-enabled, and you might wish to use the same microSD card in another
Raspberry Pi 3, with the same image, so removing the line is a good idea.

This is easily done by editing config.txt in nano:

sudo nano /boot/config.txt

Delete or comment out the corresponding line (with a preceeding #).

Prepare Your USB Boot Device

Next, connect a formatted (or ready-to-be-deleted) USB stick into a spare port on your Raspberry Pi
3. With this inserted, we’ll proceed to copy the OS across.
Begin by identifying your USB stick, with the lsblk command.


In this example, the SD card is mmcblk0 while the USB stick is sda (it’s formatted partition is sda1). If
you have other USB storage devices connected the USB stick might be sdb, sdc, etc. With the name
of your USB stick established, unmount the disk and use the parted tool to create a 100 MB partition
(FAT32) and a Linux partition:

sudo umount /dev/sda
sudo parted /dev/sda


At the (parted) prompt, enter:

mktable msdos

You might be informed that the disk is otherwise engaged. If so, select Ignore, then note the warning
instructing you that the data on the disk will be destroyed. As explained earlier, this should be a disk
that you’re happy to delete or format, so agree to this.
If you run into any problems here, you might need to switch to the desktop (either manually, or over
VNC) and confirm the disk is unmounted, before entering the mktable msdos command in a
windowed command line.
Proceed in parted with the following:

mkpart primary fat32 0% 100M
mkpart primary ext4 100M 100%
print


This will output some information concerning disk and the new partitions. Proceed to exit parted
with Ctrl + C, before creating the boot filesystem, and the root filesystem:

sudo mkfs.vfat -n BOOT -F 32 /dev/sda1
sudo mkfs.ext4 /dev/sda2

You then need to mount the target filesystems, before copying your current Raspbian OS to the USB
device.

sudo mkdir /mnt/target
sudo mount /dev/sda2 /mnt/target/
sudo mkdir /mnt/target/boot
sudo mount /dev/sda1 /mnt/target/boot/
sudo apt-get update; sudo apt-get install rsync
sudo rsync -ax --progress / /boot /mnt/target

That last one is the final command that copies everything over, and so will take a while to complete.


Next, you need to refresh the SSH host keys, to maintain the connection with the reconfigured
Raspberry Pi after an imminent reboot:


cd /mnt/target
sudo mount --bind /dev dev
sudo mount --bind /sys sys
sudo mount --bind /proc proc
sudo chroot /mnt/target
rm /etc/ssh/ssh_host*
dpkg-reconfigure openssh-server
exit
sudo umount dev
sudo umount sys
sudo umount proc

Note that after sudo chroot (the fifth command above) you’re switching to root, so the user will
change from pi@raspberrypi to root@raspberrypi until you enter exit on line 8.
Prepare for Rebooting From USB!
Just a few more things to sort out before your Raspberry Pi is ready to boot from USB. We need to
edit cmdline.txt again from the command line with:

sudo sed -i "s,root=/dev/mmcblk0p2,root=/dev/sda2," /mnt/target/boot/cmdline.txt


Similarly, the following change needs to be made to fstab:

sudo sed -i "s,/dev/mmcblk0p,/dev/sda," /mnt/target/etc/fstab

You’re then ready to unmount the filesystems before shutting down the Pi:


cd ~
sudo umount /mnt/target/boot
sudo umount /mnt/target
sudo poweroff


Note that this uses the new poweroff command as an alternative to shutdown.
When the Pi has shutdown, disconnect the power supply before removing the SD card. Next,
reconnect the power supply — your Raspberry Pi should now be booting from the USB device!




