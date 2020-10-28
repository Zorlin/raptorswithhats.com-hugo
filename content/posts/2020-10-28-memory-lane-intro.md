---
title: "Memory Lane: Introduction"
date: 2020-10-28T22:54:00+08:00
Categories: [gaming]
Tags: ['gaming','hacking','memory editing','cheats']
draft: false
---

Memory Lane is a new series of blog posts about having fun tinkering with various games in ways the developers didn’t intend, primarily through the use of memory editors but also through exploiting other interesting mechanics.

In simple terms, memory editing follows a loop of examining the state of something in memory, changing something, then examining that state again. The aim is usually to pinpoint a particular value stored in memory – so that you can change it.

As an example, imagine you’ve got $3410 of in-game currency and you’d like to have more. However, in this example there are dozens (or even hundreds or thousands) of regions of memory that have some variation of “3410” in them, so simply searching for that number directly doesn’t work.

Instead, you can search for all regions of memory that have “3410” in them, and store that list. Then, you can spend or gain some currency to change the number – and go back and search through the list you stored earlier. Suddenly the number of potential matches shrinks dramatically.

Repeat the process a few times and you’ll probably find what you’re after. Once you’ve pinpointed a specific memory address, you can then change its contents as needed.

If you’re familiar with tools like the GameShark and Game Genie, they work on a similar principle. They are essentially just memory editors. When you use GameShark or Game Genie “codes” in most cases you are actually just picking a region of memory and changing the values stored within it. Codes that require multiple lines are making multiple edits.

“Finding” GameShark codes involves examining the memory structure of a game, whether directly (by examining the code) or indirectly (making changes and then examining state, as described earlier). This is much easier to do with older game consoles, where the memory layout is consistent.

I will be using a memory editor for Android called GameGuardian, but I will stop short of recommending it – or any other particular tool, for that matter. It’s very easy to end up in trouble when working with apps that live outside of the Google Play Store, and particular caution should be taken when dealing with apps that need root access such as memory editors. Caveat emptor and all that jazz.

We’ll mostly be focusing on single-player games for a number of reasons. Most importantly, cheating in multiplayer games is stupid unless you’re playing with people that are also cheating (or at the very least are okay with *you* cheating). Secondly, in most cases it’s harder to cheat in multiplayer games and overall a lot less rewarding.

My weapon of choice for Memory Lane is the Xiaomi Mi Pad 4, running the Pixel Experience custom firmware. It’s modified with the usual stuff – such as Magisk – to allow me root access and other niceties such as having relatively up-to-date security patches compared with the original firmware.

It does currently pass SafetyNet, which is Google’s framework for detecting modified devices. I don’t expect that to last, but it’s handy for now - it’s very convenient to have access to Netflix and Stan and some games require SafetyNet to work (or more precisely, will refuse to start on a device that fails the SafetyNet checks).

I’ve got a few ideas for material for the series – plenty of games that are vulnerable to being poked by memory editors and other exploits. One interesting aspect of game hacking I’d like to explore in this series comes in the form of cheat protection – game developers battling potential cheaters by making it harder to successfully perform editing and other hacks. This can take many forms.

One of my favourite things to do is to find ways around that protection, or things the developer simply didn’t think of. Who knows – maybe if this series gets popular enough, the developers will learn from these posts and patch things up further.

The first proper post of the series is coming soon and will explore the game “Hill Climb Racing”.
