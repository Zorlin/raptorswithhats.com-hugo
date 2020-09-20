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

{{< highlight plaintext >}}
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

{{< highlight plaintext >}}
wings@blinky:~$ wget https://github.com/moosefs/moosefs/archive/v3.0.114.tar.gz
2020-09-20 12:04:18 (989 KB/s) - ‘v3.0.114.tar.gz’ saved [1227079]
wings@blinky:~$ tar xf v3.0.114.tar.gz
{{< / highlight >}}

Then I compiled it.

{{< highlight plaintext >}}
wings@blinky:~$ cd moosefs-3.0.114/
wings@blinky:~/moosefs-3.0.114$ ./linux_build.sh
{{< / highlight >}}

Compiling it took approximately 5 minutes and 45 seconds. Typically at this point you would run “make install” to install it, but I had a different idea…

There’s an excellent tool called “checkinstall” which you can run instead of running “make install”. It’s essentially a wrapper that installs everything as though you ran “make install”, but uses those changes to magically build a (basic) .deb package from them. It's super handy, especially since it means you can cleanly uninstall your newly-built software as if it was a normal .deb package.

I installed and ran it, and it went to work.

{{< highlight plaintext >}}
wings@blinky:~/moosefs-3.0.114$ sudo apt install checkinstall
wings@blinky:~/moosefs-3.0.114$ sudo checkinstall

checkinstall 1.6.3, Copyright 2010 Felipe Eduardo Sanchez Diaz Duran
           This software is released under the GNU GPL.


The package documentation directory ./doc-pak does not exist.
Should I create a default set of package docs?  [y]: n

Please write a description for the package.
End your description with an empty line or EOF.
>> moosefs
>>

*****************************************
**** Debian package creation selected ***
*****************************************

This package will be built according to these values:

0 -  Maintainer: [ root@blinky ]
1 -  Summary: [ moosefs ]
2 -  Name:    [ moosefs ]
3 -  Version: [ 3.0.114 ]
4 -  Release: [ 1 ]
5 -  License: [ GPL ]
6 -  Group:   [ checkinstall ]
7 -  Architecture: [ armhf ]
8 -  Source location: [ moosefs-3.0.114 ]
9 -  Alternate source location: [  ]
10 - Requires: [  ]
11 - Recommends: [  ]
12 - Suggests: [  ]
13 - Provides: [ moosefs ]
14 - Conflicts: [  ]
15 - Replaces: [  ]

Enter a number to change any of them or press ENTER to continue:

Installing with make install...

(A lot of output later...)

**********************************************************************

 Done. The new package has been installed and saved to

 /home/wings/moosefs-3.0.114/moosefs_3.0.114-1_armhf.deb

 You can remove it from your system anytime using:

      dpkg -r moosefs

**********************************************************************

{{< / highlight >}}

At this stage, MooseFS was installed and I had a nice package I could use for future purposes. Notably, this install included all the MooseFS services, but they are all disabled by default anyways.

Finally, I created a MooseFS system user.

{{< highlight plaintext >}}
wings@blinky:~$ sudo useradd --no-create-home --shell /bin/false mfs
{{< / highlight >}}

## Master
The first service I needed to configure was the MooseFS master. The master is responsible for looking after the metadata of the cluster and overseeing its operations. There are three configuration files that are important for the master service, but only one required any changes.

{{< highlight plaintext >}}
# Change to the MooseFS configuration directory
wings@blinky:~$ cd /etc/mfs/
# Copy the example configurations into place

{{< / highlight >}}
