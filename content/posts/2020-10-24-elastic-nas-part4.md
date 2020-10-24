---
title: "The Elastic NAS: Part 4 - The second node"
date: 2020-10-24T11:03:13+08:00
Categories: [storage]
Tags: ['storage','nas','elasticnas','moosefs']
draft: false
---

## Arrival
I was very excited to receive a box from HardKernel... or, more accurately, a box full of boxes. I was so excited that this is the only picture I took.

![HC2s ready for unboxing](/img/2020-10-23-hc2-unboxing.jpg)

3 units had arrived, giving me a total of 4 units. As alluded to in Part 1, the decision to get 4 units was driven by finding a 3D-printable "fan shroud" that allowed you to mount a standard 120mm PC case fan. The shroud was specifically designed to strap together 4x ODROID HC2s.

However, at this stage, I only had enough spare money to purchase a 2nd hard drive. I would have to settle for two units until I could get my hands on hard drives for the final two units.

## Second verse, same as the first
From an operating system and software perspective, setting up the 2nd node ("pinky") was a very similar process to the 1st node.

I unpacked it and installed a 4TB WD RED drive just like in the first node. I flashed a copy of Armbian Focal (Ubuntu) to a microSD card, popped it in, and plugged the unit in.

Once again, it came up as "odroid" on the network. I assigned a static DHCP lease as before (this time for 10.1.1.202), ran some updates and rebooted.

## MooseFS
Fortunately, this time around, compiling MooseFS wasn't necessary. I could simply copy across the rough .deb package I had created earlier...

{{< highlight plaintext >}}
root@pinky:~# scp root@10.1.1.201:/home/wings/moosefs-3.0.114/moosefs_3.0.114-1_armhf.deb .
root@10.1.1.201's password:
moosefs_3.0.114-1_armhf.deb                                                                                                                        100% 1388KB  27.6MB/s   00:00
{{< / highlight >}}

Then install it.
{{< highlight plaintext >}}
root@pinky:~# dpkg -i moosefs_3.0.114-1_armhf.deb
Selecting previously unselected package moosefs.
(Reading database ... 47598 files and directories currently installed.)
Preparing to unpack moosefs_3.0.114-1_armhf.deb ...
Unpacking moosefs (3.0.114-1) ...
Setting up moosefs (3.0.114-1) ...
{{< / highlight >}}

The "moosefs" package I built earlier contained all of the MooseFS services, tools and utilities - which was not necessarily ideal. However, other than the extra disk space they took up, this wasn't a problem. Only services I explictly enabled would end up running and taking up any resources.

Just like last time, I created a MooseFS user and gave it ownership of the MooseFS data directory.

{{< highlight plaintext >}}
wings@pinky:~$ sudo useradd --no-create-home --shell /bin/false mfs
wings@pinky:~$ sudo chown -R mfs:mfs /var/lib/mfs/
{{< / highlight >}}

Unlike last time, however, the MooseFS services I needed to run were different - a chunkserver, a metalogger, and a client. A MooseFS CE installation only has one master, so I wouldn't need to run that here. Instead, I would be running a service called a metalogger. 

A metalogger serves as a rolling backup of the metadata set. Compared to the master, it has fairly low resource requirements. With a metalogger in play, losing the master becomes much less dangerous as it can be rebuilt from a metalogger's copy of the metadata.

## Chunkserver
Setting up the chunkserver followed the same process as on the first node. I'll reiterate it here, but skip over some of the details explained earlier.

Prepare the hard drive for use as a brick:
{{< highlight plaintext >}}
# Install the XFS utilities and drivers
root@pinky:~# apt install xfsprogs
# Lay down an XFS filesystem
root@pinky:~# mkfs.xfs /dev/sda
# Create a mountpoint for the newly created brick
root@pinky:~# mkdir /mfsbrick/
# Mount the newly created brick
root@pinky:~# mount /dev/sda /mfsbrick/
{{< / highlight >}}

Enable automatic mounting of the brick:
{{< highlight plaintext >}}
root@pinky:~# echo "/dev/sda /mfsbrick xfs nodev,noatime,nodiratime,largeio,inode64 0 2" >> /etc/fstab/
{{< / highlight >}}

Copy the example configuration file into place:
{{< highlight plaintext >}}
# Change to the MooseFS configuration directory
wings@pinky:~$ cd /etc/mfs/
# Copy the example configurations for the chunkserver
wings@pinky:/etc/mfs$ sudo cp mfschunkserver.cfg.sample mfschunkserver.cfg
wings@pinky:/etc/mfs$ sudo cp mfshdd.cfg.sample mfshdd.cfg
{{< / highlight >}}

Set up the brick:
{{< highlight plaintext >}}
# Create a folder for the chunkserver to use
wings@pinky:~$ sudo mkdir /mfsbrick/moosebox && sudo chown mfs:mfs /mfsbrick/moosebox
# Edit the list of bricks
wings@pinky:~$ sudo nano /etc/mfs/mfshdd.cfg
# Add the following line and save it
/mfsbrick/moosebox
{{< / highlight >}}

And finally, I started and enabled the chunkserver service:

{{< highlight plaintext >}}
wings@pinky:~$ sudo systemctl enable moosefs-chunkserver && sudo systemctl start moosefs-chunkserver
{{< / highlight >}}

And saw it pop up in the MooseFS web interface.

Now that I had two chunkservers up and running, all the existing data in the cluster was automatically replicated across both. This is because I had left the initial goal of goal=2 alone. After replication finished, all the "undergoal" chunks were now "stable" and the cluster looked very healthy.

## Metalogger
Next up, it was time to set up the first metalogger for the cluster. (If you were wondering, there's nothing stopping us from setting up a metalogger on the same node as the master. It wouldn't do anything useful, though, so there's no real point to doing so).

First off, I copied the example configuration files into place.

{{< highlight plaintext >}}
# Change to the MooseFS configuration directory
wings@pinky:~$ cd /etc/mfs/
# Copy the example configurations into place
wings@pinky:/etc/mfs$ sudo cp mfsmetalogger.cfg.sample mfsmetalogger.cfg
{{< / highlight >}}

Once that was done, I opened up the configuration file.

{{< highlight plaintext >}}
wings@pinky:/etc/mfs$ sudo nano mfsmetalogger.cfg
{{< / highlight >}}

As with the chunkserver, I had to set the following option - "MASTER_HOST = 10.1.1.201". That's actually it, though. Easy.

I enabled and started the service, and the metalogger appeared in the MooseFS web interface.

## Client
Last up, I had to setup the MooseFS client so I could mount MooseFS on the 2nd node.

As with the chunkserver, I'll reiterate the process here but skip over the details.

Configure the client:

{{< highlight plaintext >}}
# Change to the MooseFS configuration directory
wings@pinky:~$ cd /etc/mfs/
# Copy the example configurations for the client
wings@pinky:/etc/mfs$ sudo cp mfsmount.cfg.sample mfsmount.cfg
# Edit the client configuration file
wings@pinky:~$ sudo nano /etc/mfs/mfsmount.cfg
# Add the following lines and save it
/mnt/mfs
mfsmaster=10.1.1.201
{{< / highlight >}}

Test the client:

{{< highlight plaintext >}}
wings@pinky:~$ sudo mfsmount
mfsmaster accepted connection with parameters: read-write,restricted_ip,admin ; root mapped to root:root
{{< / highlight >}}

And enable automatic mounting:

{{< highlight plaintext >}}
# Edit the filesystem table
wings@pinky:~$ sudo nano /etc/fstab
# Add this line
mfsmaster: /mnt/mfs moosefs defaults,mfsdelayedinit 0 0
{{< / highlight >}}

## Media and Samba (Windows File Sharing)
It was time to add some basic services to make this NAS work like... well, a NAS. Adding Samba would allow me to use SMB (Windows File Sharing) to access and store media. I decided to add it to node #1 (blinky).

First up, I created a "media" user:

{{< highlight plaintext >}}
root@blinky:~# useradd -u 1100 -d /mnt/mfs/media media
{{< / highlight >}}

Then I installed and configured Samba:

{{< highlight plaintext >}}
# Install the Samba package
root@blinky:~# apt install samba
# Open the Samba configuration file
root@blinky:~# nano /etc/samba/smb.conf
{{< / highlight >}}

And finally added a few lines to the Samba configuration to enable access to the new "media" share.

{{< highlight plaintext >}}
[media]
   comment = Samba on Ubuntu
   path = /mnt/mfs/media
   read only = no
   browseable = yes
{{< / highlight >}}

With those changes in place, I started the Samba service and enabled it to run on boot.

{{< highlight plaintext >}}
wings@blinky:~$ sudo systemctl enable smbd && sudo systemctl start smbd
{{< / highlight >}}

I wasn't quite done, however. Out of the box, Samba has its own lists of accounts and passwords. So - I had to create a "media" Samba user.

{{< highlight plaintext >}}
root@blinky:~# smbpasswd -a media
New SMB password:
Retype new SMB password:
Added user media.
{{< / highlight >}}

With all that out of the way, finally, I could start using the NAS properly.

## Action Shot
![HC2s in action](/img/2020-10-23-hc2-in-action.jpg)

This was my setup for a little while. When treated as separate units like this, active cooling isn't super necessary. Handily, my VDSL router happened to have a 4-port gigabit switch built into it, so I was able to use it to provide simple networking.

However, this setup didn't look particularly neat and I was a little concerned about leaving things exposed like that.

## A LACK of Colour
I decided to get a "LACK" table from IKEA - initially to neaten things up, but possibly to mount some hardware in later. For those unfamiliar with the ["LackRack"](https://wiki.eth0.nl/index.php/LackRack), the LACK side table happens to have the same inner width as a standard server rack.

With the LACK in hand I did a "test fit", setting up all 4 units as though all were in service - complete with colour-coded Ethernet cables.

![Four HC2s on a LACK table](/img/2020-10-23-LACK-testfit.jpg)

I actually really liked the look, so I kept everything stacked in that configuration. You might notice that only two units were powered at this stage.

![Final configuration for a while](/img/2020-10-23-hc2-finalfit.jpg)

## Cool noises
I'm still not 100% sure why, but with all the units stacked up like that - heat eventually became a problem despite only having two of the units powered. 

While I could easily go back to the previous setup, that would be a temporary fix at best. I'd still need to solve the problem eventually, and it would only get worse with all 4 units in service.

My evil plan was to find a decent 120mm fan that I could power using 5v - allowing me to power it using the USB ports on the back of the HC2s. Combined with the fan shroud, this would keep the stack cool, and a low power fan would be quiet. My initial search didn't turn up anything super promising.

As a stopgap measure, I found a desk fan that was just the right size to point at the cluster. Unfortunately, this didn't last forever - consumer desk fans aren't really meant to run 24/7 - and it eventually ate itself.

## For $5, what did you expect?
I went to the shops to see if I could find anything to use as a *new* stopgap measure. There were a few desk fans - that could work! - except that they were far too big and wouldn't fit under the second LACK table (which I had added to neaten things up further).

Then, at my local electronics shop, I [struck gold](https://www.jbhifi.com.au/products/flea-market-usb-desk-fan-black-gold) - 

![USB fan](/img/2020-10-23-usbfan.jpg)

I found a USB fan on clearance for $5. Awesome!

At this point I need to point out that the Elastic NAS was physically located right next to the couch - it had to be quiet, and any noise it did make needed to be easy to ignore. This fan did not meet either criteria. While it performed surprisingly well in terms of efficiently moving air, it was whiny and made weird noises. Drat, back to the drawing board.

Meanwhile, my brother-in-law had kindly 3D printed the fan shroud for me. Since beginning the project, I had decided on a [slightly different design](https://www.thingiverse.com/thing:3113439) which was based on the original but was designed to force more air through the HC2s.

![The new shroud design](/img/2020-10-23-newshroud.jpg)

## Fan-tastic
I had the shroud in-hand, but no fan and no power solution. I continued researching for a while and found that there were no really great options for 120mm fans that could be powered via USB - very few fans were rated to run at 5v and most likely wouldn't even spin up at that voltage.

One option was to get a Molex-to-fan-header adapter and then a 12v power supply with a Molex output. Then I could power a high quality 12v fan. While looking into this, however, I found a 12v power supply with a barrel-style plug, as well as a fan adapter that worked with that. Simple!

I went ahead and bought a 12v 120mm Noctua fan and put it all together. Handily it came with a resistor to run the fan at a lower, quieter speed.

Here's the complete fan setup, taken when I was test-fitting the shroud in the kitchen. We've got a barrel-plug style power suppy, feeding a fan adapter, feeding a resistor, feeding the actual fan.

![The final cooling setup](/img/2020-10-23-cooling-testfit.jpg)

And here it is fully functional and attached to the Elastic NAS, which had grown to 3 nodes by this point.

![Elastic NAS fitted with the final cooling setup](/img/2020-10-23-hc2-finalcooling.jpg)

My goal was to have a very quiet cooling solution, and as it turns out I couldn't hear any significant noise from as close as 15cm away. I was ecstatic.

Join us next time as we complete the build and talk about goals and snapshots.
