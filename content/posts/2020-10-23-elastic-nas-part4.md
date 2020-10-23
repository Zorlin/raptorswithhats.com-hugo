---
title: "The Elastic NAS: Part 4 - The second node"
date: 2020-10-23T14:33:30+08:00
Categories: [storage]
Tags: ['storage','nas','elasticnas','moosefs']
draft: true
---

## Arrival
I was very excited to receive a box from HardKernel... or, more accurately, a box full of boxes. I was so excited that this is the only picture I took.

![HC2s ready for unboxing](/img/2020-10-23-hc2-unboxing.jpg)

3 units had arrived, giving me a total of 4 units. As alluded to in Part 1, the decision to get 4 units was driven by finding a 3D-printable "fan shroud" that allowed you to mount a standard 120mm PC case fan. The shroud was specifically designed to strap together 4x ODROID HC2s.

However, at this stage, I only had enough spare money to purchase a 2nd hard drive. I would have to settle for two units until I could get my hands on hard drives for the final two units.

## Second verse, same as the first
From an operating system and software perspective, setting up the 2nd node ("pinky") was a very similar process to the 1st node.

I unpacked it, installed a 4TB WD RED drive just like in the first node. I flashed a copy of Armbian Focal (Ubuntu) to a microSD card, popped it in, and plugged the unit in.

Once again, it came up as "odroid" on the network. I assigned a static DHCP lease as before (this time for 10.1.1.202), ran some updates and rebooted.

## MooseFS

## Chunkserver

## Metalogger

## Samba
