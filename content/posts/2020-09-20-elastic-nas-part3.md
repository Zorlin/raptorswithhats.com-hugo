---
title: "The Elastic NAS: Part 3 - MooseFS and the first node"
date: 2020-09-20T10:07:37+08:00
Categories: [storage]
Tags: ['storage','nas','elasticnas','moosefs']
draft: true
---

There are a few different ways to get MooseFS installed on the HC2. You can either use their “Raspbian” repository to install a version of the software compiled for Raspbian on the Raspberry Pi, or you can compile and install it yourself. I chose to compile it.

## Prerequisites
First things first, I installed the prerequisites needed to build and use MooseFS.

{{< highlight bash >}}
wings@blinky:~$ sudo apt install build-essential libpcap-dev zlib1g-dev libfuse3-dev pkg-config fuse3
{{< / highlight >}}

## World Go Boom
Then something weird happened. I saw this output while installing the prerequisites:

{{< highlight bash >}}
Processing triggers for initramfs-tools (0.136ubuntu6.3) ...
update-initramfs: Generating /boot/initrd.img-5.4.58-211
W: missing /lib/modules/5.4.58-211
W: Ensure all necessary drivers are built into the linux image!
depmod: ERROR: could not open directory /lib/modules/5.4.58-211: No such file or directory
depmod: FATAL: could not search modules: No such file or directory
cat: /var/tmp/mkinitramfs_CEJbdC/lib/modules/5.4.58-211/modules.builtin: No such file or directory
find: ‘/var/tmp/mkinitramfs_CEJbdC/lib/modules/5.4.58-211/kernel’: No such file or directory
depmod: WARNING: could not open modules.order at /var/tmp/mkinitramfs_CEJbdC/lib/modules/5.4.58-211: No such file or directory
depmod: WARNING: could not open modules.builtin at /var/tmp/mkinitramfs_CEJbdC/lib/modules/5.4.58-211: No such file or directory
{{< / highlight >}}

Upon investigation, things seemed “okay” and there were no missing package updates or badly installed packages. I wanted to check that the machine would still boot, so I rebooted and… nothing. 10 minutes later, the machine was unresponsive.

Remember how I mentioned Armbian earlier? At this point I decided I’d give it a go. I grabbed the latest Armbian Focal (Ubuntu) image for the HC2 and re-flashed the SD card with it.
## Armbian
Once I popped the microSD card back into Blinky, I ssh’d in with the default credentials (root : 1234) and was presented with the first-time setup of Armbian.

![Armbian first-time setup](/img/2020-09-20-hc2-armbian.png)

I then followed the same procedures as last time, including installing the MooseFS prerequisites above. I also applied an “optimized board configuration” supplied by Armbian’s developers.

I crossed my fingers and rebooted once more, and this time was met with success – I had a working system and was ready for MooseFS.

## Getting and compiling MooseFS
With all the setup out of the way, I went to the MooseFS releases page and downloaded the latest version (v3.0.114 at the time of this writing), then extracted it.
