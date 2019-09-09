---
title: "The Elastic NAS: Part 1 - The Beginning"
date: 2019-03-11T10:52:37+08:00
Categories: [storage]
Tags: ['storage','nas','elasticnas','moosefs']
draft: true
---

My first NAS was a QNAP 4-bay NAS which served for years as a place to store photos, videos and backups with relative ease. When I moved into the city, the NAS stayed behind to continue dutifully serving those roles, but I wanted a NAS of my own.

I knew I wanted a 4-bay NAS but I couldn't find a decent 4-bay unit for less than about $650 AUD. And most traditional NAS units were fairly underpowered. I wanted something beefier.

Having used MooseFS in the past, I also longed to have a storage cluster of my own. Eventually I realized I could combine these two projects and build a NAS for less than a traditional NAS would cost.

After looking at various options (such as a bunch of Raspberry Pis strapped to some portable hard drives) I discovered the ODROID HC2 by HardKernel, after reading an [interesting post](https://www.reddit.com/r/DataHoarder/comments/8ocjxz/200tb_glusterfs_odroid_hc2_build/) on Reddit. 

## Enter HC2
![ODROID HC2](/img/2019-03-11-hc2.jpg)

The HC2 is a small ARM board with an octa-core processor, 2GiB of RAM and - most importantly - SATA3 + Gigabit Ethernet. This board is attached to a large heatsink, which combined with the SATA port is designed to allow you to install a fullsize 3.5" hard drive. It's explicitly marketed for use as a NAS, and can be found fairly cheaply (around $105 AUD per unit including a power supply). 

Given all these traits, it sounded like the perfect platform on which to build my Storage Cluster. I could strap 4 of them together, and, combined with MooseFS, create an "Elastic NAS" that I could expand at will. 

I settled on a target of 4 nodes largely because I wanted to use a [fan shroud](https://www.thingiverse.com/thing:2982075) design I found on Thingiverse to add a 120mm fan to keep the whole thing cool.

![ODROID HC2 Fan Shroud](/img/2019-03-11-hc2-shroud.jpg)

## The Beginning

So I went ahead and ordered one node (and matching power supply + case) from an Australian reseller called JiffyShop, for around $155.

It arrived in the mail today, and I immediately went out to my local PC shop and bought some additional gear.

Shopping list:

- 16GB microSD card ($25)
- 4TB WD RED drive ($180)
- Network cable ($3)
- Wireless adapter for The Cube (the loungeroom PC) ($69)

## Getting started

First things first, I plugged the microSD card into my MacBook and went looking for images. I grabbed the latest Ubuntu 18.04 image from https://wiki.odroid.com/odroid-xu4/os_images/linux/ubuntu_4.14/ubuntu_4.14 and pulled it down with axel. As it was an xz archive, I needed unxz to decompress it.

{{< highlight bash >}}
rain:images wings$ unxz ubuntu-18.04-4.14-minimal-odroid-xu4-20181203.img.xz
{{< / highlight >}}

HardKernel (the manufacturer of the ODROID boards) recommends you use an open source utility called Etcher to burn the image to the SDcard, which I did. A few minutes later, I had my boot media ready. I plugged the microSD card into blinky, fired it up and... nothing. 

After a power cycle, however, it roared to life as as 10.1.1.66! Hooray! I logged in with the default user "root" and password "odroid", and set a new root password. I used the MAC address to create a static DHCP lease (for 10.1.1.201). Then I ran updates and upgrades, set the hostname (blinky) and rebooted.

There's a handy page on https://wiki.odroid.com/odroid-xu4/troubleshooting/shutdown_script about a known issue with the HC1/HC2 - bad parking behaviour on power-off - which is worth fixing. I grabbed the included script and installed it.

Now it was time for the star of the show: blinky's hard drive. Some grepping around in /dev indicated it lived at /dev/sda. This made things quite simple. Instead of partitioning it, I simply laid down an ext4 filesystem straight onto the disk and mounted it.

{{< highlight bash >}}
# Lay down an FS.   
root@blinky:~# mkfs.ext4 /dev/sda  
# Make our brick directory  
root@blinky:~# mkdir /mfsbrick
# Mount our brick as /mfsbrick/  
root@blinky:~# mount /dev/sda /mfsbrick/
{{< / highlight >}}

I checked out the newly mounted brick:

{{< highlight bash >}}
root@blinky:~# df -h  
Filesystem      Size  Used Avail Use% Mounted on  
/dev/sda        3.6T   89M  3.4T   1% /mfsbrick
{{< / highlight >}}

I then unmounted the disk and created an /etc/fstab entry:
{{< highlight fstab >}}
/dev/sda /mfsbrick ext4 nodev,noatime,nodiratime 0 2
{{< / highlight >}}

It should cleanly mount when you run "mount -a".

That worked, so the brick is ready to go. The next steps will be some basic benchmarking, setting up MooseFS and adopting this first brick, and some housekeeping. That's a job for tomorrow.

*This post is part of a series. If you're interested in reading the rest of it, click [here](/tags/elasticnas/).*
