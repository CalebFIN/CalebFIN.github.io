---
title: Capturing Remote Wi-Fi Traffic on OpenWRT with Wireshark  
date: 2025-04-06 22:37:30 -0500  
tags:  
  - networking  
  - lab  
  - wireshark  
categories:  
  - Networking  
  - Homelab  
  - Wifi  
---

## Overview

I’ll explain how I transformed my OpenWrt access point into a remote Wi-Fi packet capture device. By leveraging SSH, tcpdump, and a custom channel hopping script, I can stream live Wi-Fi traffic directly into Wireshark on my Windows PC without needing physical access to the router.

I used a Netgear R7450 AC2600 to set this up.

I aimed to capture Wi-Fi packets remotely by putting my OpenWrt device into monitor mode and streaming the data over SSH. The main challenges I encountered were:

- **Device Mode Configuration:** Initially, setting up monitor mode correctly on the device was tricky. Using the wrong interface parameters led to errors.
- **Channel Hopping:** My device uses two radios—one for 2.4GHz and one for 5GHz. I needed a way to cycle through channels to capture a comprehensive range of traffic.

### 1. Configuring Monitor Mode

I used the following commands to push my radio cards into monitoring mode:

```shell
iw phy phy0 interface add mon0 type monitor
iw phy phy1 interface add mon1 type monitor
```

I ensured both wireless interfaces were set to monitor mode. Running the command:

```sh
OpenWrt:~# iw dev
```

yielded the following configuration:

```bash
phy#1
        Interface mon1
                ifindex 16
                wdev 0x100000004
                addr c8:9e:43:e2:d0:66
                type monitor
                channel 48 (5240 MHz), width: 20 MHz (no HT), center1: 5240 MHz
                txpower 20.00 dBm
phy#0
        Interface mon0
                ifindex 15
                wdev 0x4
                addr c8:9e:43:e2:d0:65
                type monitor
                channel 7 (2442 MHz), width: 20 MHz (no HT), center1: 2442 MHz
                txpower 20.00 dBm
```

This confirmed that the interfaces were ready for packet capture.

### 2. Implementing Channel Hopping

To cover the entire Wi-Fi spectrum, I created a dual-band channel hopping script. This script cycles through the supported channels for both the 2.4GHz (mon0) and 5GHz (mon1) interfaces.

Here’s the final version of the script:

```sh
#!/bin/sh
IFACE_24="mon0"
IFACE_5="mon1"
CHANNELS_24="1 2 3 4 5 6 7 8 9 10"
CHANNELS_5="36 40 44 48"
DWELLTIME_24=1
DWELLTIME_5=1
command -v iw >/dev/null 2>&1 || { echo "Error: iw not found. Please install it."; exit 1; }
command -v sleep >/dev/null 2>&1 || { echo "Error: sleep not found. Please install it."; exit 1; }
if [ "$(id -u)" -ne 0 ]; then
    echo "Error: This script must be run as root." >&2
    exit 1
fi
echo "Starting channel hopping..."
echo "  $IFACE_24 (2.4GHz) channels: $CHANNELS_24"
echo "  $IFACE_5  (5GHz) channels: $CHANNELS_5"
echo "Dwell times: ${DWELLTIME_24}s (2.4GHz), ${DWELLTIME_5}s (5GHz)"
channel_hop() {
    INTERFACE="$1"
    CHANNELS="$2"
    DWELL="$3"
    while true; do
        for CH in $CHANNELS; do
            echo "Setting $INTERFACE to channel $CH"
            iw dev "$INTERFACE" set channel "$CH" 2>/dev/null
            if [ $? -ne 0 ]; then
                echo "Warning: Failed to set $INTERFACE to channel $CH"
            fi
            sleep "$DWELL"
        done
    done
}
channel_hop "$IFACE_24" "$CHANNELS_24" "$DWELLTIME_24" &
PID_24=$!
channel_hop "$IFACE_5" "$CHANNELS_5" "$DWELLTIME_5" &
PID_5=$!
echo "Channel hopping started:"
echo "  $IFACE_24 (2.4GHz) - PID $PID_24"
echo "  $IFACE_5  (5GHz)  - PID $PID_5"
echo "Press Ctrl+C to stop."
wait
```

This script, adjusted for my device’s regulatory domain, excludes unsupported channels and those requiring DFS, which did take me a while to figure out.

![Channel Hopping in Action](assets/img/media/ChannelHop.png)

### 3. Running the Capture

With the channel hopper active, capturing the Wi-Fi traffic was straightforward. To capture 5 GHz, I used interface mon1.  
![Wireshark SSH](assets/img/media/tcpdumpsettings.png)

![Wireshark Capture](assets/img/media/Capture.png)
