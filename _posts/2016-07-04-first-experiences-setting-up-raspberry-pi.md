---
layout: post
title: 'First Experiences Setting Up A Raspberry Pi'
date: 2016-07-03 22:01:43 -0400
comments: true
categories:
---

Lauren got me a Raspberry Pi for the holidays last year, and I finally had a chance
to break it out (as a quick side note, if you're interested in a Pi then you'll need
a Pi, a micro USB power adapter, a microSD card, an ethernet cable, an HDMI-supported
monitor, and USB peripherals).

First thing I did was hook everything up, but I'm sure you could've figured that without
me. After that I tossed the SD card into my MacBook and [downloaded Raspbian](https://www.raspberrypi.org/downloads/raspbian/) onto it.
Raspbian is a Raspberry Pi-flavored fork of Debian.

Next, I plugged in the SD card into my Pi and plugged in the power. One thing to
note about the Pi (or at least my Pi which is a B+ model) is that there's no power
switch; it automatically powers on when you plug in the power adapter. When you first
boot up the Pi you should be greeted with a install wizard (similar to the Ubuntu one).
If not, you may need to make sure that you properly unpacked the downloaded OS image.
Otherwise, everything should be kosher and you can choose to install Raspbian. The install
took me around 20 minutes, and the initial boot took a bit as well before the desktop
came up.

Once the installation is complete I setup my wireless network. It's pretty straight
forward, but you can start [following the steps here](http://lifehacker.com/the-always-up-to-date-guide-to-setting-up-your-raspberr-1781419054) if you get lost.

Next, I wanted to setup SSH access so I could rid my living room of the mess of wires
I'd strewn everywhere. SSH'ing into the Pi is pretty easy; you can use `ifconfig` to
list your various network interfaces. If you have a wireless interface then I recommend
using that one. The `pi` user is setup by default with a password of `raspberry`. Once
you have SSH access rolling then you can remove all the unnecessary peripherals as
well as the ethernet connection. As a quick tip, you can reboot your Pi remotely using `sudo reboot`.

One of the things I noticed about my Pi is that it was pretty dang slow over SSH. The
first thing I tried was updating my Pi's firmware using [rpi-update](https://github.com/Hexxeh/rpi-update/).

```
sudo apt-get install rpi-update
sudo rpi-update
sudo reboot
```

Doing that seemed to upgrade a few things, but it didn't solve my problem. It seemed
like my SSH connection was occassionally slowing down, and I didn't know why. My first
thought was to Google for other Pi users using the same wireless adapter as me. I'm using
an Edimax EW-7811Un adapter, and thankfully I found [a great blog post](http://www.jeffgeerling.com/blogs/jeff-geerling/edimax-ew-7811un-tenda-w311mi-wifi-raspberry-pi) which used that adapter.

It turns out that the Edimax is configured by default to enter a sleep mode when there
is any inactivity. Adding the following to my `/etc/modprobe.d/8192cu.conf` seemed to do
the trick:

```
# Disable power management
options 8192cu rtw_power_mgnt=0 rtw_enusbss=0
```

Then just reboot using `sudo reboot` and voila, problem solved.
