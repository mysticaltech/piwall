# Changing the videos and splash screens

### Changing the video

Place test.h264 in the 'Untitled' volume, available on the server image.

### Changing the splash screen

Place splash.jpg in the 'Untitled' volume on any of the screens. It will take 2 full boots for the screen to be fully updated.

# Setup from premade images

There are three images - one for the server, one for the left screen, one for the right screen

### install the image

    tar xf piwall-server.tar.gz # see piwall-left.tar.gz and piwall-right.tar.gz for the clients
    sudo diskutil unmount /dev/rdisk1s1
    sudo dd bs=16m if=piwall.img of=/dev/rdisk1
    sudo diskutil unmount /dev/rdisk1s1
    sudo diskutil eject /dev/rdisk1

# Setting up from scratch

Here are all the steps required to get from a blank sd card to this working setup.

### Install raspbian

Download and burn raspbian onto an sd card, from your local machine

    curl -L http://downloads.raspberrypi.org/raspbian_latest -o raspbian.zip
    unzip raspbian.zip
    sudo diskutil unmount /dev/rdisk1s1
    sudo dd bs=1m if=2014-06-20-wheezy-raspbian.img of=/dev/rdisk1
    sudo diskutil unmount /dev/rdisk1s1
    sudo diskutil eject /dev/rdisk1

### Configure raspbian

Use raspi-config to change the keyboard layout and enable ssh.

### Update the distribution

    sudo apt-get update
    sudo apt-get upgrade
    sudo rpi-update

### Add a shared partition

Add a 0B (W95 FAT32) partition where the free space was

    sudo cfdisk /dev/mmcblk0
    sudo reboot

Mount as /shared on boot

    sudo apt-get install dosfstools
    sudo mkfs.msdos /dev/mmcblk0p3
    sudo mkdir /shared
    sudo chown pi:pi /shared
    sudo sh -c "echo '/dev/mmcblk0p3 /shared vfat auto,rw,user,users,exec,noatime,uid=1000,gid=1000,umask=000 0 0' >> /etc/fstab"
    sudo mount -a

### Clone this respository

    git clone https://github.com/jedahan/piwall.git
    cd piwall

### Install the shared files, and init scripts

    cp shared/* /shared/
    sudo cp -r init/* /etc/init.d

### Install a splash screen & the splash screen updater

    sudo apt-get install fbi
    sudo update-rc.d splashscreen defaults
    sudo update-rc.d updatesplash defaults

### Hide the boot text

    sudo sed -e 's/tty1/tty3/' -e 's/$/ loglevel=3 vt.global_cursor_default=0 logo.nologo/' -i /boot/cmdline.txt

### Install piwall applications and default wall config

These packages were downloaded from [dl.piwall.co.uk](dl.piwall.co.uk)

    sudo dpkg -i packages/*

Make sure we can convert video

    sudo apt-get install libav-tools

Install the default config

    sudo cp .pi* /root/

### Setup networking

Install the network changing scripts

    sudo cp /etc/network/interfaces{,.bak}
    sudo cp network/interfaces* /etc/network
    echo 'alias lan=/home/pi/piwall/network/localnetwork.sh' >> ~/.bashrc
    echo 'alias wan=/home/pi/piwall/network/globalnetwork.sh' >> ~/.bashrc
    source ~/.bashrc

Make sure to choose different octets for each pi

    nano network/interfaces.local

Now you can switch just by doing `lan` or `wan`

    lan

# Enable the correct init scripts

### For each tile

    sudo update-rc.d piwalltile defaults

### For the Server

    sudo update-rc.d piwallserver defaults
