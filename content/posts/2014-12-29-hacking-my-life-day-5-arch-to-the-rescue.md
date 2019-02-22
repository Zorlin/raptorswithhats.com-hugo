---
title: "Hacking My Life: Day 5 - Arch to the Rescue"

date: 2014-12-29T17:22:20-08:00
Categories: [hml]
Tags: [everyman, arch, linux, "arch linux"]
---

*This post is part of a series. If you're interested in reading the rest of it, click [here](/categories/hml/).*

So I wasn't successful in switching to the Everyman E3 sleep schedule. I'm still curious about it and will probably try again, but I had a full sleep and loved it. I will need to figure out how it'll affect my work.

Anyways, today's post is about a dilemma I found myself in earlier today. I <strike>had</strike> wanted to reinstall my Arch Linux install to get a fresh start, so I booted into Arch via DriveDroid and proceeded to (try) to reinstall it. Immediately after wiping it I realized I didn't have an internet connection...
<!-- more -->

So this was quite a pickle. In an installed copy, I can simply compile and install the wireless drivers and be up and running, but you can't exactly do that with a live install because it won't survive a reboot. So... what to do?

**SIDENOTE AND BORING DISCLAIMER**: I'm going to be doing some dangerous stuff in this post, so please be careful and proceed at your own risk.

So, what do we do? Simple. Start -> "Disk Management" -> find your disk and see what it's labelled as. Mine is "Disk 0".

Change to the directory you've got VirtualBox installed in:

cd "C:\Program Files\Oracle\VirtualBox"

Now that we know what VirtualBox will see the disk as, we can use it at the end of our command.

VBoxManage internalcommands createrawvmdk -filename C:\Users\wings\archredirect.vmdk -rawdisk \\.\PhysicalDrive0

Hopefully you'll get a message like this: "RAW host disk access VMDK file C:\Users\wings\archredirect.vmdk created successfully."

Open VirtualBox **as an Administrator** and create a new VM. Call it "Arch" - the name doesn't matter since we'll delete it afterwards, but that tells VirtualBox it's an Arch-64 box.

Follow the wizard until it asks for a disk. Select "Use an existing virtual hard drive file" and point it at your VMDK. Follow the rest of the wizard and assign your Arch Linux ISO to it.

Start the VM and follow the install process. Once you're finished, cleanly shut it down and reboot - you should boot straight into your new Linux machine! Magic!
