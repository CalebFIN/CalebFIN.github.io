---
title: "Reviving a Cisco Catalyst 2960G"
date: 2025-04-04 23:00:00 -0500
categories:
  - Cisco
  - Hardware
  - Homelab
tags:
  - networking
  - repair
  - lab
---

## Reviving a Cisco Catalyst 2960G

One of the coolest wins for my homelab recently was scoring a Cisco Catalyst 2960G switch. The reason it was free? It wouldn't boot.

![Cisco Catalyst 2960G Front Panel](assets/img/media/IMG_3157.jpg)

When I first powered it on, all the indicator lights looked good—except the top "SYST" light, which stayed solid amber. Normally, this means there's a hardware fault, so I initially suspected a bad power supply. But after some digging, I discovered that many others had seen this exact issue, and it was often caused by a failed RAM chip.

---

### Investigating the Faulty RAM

After opening up the switch and taking a closer look at the board, I found the suspected RAM chip.

![Close-up of Faulty RAM Chip](assets/img/media/IMG_3174.PNG)

Based on forum posts and YouTube videos, it seemed like thermal instability in the chip was causing the boot issue.

So, I decided to test a workaround: **heating the RAM chip just before powering on the switch**. I grabbed a small handheld heat gun from Walmart, gave the chip a solid blast of hot air, and sure enough—the switch booted right up!

---

### Still works

Now, I’ve got a working 2960G in my lab setup—but there’s a catch: if the device powers down, I have to reheat the chip before booting again. Definitely not ideal, but it works for now.

![Inside of the Cisco Switch with RAM and Heat Sink](assets/img/media/IMG_3179.JPG)

Eventually, I’d like to attempt **desoldering the bad chip** and replacing it with a new one. Until then, it’s staying warm and running!

---

For anyone doing homelab on a budget, this kind of **hardware revival trick** is worth keeping in your toolbox. 
