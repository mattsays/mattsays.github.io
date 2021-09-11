---
author: Mattia Sicoli
title: ManjaroBook
date: 2021-09-10
description: "My journey into installing Manjaro on a MacBook Pro"
tags: ["macbook-pro", "apple", "linux", "manjarobook", "linux-on-macbook", "manjaro"]
---

## How the journey started

![ManjaroBook](/manjaro_book/manjaro_view.png)

One of my favorite Linux distros overall is without any doubt _[Manjaro](https://manjaro.org/)_, a Linux distribution which derivates from ArchLinux.

Some days ago, annoyed by the **restricted Apple macOS**, I decided to install **Manjaro on my MacBook Pro 13' 2017 base model**. This decision was also made because of the continuous and loud sound of the fan; Yes fan and not fans because this particular model has only one fan to cool down the entire computer. 

## Preliminary Steps

Before actually starting installing Manjaro, I had to do a few preliminary steps:
- Installing a new boot manager for dual booting with macOS and to properly start all hardware internal components.
- Get an external storage to install Manjaro into it (This could be seen as an easy step, but it's not 😂).
- Search useful resources to start diving into the `linux on macbook pro` world.

### Installing a new boot manager: rEFInd

As the title suggests, `rEFInd` is a really cool boot manager, not only used in Apple systems but also used in usual pc systems.

It is very easy to install; You just have to follow this [guide](https://www.rodsbooks.com/refind/installing.html#installsh) that makes you install rEFInd with a handy script. What this script basically does is creating a new folder called `refind` in an EFI partition called `ESP` in your main Macintosh drive which contains all the necessary files.

### Finding a drive for Manjaro

This step as I said could be undervalued. In my case, at first, I was using a WesternDigital Passport SSD. It is an external USB-C drive which should work without any big issues. And yeah, that's what I thought. However, Linux is sometimes weird, and It can't use my SSD for starting up Manjaro.  
Well, actually, this issue happens only when starting a Linux distro that runs kernel version 5.12 or later.   
I discovered this by trying to install other distros like Fedora or Ubuntu. These distros have in common that use earlier kernel versions. The problem is related to all USB 3.0 drives, so to get it working you need to use an USB 2.0 device.   
So in the end I had to not use my WesternDigital SSD and rely on a [Samsung SSD with 512 GB of storage](https://letmegooglethat.com/?q=samsung+ssd+850+evo) via a USB 2.0 cable.


### A useful resource for MacBook Linux users

In order to find all the required drivers and modules to make Apple hardware work with Linux.
Luckily, there is a complete guide to do so on [GitHub](https://github.com/Dunedan/mbp-2016-linux) thanks to a guy called Dunedan.

### Actual distro installation

Now that everything has been set, I could start flashing my USB stick with a fresh new Manjaro installation image.

After that, using rEFInd I booted into the installation system 

![rEFInd](/manjaro_book/refind.jpeg)

Hopefully, It starts without any problem like has happened to me. 
> * Older MacBooks may require additional parameters to make it boot.

After doing the first configuration steps when reaching the installation drive step, I had to not just use the `Erase Disk` method. I chose the custom partitioning option.

Choosing that option means that you have to create all the necessary partitions by yourself.
So I made the following ones:
- An _fat32_ EFI partition. This one in particular needs to be mounted on '**/boot**' instead of '_/boot/efi_ ' and needs the `boot` flag 
- An _ext4_ partition mounted on '**/**'
- A _linuxswap_ partition with no hibernation because currently Linux on MacBook doesn't allow a satisfying [suspend/resume functionality](https://github.com/Dunedan/mbp-2016-linux#suspend--hibernation)

After this partition setup, I just followed the installation process and when it finished, I had to label my EFI partition by doing:
```
fatlabel /dev/your_boot_partition MANJARO
```

Finally, before being able to start our freshly new Linux distro, I had to modify rEFInd, so it has a custom new boot entry that can start Manjaro correctly.

To do that, You just have to find the correct partition name by executing `blkid` and it should show you a device called `/dev/nvme0n1p1` or something like that. This is our refind partition. To mount it, just type:
```
mkdir /mnt/APPLE_EFI
mount /dev/nvme0n1p1 /mnt/APPLE_EFI
```  

After that, Modify the file `refind.conf` situated in `/mnt/APPLE_EFI/EFI/refind/` and add this entry:
```
menuentry "Manjaro" {
    icon     /EFI/refind/icons/os_manjaro.png
    volume   "MANJARO"
    loader   /vmlinuz-5.13-x86_64
    initrd   /initramfs-5.13-x86_64.img
    options  "root=PARTUUID=YOUR_ROOT_PARTITION_UUID resume=PARTUUID=YOUR_SWAP_PARTITION_UUID rw add_efi_memmap"
    submenuentry "Boot using fallback initramfs" {
        initrd /boot/initramfs-5.13-x86_64-fallback.img
    }
    submenuentry "Boot to terminal" {
        add_options "systemd.unit=multi-user.target"
    }
}
```
Replace _YOUR_ROOT_PARTITION_UUID_ and _YOUR_SWAP_PARTITION_UUID_ with your partition's UUIDs found by executing `blkid`

I also changed some settings situated in the same file:
```
enable_mouse # This just to be able to easily move in the menu
spoof_osx_version 10.9 # We need to make sure that our mac thinks we are booting into macOs
```

Now we should be all set and can hopefully boot our Manjaro installation. Just reboot your system, remove your USB stick, and a menu entry called "Manjaro" should appear. 
>* Make sure to not boot from the kernel directly, just use our custom menu entry because avoiding doing that, It will boot a Linux kernel without providing the necessary boot and swap partition's UUIDs.

![Manjaro installation completed](/manjaro_book/manjaro_installation_complete.png)

### Post install configurations

Hooray, you have finally booted your so much desired distro Manjaro!

But, hey, you have some more work to do 🥲.  
You have to install some additional drivers that may not be included in your Manjaro installation.
In my case, MacBook keyboard and touchpad, Wi-Fi, graphics, etc. are working out of the box.
The things that were not working, were: Sound input and output, Bluetooth, FaceTime camera and suspend/resume functionality.

#### Sound

To make sound working following the guide provided by Dunedan, I had to just install [this driver](https://github.com/davidjo/snd_hda_macbookpro).
And after a reboot, only sound output worked because as Dunedan said in the guide:

> With the MacBookPro14,1 the internal audio output is working, however the internal audio input is not working.

Despite this issue, I am satisfied because I use an external Samson Meteor Microphone as my default sound input device.

#### Bluetooth

Also, Bluetooth required the installation of a [driver](https://github.com/leifliddy/macbook12-bluetooth-driver) and after another reboot everything worked correctly

#### FaceTime camera

This one also required another [driver](https://github.com/patjak/bcwc_pcie/wiki/Installation#get-started-on-arch) that can be installed without any problem.

This one doesn't require any git clone or so ever; I had to just install it with `pacman`. 

#### Resume/suspend

Unfortunately, this isn't working for almost every MacBook. I've tried to do:
```
echo 0 > /sys/bus/pci/devices/0000\:01\:00.0/d3cold_allowed
```
But it didn't work, maybe will work for you. It should be executed at every startup.

### Bonus: fans

The one of the reasons I wanted to switch to Manjaro is also a better fans management system. However, right now there is only this [software](https://github.com/linux-on-mac/mbpfan#arch-linux) that adjusts fans speed based on the temperature of the CPU. It works good for doing not very intensive tasks, but when you use more programs at once like IntelliJ IDEA and other programs, CPU Frequency is stuck at 400mhz. This is because MacBook internal systems also not only checks for internal CPU temperature to make it work at full power, it also checks for other temperatures values that can be found in `/sys/devices/platform/applesmc.768`.  
So to solve this issue I am thinking of making a new software in rust that can solve this issue. In this way, I can also learn a new programming language and learn new stuff 😁.

### Conclusion

After doing all of these steps, you will be able to enjoy the beauty of the Linux world and specifically the beauty of the ArchLinux world 😚.

I decided to write this article to encourage everyone who is trying to install Linux on their MacBook. Before actually started booting Manjaro without any problems I had to struggle a lot and figure out some weird problems like the SSD one, but after all I am proud of what I have done! 🎊🎊🎊