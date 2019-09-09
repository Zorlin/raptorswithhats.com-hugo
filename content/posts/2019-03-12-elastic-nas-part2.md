---
title: "The Elastic NAS: Part 2 - MooseFS"
date: 2019-03-12T14:25:58+08:00
Categories: [storage]
Tags: ['storage','nas','elasticnas','moosefs']
draft: true
---
The goal for today was simple - get MooseFS fully up and running on my first HC2 node and evaluate it.

Before I began setting up services etc, some minor tuning and tweaking was needed.

## Tuning and Tweaking

By default, ext4 reserves 5% of your block device for various special purposes. On most drives this is a sane default, but at this size that means a ridiculous ~200GB or so. So I needed to fix that.

{{< highlight bash >}}
wings@blinky:~# sudo tune2fs -m 0.5 /dev/sda
tune2fs 1.44.1 (24-Mar-2018)
Setting reserved blocks percentage to 0.5% (4883773 blocks)
{{< / highlight >}}

Now check it out again...

{{< highlight bash >}}
wings@blinky:~# df -h
Filesystem           Size  Used Avail Use% Mounted on
/dev/sda             3.6T   90M  3.6T   1% /mfsbrick
{{< / highlight >}}

Much better.

To make communication within the cluster a little easier, I decided to set up netatalk to enable me to address the nodes as nodename.local (eg: blinky.local). This was simple to set up - just install the netatalk package.

{{< highlight bash >}}
wings@blinky:~$ sudo apt install netatalk
{{< / highlight >}}

I also did a very basic benchmark to see what write speeds the raw disk could achieve - 
{{< highlight bash >}}
dd if=/dev/zero of=./largefile bs=1M count=1024
{{< / highlight >}}

I got a result of roughly 180MB/s. This is more than enough to saturate the Gigabit networking, and would later give me an idea of the overhead of running MooseFS. Read speeds were also around 180MB/s.

## Building and Installing MooseFS

Time to set up MooseFS.

Before I could build it, I needed to install its dependencies. While I was there I also grabbed fuse (for mounting the filesystem) and checkinstall (which would come in handy later).

{{< highlight bash >}}
wings@blinky:~$ sudo apt install build-essential libpcap-dev zlib1g-dev libfuse-dev pkg-config fuse checkinstall
{{< / highlight >}}

I grabbed [the latest MooseFS](https://github.com/moosefs/moosefs/releases) (v3.0.103 at the time) from the MooseFS GitHub and extracted it to my user's home.

{{< highlight bash >}}
wings@blinky:~$ wget https://github.com/moosefs/moosefs/archive/v3.0.103.tar.gz
wings@blinky:~$ tar xvf v3.0.103.tar.gz
{{< / highlight >}}

Then I built it -
{{< highlight bash >}}
wings@blinky:~$ cd moosefs-3.0.103/
wings@blinky:~/moosefs-3.0.103$ ./linux_build.sh
{{< / highlight >}}

Once it's built, I used checkinstall to install it while building a useful .deb package for later, using just "moosefs" for the description and changing no other options.

{{< highlight bash >}}
wings@blinky:~/moosefs-3.0.103$ sudo checkinstall
{{< / highlight >}}

## Creating the MooseFS users and groups
As the MooseFS components run as mfs:mfs by default, I created an appropriate user + group, and ensured it owned /var/lib/mfs + /mfsbrick

{{< highlight bash >}}
sudo adduser --no-create-home mfs
sudo chown mfs:mfs /var/lib/mfs
sudo chown mfs:mfs /mfsbrick
{{< / highlight >}}

I chose to set a null password to prevent logins as this user, and set its shell to /bin/false.

## Setting up the MooseFS Master
The first component to be configured was the Master. The Master is responsible for looking after the metadata of the cluster, and overseeing its operations.

I copied the sample configuration into place.

{{< highlight bash >}}
wings@blinky:~$ cd /etc/mfs/
sudo cp mfsmaster.cfg.sample mfsmaster.cfg
sudo cp mfsexports.cfg.sample mfsexports.cfg
{{< / highlight >}}

Then initialised the master by creating an empty metadata set.
{{< highlight bash >}}
wings@blinky:~$ cd /var/lib/mfs
sudo cp metadata.mfs.empty metadata.mfs
sudo chown mfs:mfs metadata.mfs
sudo rm metadata.mfs.empty
{{< / highlight >}}

Then started the master as well as the cgiserv service (which can be used to monitor the cluster).

{{< highlight bash >}}
sudo systemctl start moosefs-master
sudo systemctl start moosefs-cgiserv
{{< / highlight >}}

## Setting up the Chunkserver
The process for setting up a chunkserver was much the same as the master.

First, I copied the configuration into place:

{{< highlight bash >}}
wings@blinky:~$ cd /etc/mfs
sudo cp mfschunkserver.cfg.sample mfschunkserver.cfg
sudo cp mfshdd.cfg.sample mfshdd.cfg
{{< / highlight >}}

Then edited the configuration:
{{< highlight bash >}}
wings@blinky:~$ cd /etc/mfs
# Set MASTER_HOST = 10.1.1.201 and uncomment it
sudo nano mfschunkserver.cfg
# Add /mfsbrick to the end of the file
sudo nano mfshdd.cfg
{{< / highlight >}}

Once that was done, I started the new chunkserver:
{{< highlight bash >}}
sudo systemctl start moosefs-chunkserver
{{< / highlight >}}

## Setting up the MooseFS Client
Now that everything is running, it was time to configure the client and mount the filesystem for the first time.

As before, copied the configuration into place.
{{< highlight bash >}}
wings@blinky:~$ cd /etc/mfs
sudo cp mfsmount.cfg.sample mfsmount.cfg
{{< / highlight >}}

Then edited the configuration:
{{< highlight bash >}}
wings@blinky:~$ cd /etc/mfs
# Set mountpoint and mfsmaster
sudo nano mfsmount.cfg
{{< / highlight >}}

I edited the mfsmount.cfg configuration to look like this:
{{< highlight bash>}}
/mnt/mfs
mfsmaster=blinky.local
{{< / highlight >}}

The first line sets the default location for mounting MooseFS, while the second line tells the MooseFS client where to find the master.

With that done, I created the mountpoint and ran mfsmount.

{{< highlight bash>}}
sudo mkdir /mnt/mfs
sudo mfsmount
{{< / highlight >}}

Result?
{{< highlight bash>}}
mfsmaster accepted connection with parameters: read-write,restricted_ip,admin ; root mapped to root:root
{{< / highlight >}}

MooseFS is now mounted and functional with one chunkserver.

## Benchmarking, Testing and Other Fun

As before, I ran my basic dd write speed benchmark:

{{< highlight bash >}}
root@blinky:/mnt/mfs# dd if=/dev/zero of=./largefile bs=1M count=1024
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 8.08992 s, 133 MB/s
{{< / highlight >}}

At 133MB/s, the results were a bit slower than the raw disk but still fast enough to easily saturate the Gigabit networking of the node. As before, read speeds were similar.

## Examining The Cluster
You can examine the state of the cluster by looking at the CGI web interface. This usually lives on the active master on port 9425. For me, this was at http://10.1.1.201:9425/.

You may need to enter the address of your master in the text box that appears:
![MooseFS management interface](/img/2019-03-12-mfs1.png)

Once you do so, the web interface should pop up.
![MooseFS management interface](/img/2019-03-12-mfs2.png)

Once you start adding data, you'll notice that chunks are being created but are marked in the web interface as being "undergoal". This is normal and happens because the default goal for MooseFS directories and files is "2" - which means it will try to ensure they exist on at least 2 chunkservers.

## Going Into "Production"
Now that I had a functional MooseFS filesystem, it was time to add content and users.

I created a "media" directory in the MooseFS volume, and a "media" user on blinky with that as its home. 

I installed the Samba4 file server:

{{< highlight bash >}}
sudo apt install samba
sudo nano /etc/samba/smb.conf
{{< / highlight >}}

and added the following to the end of smb.conf:
{{< highlight ini >}}
[media]
    comment = Samba on Ubuntu
    path = /mnt/mfs/media
    read only = no
    browsable = yes
{{< / highlight >}}

then restarted Samba
{{< highlight bash >}}
sudo systemctl restart smbd
{{< / highlight >}}

Once that was in place, I set up automatic mounting for the MooseFS volume on boot by adding the following line to /etc/fstab:
{{< highlight bash >}}
mfsmaster: /mnt/mfs moosefs defaults,mfsdelayedinit 0 0
{{< / highlight >}}

I also set all the needed services to start on boot:
{{< highlight bash >}}
sudo systemctl enable moosefs-master
sudo systemctl enable moosefs-cgiserv
sudo systemctl enable moosefs-chunkserver

{{< / highlight >}}

And rebooted the node to see if everything came up cleanly. (It did).

Next time, we'll be adding a second node to the cluster.

*This post is part of a series. If you're interested in reading the rest of it, click [here](/tags/elasticnas/).*
