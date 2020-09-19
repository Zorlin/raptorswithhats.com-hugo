---
title: "The Elastic NAS: Part 1 - Setting up the ODROID HC2"
date: 2020-09-19T12:09:37+08:00
Categories: [storage]
Tags: ['storage','nas','elasticnas','moosefs']
draft: true
---

After ordering the HC2 along with a power supply and case from JiffyShop, I went to PLE Computers and bought a cheap microSD card, an Ethernet cable and a WD RED 4TB hard drive. Once everything arrived, it was time to get started.

Physically, setting up the HC2 couldn’t be simpler. I carefully slotted the hard drive into place, screwed it in, then installed the case around the unit. Once that was done, I needed to prepare and install the microSD card after loading it with an appropriate image.

## Getting an OS image
I decided to start with the official Ubuntu images for the ODROID HC2 (rather than something like Armbian, which I may try later). The ODROID HC2 is a stripped-down version of the ODROID XU4, so it uses the same images as the XU4. As there is no video output, however, it’s usually best to go with the “minimal” images rather than the ones that come with a graphical desktop.

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
I gently installed the microSD card into the HC2, plugged in the Ethernet cable, then turned it on. After a couple of minutes, the HC2 showed up in my router as a new device, a default hostname of “odroid” and a DHCP-provided address.

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

Finally, I went into my router and created a static DHCP lease, permanently giving Blinky the IP of "10.1.1.201".

![Creating a static lease](/img/2020-09-19-hc2-router-staticlease.png)

A quick reboot later, the HC2 has picked up the address and is now ready for action.

{{< highlight bash >}}
root@odroid:~# reboot
Connection to 10.1.1.6 closed by remote host.
Connection to 10.1.1.6 closed.
rain:~ wings$ ssh root@10.1.1.201
root@10.1.1.201's password:
root@blinky:~#
{{< / highlight >}}

## Setting up the hard drive
