---
title: "The Elastic NAS: Part 3 - MooseFS and the first node"
date: 2020-09-20T16:07:37+08:00
Categories: [storage]
Tags: ['storage','nas','elasticnas','moosefs']
draft: false
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

Finally, I created a MooseFS user, then gave it ownership of the MooseFS data directory.

{{< highlight plaintext >}}
wings@blinky:~$ sudo useradd --no-create-home --shell /bin/false mfs
wings@blinky:~$ sudo chown -R mfs:mfs /var/lib/mfs/
{{< / highlight >}}

## Master
The first service I needed to configure was the MooseFS master. The master is responsible for looking after the metadata of the cluster and overseeing its operations.

First, I set up the configuration files for the master:
{{< highlight plaintext >}}
# Change to the MooseFS configuration directory
wings@blinky:~$ cd /etc/mfs/
# Copy the example configurations into place
wings@blinky:/etc/mfs$ sudo cp mfsmaster.cfg.sample mfsmaster.cfg
wings@blinky:/etc/mfs$ sudo cp mfsexports.cfg.sample mfsexports.cfg
{{< / highlight >}}

Since there was no metadata file yet, I needed to create one for the master to start with. I used the blank example provided. (You should only have to do this once. On an operational cluster, this command will wipe the metadata clean, so be careful!)
{{< highlight plaintext >}}
sudo cp /var/lib/mfs/metadata.mfs.empty /var/lib/mfs/metadata.mfs
{{< / highlight >}}

Now I could start and enable the master service.
{{< highlight plaintext >}}
wings@blinky:/etc/mfs$ sudo systemctl enable moosefs-master && sudo systemctl start moosefs-master
{{< / highlight >}}

## Chunkserver
Next up, I had to configure the MooseFS chunkserver service. 

First up, I copied the example configuration files into place.

{{< highlight plaintext >}}
# Change to the MooseFS configuration directory
wings@blinky:~$ cd /etc/mfs/
# Copy the example configurations for the chunkserver
wings@blinky:/etc/mfs$ sudo cp mfschunkserver.cfg.sample mfschunkserver.cfg
wings@blinky:/etc/mfs$ sudo cp mfshdd.cfg.sample mfshdd.cfg
{{< / highlight >}}

Then I had to make two changes. First off, within the main chunkserver config file...
{{< highlight plaintext >}}
wings@blinky:/etc/mfs$ sudo nano mfschunkserver.cfg
{{< / highlight >}}

I had to set the following option - "MASTER_HOST = 10.1.1.201".

The second change was to give the chunkserver a list of its bricks (or in this case, brick).

It's possible to point it straight at the brick's mountpoint - /mfsbrick/ - but I prefer to create a folder within the brick and tell MooseFS to use that instead. This way, if the brick isn't mounted for whatever reason, the chunkserver will fail to start rather than ending up in a weird state.

{{< highlight plaintext >}}
# Create a folder for the chunkserver to use
wings@blinky:~$ sudo mkdir /mfsbrick/moosebox && sudo chown mfs:mfs /mfsbrick/moosebox
# Edit the list of bricks
wings@blinky:~$ sudo nano /etc/mfs/mfshdd.cfg
# Add the following line and save it
/mfsbrick/moosebox
{{< / highlight >}}

Start and enable the chunkserver service.
{{< highlight plaintext >}}
wings@blinky:~$ systemctl enable moosefs-chunkserver && systemctl start moosefs-chunkserver
{{< / highlight >}}

## Web interface
The final service was “moosefs-cgiserv”, which provides a web interface for MooseFS. Unlike the other services, it doesn’t require any configuration, so just starting it was enough.

{{< highlight plaintext >}}
wings@blinky:~$ systemctl start moosefs-cgiserv
{{< / highlight >}}

Then I navigated to http://10.1.1.201:9425/mfs.cgi. As we’re not using the default DNS name of “mfsmaster”, I put “10.1.1.201” into the “Input your DNS master name” field and clicked the “Try it !!!” button.

The front page of the MooseFS web interface popped up.
![The MooseFS front page](/img/2020-09-20-moosefs-front.png)

I clicked on the Servers tab, and could see that it had 1 chunkserver connected. It looked roughly like this (ignore the used space):
![The MooseFS servers tab](/img/2020-09-20-moosefs-servers.png)

## Mounting the MooseFS filesystem
With all the components configured and started, the filesystem was ready for use.

{{< highlight plaintext >}}
# Create a mountpoint for the MooseFS filesystem.
mkdir /mnt/mfs/
# Mount it for the first time
wings@blinky:~$ sudo mfsmount -H 10.1.1.201 /mnt/mfs/
mfsmaster accepted connection with parameters: read-write,restricted_ip,admin ; root mapped to root:root
# Check it out
wings@blinky:~$ df -h /mnt/mfs/
Filesystem           Size  Used Avail Use% Mounted on
mfs#10.1.1.201:9421  3.7T     0  3.7T  1% /mnt/mfs
{{< / highlight >}}

Success! It worked.

## Initial testing

As a first test, I used Git to check out a copy of the MooseFS repository.

{{< highlight plaintext >}}
wings@blinky:~$ cd /mnt/mfs/
wings@blinky:/mnt/mfs$ sudo git clone https://github.com/moosefs/moosefs.git
Cloning into 'moosefs'...
remote: Enumerating objects: 633, done.
remote: Counting objects: 100% (633/633), done.
remote: Compressing objects: 100% (432/432), done.
remote: Total 10355 (delta 442), reused 351 (delta 192), pack-reused 9722
Receiving objects: 100% (10355/10355), 5.14 MiB | 3.91 MiB/s, done.
Resolving deltas: 100% (8766/8766), done.
Updating files: 100% (434/434), done.
{{< / highlight >}}

After doing so, 468 chunks appeared both on the frontpage and in the Servers tab.

![The MooseFS servers tab, now with chunks!](/img/2020-09-20-moosefs-servers-chunks.png)

On the frontpage the new chunks were listed in the "chunk matrix" table.

![The MooseFS "chunk matrix"](/img/2020-09-20-moosefs-chunk-matrix.png)

Within MooseFS, there's the concept of goals - a setting which defines how many copies a given file or folder you want to aim for. Each chunkserver can only contribute one copy. The default is "2", so the chunks I had just created had a goal of 2. As there's only one chunkserver at this stage, they are listed as "undergoal"/"endangered". I will go into more detail about goals in a later post.

## Client
I had a working MooseFS filesystem, but when mounting it I still needed to specify a mountpoint and tell the MooseFS client which master to connect to. With a final bit of configuration, I could set some defaults for mounting MooseFS via the client configuration.

{{< highlight plaintext >}}
# Change to the MooseFS configuration directory
wings@blinky:~$ cd /etc/mfs/
# Copy the example configurations for the client
wings@blinky:/etc/mfs$ sudo cp mfsmount.cfg.sample mfsmount.cfg
# Edit the client configuration file
wings@blinky:~$ sudo nano /etc/mfs/mfsmount.cfg
# Add the following lines and save it
/mnt/mfs
mfsmaster=10.1.1.201
{{< / highlight >}}

With the configuration in place, I could now mount MooseFS by just calling mfsmount:
{{< highlight plaintext >}}
wings@blinky:~$ sudo mfsmount
mfsmaster accepted connection with parameters: read-write,restricted_ip,admin ; root mapped to root:root
{{< / highlight >}}

## Automatically mounting MooseFS
I finished off the single-node cluster by setting up an entry in /etc/fstab to automatically mount MooseFS on boot:
{{< highlight plaintext >}}
# Edit the filesystem table
wings@blinky:~$ sudo nano /etc/fstab
# Add this line
mfsmaster: /mnt/mfs moosefs defaults,mfsdelayedinit 0 0
{{< / highlight >}}

Upon running mount -a, I could see that MooseFS was successfully auto-mounted.
{{< highlight plaintext >}}
wings@blinky:~$ sudo mount -a
wings@blinky:~$ df -h /mnt/mfs/
Filesystem           Size  Used Avail Use% Mounted on
mfs#10.1.1.201:9421  3.7T     0  3.7T  1% /mnt/mfs
{{< / highlight >}}

## Benchmarking
I used the same quick-and-dirty DD command from Part 2 so we could see the performance impact of using MooseFS instead of writing directly to a local hard drive.

{{< highlight plaintext >}}
root@blinky:/mnt/mfs# dd if=/dev/zero of=./largefile bs=1M count=1024
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 6.95196 s, 154 MB/s
{{< / highlight >}}

And I used a similar command to check the read speeds…

{{< highlight plaintext >}}
root@blinky:/mnt/mfs# dd if=./largefile of=/dev/null
2097152+0 records in
2097152+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 11.2736 s, 95.2 MB/s
{{< / highlight >}}

Write speeds of 154MB/s and read speeds of 95.2MB/s. While these results weren’t quite as fast as the local drive, they are both faster than the Gigabit networking of the unit with all overhead taken into account.

## The decision
Having tested MooseFS on the HC2, I was sold on the concept. I pulled the trigger and ordered 3 more HC2s, this time directly from their manufacturer, HardKernel.

![The MooseFS "chunk matrix"](/img/2020-09-20-hardkernel-hc2-order.png)

Join us next time, where we’ll expand to 3 more nodes and talk about how goals work.
