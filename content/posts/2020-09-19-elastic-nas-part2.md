---
title: "The Elastic NAS: Part 2 - Setting up the ODROID HC2"
date: 2020-09-19T12:09:37+08:00
Categories: [storage]
Tags: ['storage','nas','elasticnas','moosefs']
draft: false
---

After ordering the HC2 along with a power supply and case from JiffyShop, I went to PLE Computers and bought a cheap microSD card, an Ethernet cable and a WD RED 4TB hard drive. Once everything arrived, it was time to get started.

Physically, setting up the HC2 couldn’t be simpler. I carefully slotted the hard drive into place, screwed it in, then installed the case around the unit. Once that was done, I needed to prepare and install the microSD card after loading it with an appropriate image.

## Getting an OS image
I decided to start with the official Ubuntu images for the ODROID HC2 (rather than something like Armbian, which I may try later). The ODROID HC2 is a stripped-down version of the ODROID XU4, so it uses [the same images as the XU4](https://wiki.odroid.com/odroid-xu4/os_images/os_images). As there is no video output, however, it’s usually best to go with the “minimal” images rather than the ones that come with a graphical desktop.

I went with the latest Ubuntu 20.04 image, making sure to grab the minimal version (as well as its corresponding md5sum file for an integrity check).

{{< highlight bash >}}
rain:HC2 wings$ wget https://odroid.in/ubuntu_20.04lts/XU3_XU4_MC1_HC1_HC2/ubuntu-20.04.1-5.4-minimal-odroid-xu4-20200812.img.xz
rain:HC2 wings$ wget https://odroid.in/ubuntu_20.04lts/XU3_XU4_MC1_HC1_HC2/ubuntu-20.04.1-5.4-minimal-odroid-xu4-20200812.img.xz.md5sum
{{< / highlight >}}

Then I checked the md5 hash of the image, comparing it to the md5sum file.

{{< highlight bash >}}
rain:HC2 wings$ md5 ubuntu-20.04.1-5.4-minimal-odroid-xu4-20200812.img.xz
MD5 (ubuntu-20.04.1-5.4-minimal-odroid-xu4-20200812.img.xz) = 205792423ae785b28b7ed26b97f3d813
rain:HC2 wings$ cat ubuntu-20.04.1-5.4-minimal-odroid-xu4-20200812.img.xz.md5sum
205792423ae785b28b7ed26b97f3d813  ubuntu-20.04.1-5.4-minimal-odroid-xu4-20200812.img.xz
{{< / highlight >}}

Finally, I unpacked the image using unxz.

{{< highlight bash >}}
rain:HC2 wings$ unxz ubuntu-20.04.1-5.4-minimal-odroid-xu4-20200812.img.xz
{{< / highlight >}}

## Burning the image
There are plenty of methods you can use to burn a disk image to an SD card, but I went with the Raspberry Pi Imager. It’s free and open source software, and it’ll even verify the image was written correctly at the end. Download it, pick the image you just downloaded, pick your SD card and hit “write” and off it goes. It took about 5 minutes to burn and then check the SD card.

![Burning an image](/img/2020-09-19-hc2-sdcard.png)

## First boot
I gently installed the microSD card into the HC2, plugged in the Ethernet cable, then turned it on. After a couple of minutes, the HC2 showed up in my router as a new device, with a default hostname of “odroid” and a DHCP-provided address.

![The HC2 showing on my router](/img/2020-09-19-hc2-router.png)

This was all I needed to log into it for the first time (using the default password "odroid").

{{< highlight bash >}}
rain:~ wings$ ssh root@10.1.1.6
root@10.1.1.6's password:
Welcome to Ubuntu 20.04.1 LTS (GNU/Linux 5.4.58-211 armv7l)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Last login: Wed Aug 12 20:57:48 2020
root@odroid:~#
{{< / highlight >}}

## Initial OS setup
First things first, I needed to set a hostname. I decided that the machines in the cluster would be named after the (main) ghosts in Pacman - Blinky, Pinky, Inky and Clyde. Being the first node, this would be "Blinky".

{{< highlight bash >}}
root@odroid:~# sudo hostnamectl set-hostname blinky
{{< / highlight >}}

Then, I set a proper root password.

{{< highlight bash >}}
root@odroid:~# passwd
New password:
Retype new password:
passwd: password updated successfully
{{< / highlight >}}

Once that was done, I created my own user and added it to the sudo group.
{{< highlight bash >}}
root@odroid:~# adduser wings
Adding user `wings' ...
Adding new group `wings' (1000) ...
Adding new user `wings' (1000) with group `wings' ...
Creating home directory `/home/wings' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for wings
Enter the new value, or press ENTER for the default
	Full Name []: Wings
	Room Number []:
	Work Phone []:
	Home Phone []:
	Other []:
Is the information correct? [Y/n] y
root@odroid:~# usermod -aG sudo wings
{{< / highlight >}}

I ran some quick package upgrades

{{< highlight bash >}}
root@odroid:~# apt update && apt upgrade
{{< / highlight >}}

Finally, I went into my router and created a static DHCP lease, permanently giving Blinky the IP of "10.1.1.201".

![Creating a static lease](/img/2020-09-19-hc2-router-staticlease.png)

A quick reboot later, the HC2 had picked up the address and was ready for action.

{{< highlight bash >}}
root@odroid:~# reboot
Connection to 10.1.1.6 closed by remote host.
Connection to 10.1.1.6 closed.
rain:~ wings$ ssh root@10.1.1.201
root@10.1.1.201's password:
root@blinky:~#
{{< / highlight >}}

## Setting up the hard drive
Now that the basic OS stuff was out of the way, I needed to set up the hard drive. Fortunately, on the HC2 the hard drive always shows up as “/dev/sda”, which makes things pretty easy. Since the MooseFS developers recommend using the XFS filesystem for bricks, I went with XFS. 

For simplicity, I didn’t create any partitions, opting to lay the new XFS filesystem directly onto the disk and mount it. If you didn’t know that was possible, you’re one of [today’s lucky 10,000](https://xkcd.com/1053/). Anyways, here’s the procedure.

{{< highlight bash >}}
# Install the XFS utilities and drivers
root@blinky:~# apt install xfsprogs
# Lay down an XFS filesystem
root@blinky:~# mkfs.xfs /dev/sda
# Create a mountpoint for the newly created brick
root@blinky:~# mkdir /mfsbrick/
# Mount the newly created brick
root@blinky:~# mount /dev/sda /mfsbrick/
{{< / highlight >}}

(Sidenote: “Bricks” here refers to the data storage disks used by MooseFS chunkservers. It’s not MooseFS terminology – I think I may have even stolen it from GlusterFS – but to me, it’s less confusing than just calling them “disks”).

I checked out the newly mounted brick:
{{< highlight bash >}}
root@blinky:~# df -h  
Filesystem      Size  Used Avail Use% Mounted on  
/dev/sda        3.7T   89M  3.6T   1% /mfsbrick
{{< / highlight >}}

And it all looked healthy. Time to benchmark it!

## Benchmarking
I used a quick-and-dirty DD command to get a basic idea of the write speed of the drive.
{{< highlight bash >}}
root@blinky:/mfsbrick# dd if=/dev/zero of=./largefile bs=1M count=1024
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 5.79871 s, 185 MB/s
{{< / highlight >}}
I got a result of roughly 185MB/s, and in a similar test found that read speeds were also around 180MB/s. This was more than enough to saturate the Gigabit networking of the unit. I could also use these results to figure out the overhead of MooseFS once everything was up and running.

## Automatically mounting the brick
Now that everything was working, it was time to setup the brick so that it would mount on boot.

I added this line to my /etc/fstab file:

{{< highlight bash >}}
/dev/sda /mfsbrick xfs nodev,noatime,nodiratime,largeio,inode64 0 2
{{< / highlight >}}

You may notice the options "nodev,noatime,nodiratime,largeio,inode64" - this applies some tuning. I'll elaborate in a later post.

With these changes made, the brick automatically came back on boot.

{{< highlight bash >}}
root@blinky:~# df -h /mfsbrick
Filesystem      Size  Used Avail Use% Mounted on  
/dev/sda        3.7T   89M  3.6T   1% /mfsbrick
{{< / highlight >}}

Tune in next time, where we'll install MooseFS and get the first node up and running.
