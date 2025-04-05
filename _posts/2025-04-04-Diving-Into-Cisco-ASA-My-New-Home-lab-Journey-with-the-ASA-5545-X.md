---
title: " Diving Into Cisco ASA: My New Home lab Journey with the ASA 5545-X"
date: 2025-04-04 22:00:00 -0500
categories:
  - Cisco
  - Firewalls
  - Networking
  - devnet
tags:
  - data
  - networking
---
So I recently picked up a **Cisco ASA 5545-X** off eBay. It’s a bit larger (and louder) than I expected, but I’m excited to finally start learning the Cisco platform more seriously—especially for **home labbing** and expanding my skills with advanced configurations.
![ASA Front](assets/img/media/IMG_3325.jpg)
The unit came with **dual power supplies** and bays for two SSDs. When I powered it up, I immediately noticed how **loud** it was. It’s definitely not optimized for quiet indoor use, but for now, it’ll do while I’m experimenting and learning.

Interestingly, the ASA came with **perpetual licenses** already loaded on it. While that’s nice for lab work, I know this model is **End of Life (EOL)**, so I’m hesitant to rely on it for any serious production use—especially if I eventually want to **self-host a DMZ** setup. Still, it’s more than good enough for learning and building fun, complex network environments at home.

Currently, I’ve got the ASA **plugged into a Cisco 2960 switch**, which is connected to my main internet line. I was trying to get ADSM working but ran into issues—likely due to licensing restrictions. I’ll troubleshoot that more later.

I’m already considering **modding it to make it quieter**. I cracked it open and noticed it runs with **DDR3 RAM** and a **Xeon processor**, which surprised me in a good way. Right now, I’m looking at quieter **aftermarket fans** (found a few decent ones on Newegg) or possibly building a **custom enclosure** to reduce the noise and make it more indoor-friendly.
![ASA Inside](assets/img/media/IMG_3330.jpg)
For now, I’m only using **one power supply** since I don’t need redundancy in a homelab setting, and I’d rather not deal with extra heat and power draw.

Eventually, I plan to **run my VoIP setup through it**. I’ve got a **Yealink IP phone** and a **Raspberry Pi** acting as my **Session Border Controller (SBC)**. That SBC connects to my **3CX cloud PBX**, where I have my SIP trunk registered. 
