---
title: "The Elastic NAS: Introduction"
date: 2019-09-12T13:50:37+08:00
Categories: [storage]
Tags: ['storage','nas','elasticnas','moosefs']
draft: false
---

My first NAS was a QNAP TS-412, a 4-bay NAS which served for years as a place to store photos, videos and backups with relative ease. It cost around $650 when brand new. When I moved out after getting my first apartment, the QNAP stayed behind to continue dutifully serving those roles for my family.

After moving, I knew I wanted a NAS of my own. I had loved the QNAP, but it had a few issues:

* It took forever to boot - sometimes as long as 15 minutes.
* Changing the RAID layout was possible but difficult and nerve-racking.
* It had a fairly weak processor - with only 256MB of RAM and a single-core Marvell CPU.
* And most importantly, it was expensive.

I knew I wanted a 4-bay NAS but I couldn't find a decent 4-bay unit for less than about $650 AUD – let alone one that would solve the issues I had with the QNAP.

## MooseFS
I've personally been using MooseFS on and off since September 2013, having started with it at a previous job at a college called DigiPen. By the time I left DigiPen, we were managing 170TB of raw storage and a huge amount of files and metadata – over 100 million files. For me, MooseFS has been simple, fast and stable.

I always liked the idea of having a MooseFS-based cluster at home, so that I could keep my skills sharp and have a place to experiment with things. So I began the search for something to build a MooseFS cluster with.

I looked into the Raspberry Pi 2B, but the lack of proper Gigabit Ethernet was a deal-breaker for me. I looked at the Banana Pi, with its SATA port, but it didn’t “feel right”. Finally, I found an x86 board that looked really interesting, but was busy and didn’t end up purchasing it.
At that point the idea of a home cluster went to the back of my mind and I mostly forgot about it.

## Enter HC2
Then, one day, inspiration struck. Someone posted this to Reddit...

![ODROID HC2 200TB Gluster](/img/2020-09-12-hc2-200tb-gluster.jpg)
(Credit: u/BaxterPad on Reddit)

It’s a [200TB GlusterFS-based storage cluster](https://www.reddit.com/r/DataHoarder/comments/8ocjxz/200tb_glusterfs_odroid_hc2_build/), built with 20 ODROID HC2 single board computers. Reading about their experiences inspired me to revive my dream of a home storage cluster.

I looked up the single board computer they used in that build – the ODROID HC2 – and was intrigued. The HC2 was a powerhouse, with 8 cores and 2GiB of RAM, but most importantly, it had a real SATA3 port… which handled both data *and* power. Here it is:

![ODROID HC2](/img/2020-09-12-hc2.jpg)

It was simple, stackable and self-contained: essentially a massive heatsink and a tiny board, with mounting points for a standard 3.5" hard drive. It had Gigabit Ethernet and a microSD card slot for booting. Interestingly, it’s completely headless - there is no video output at all.

I started researching the ODROID HC2 and how people were using it and stumbled upon this on Thingiverse. It's a simple 3D-printable case that allows you to add a fan to a set of four HC2s while keeping them neatly together.
![ODROID HC2 Fan Shroud](/img/2020-09-12-hc2-shroud.jpg)

Then the idea struck me – instead of buying a NAS, and building a MooseFS cluster – what if I built a MooseFS-based NAS? I could purchase 4x ODROID HC2 units and strap them together with MooseFS – and it would even be cheaper than buying a traditional 4-bay NAS.

## The plan
The first step was to evaluate whether this would be a good idea. I figured I would purchase a single board and test it out. The HC2 is considered a "developer" board and as a result has a very short warranty period (1 month), but I found an Australian reseller called JiffyShop that offered a 3-month warranty, though at a higher cost than purchasing directly from HardKernel (the manufacturers of the ODroid HC2).

I figured the difference was worth it for the peace of mind in case I ran into any issues. All up, the first board cost me about $160 including power supply and some accessories, such as a "case".

Once I had my board in hand, I would set up a basic single-node installation on it, and determine if the speeds, latency and reliability were good enough to justify continuing. If they were, I would buy 3 more nodes and expand the cluster to span all 4.

Shopping list (Prices in AUD):

* ODROID HC2 + Power supply + Case ($160)
* 16GB microSD card ($25)
* 4TB WD RED drive ($180)
* Network cable ($3)

Tune in next time to see what happened after the first HC2 arrived, and what the setup process looked like.
