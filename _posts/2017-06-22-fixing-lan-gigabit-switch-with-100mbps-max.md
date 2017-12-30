---
layout: post
title: "Why my Gigabit switch was limiting my desktop to 100Mbps and how I fixed it."
keywords:
description:
thumbnail:
facebook_type:
facebook_image:
---
I've been slowly reworking my home network over the past few months and ran into a new problem today - my desktop, wired to a gigabit switch via Cat6 cabling, would never reach speeds above 100Mbps. Bypassing the switch resulted in the expected speeds that were double that.

My network looks approximately like this: **[ROUTER]** cat6 **[SWITCH]** cat6 **[SWITCH]** cat6 **[NIC]**  
(Yes, I have multiple switches inline).

I found my solution in the Arch forums [here](https://bbs.archlinux.org/viewtopic.php?pid=801062#p801062).

It turns out the problem was caused by auto-negotiation being enabled at both ends of the link and not agreeing on who sets speed/duplex for link at gigabit speeds, resulting in the NIC staying at Fast Ethernet (10/100 Mbps) speeds.

The solution was simple.

- Install ethtool
    - I use Arch, ethtool is in the standard repos. Not sure for Deb/*buntu/etc users.
- Find your NIC identifier
    - `ip link`. In my case it was `eno1`.
- Disable Auto-negotiation and manually set the speed.
    - `sudo ethtool -s eno1 autoneg off speed 1000 duplex full`

And that's it.
