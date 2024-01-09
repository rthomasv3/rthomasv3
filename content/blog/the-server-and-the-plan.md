
---
title: The Server and the Plan
date: 2021-03-21
authors:
  - name: rthomasv3
    link: https://github.com/rthomasv3
    image: https://avatars.githubusercontent.com/u/39268276?v=4
tags:
  - Server
excludeSearch: false
---

![The Server](/images/676126.jpg)

## Intro

Now that I have my very own shiny, decade-old ebay server, it's time to actually plug it in, make sure it works, and get everything setup. First lets start with the hardware.<!--more-->

## The Hardware

> CPUs: 2x Xeon X5660 @ 2.80 GHz (3.20 GHz turbo) for a total of 12 cores and 24 threads
RAM: 32 GB (16 x 2 GB) DDR3 10600R 1333MHz
SSDs: 1x 120 GB PNY CS900
HDDs: 7x 146 GB 10k 2.5" HDDs
RAID: HP Smart Array P410i
Power: 2x 750w Power Supplies
Bonus: DVD Drive

The server actually came with 8 of the 10k HDDs, but I replaced one with an SSD for the main OS install.

## Making Sure it Works

The first thing I did was open it all up and make everything looked good inside. To my surprise, it was actually pretty clean and all looked good. Even the thermal paste on the CPUs looked pretty good. I went ahead and did a full cleaning anyway and replaced the paste before turning anything on.

With the cleaning done I plugged everything in and hit the button...

Lights! Really loud fans! And a blank screen.

This is my first server so I wasn't really sure what to expect, but I've heard they can take a while to boot. So I waited about a minute or two - and to my great relief, a cursor appeared on the screen followed quickly by a splash screen and loading bar. Success!

I hit F9 and verified all the hardware was correct before moving on the the next step.

## The Plan

As with most servers, the goal of all that hardware isn't really to run one machine. The goal is to run many servers via virtualization. I'd used desktop virtualization in the past (VMWare, VirtualBox) but never virtualization for servers.

For that, I went to youtube where I quickly came across a channel called [Craft Computing](https://youtube.com/CraftComputing) with tutorials on a virtualization OS called [Proxmox](https://www.proxmox.com/en/). If you can get over the exaggerated youtuber voice he does, it's actually really well done and informative content.

With that piece of the puzzle, the plan was starting to come together.

Goals:

1. Get a server.
2. Spin up some Linux VMs.
3. Setup routers and firewalls so the server can be accessed from the outside world.
4. Serve multiple websites from my one public IP address.

Easy, right?
