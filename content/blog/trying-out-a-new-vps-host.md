---
title: Trying Out a New VPS Host
date: 2021-08-07
authors:
  - name: rthomasv3
    link: https://github.com/rthomasv3
    image: https://avatars.githubusercontent.com/u/39268276?v=4
tags:
  - VPS
excludeSearch: false
---

A while back I found out about a really cheap VPS hosting service called [Server Cheap](https://servercheap.net/) and I'd been wanting to give it a shot. Well, today turned out to be the day I finally had a little free time, so I decided to give it a shot.

When I first came across the service I was really surprised at the pricing - a dual-core VPS with 2 GB RAM, 60 GB SSD storage, and an unmetered gigabit connection for only $4.50/mo. Maybe I'm just not up-to-date with standard VPS pricing, but that seemed way too good to be true. On top of that, they had lifetime coupons for between 10% and 15% off.<!--more-->

Of course, the 15% off coupon didn't work for that particular selection, but the 10% sure did! So within a couple minutes a brand new Debian 10 VM was spun up and ready to play with.

Turns out they were telling the truth about the specs:

> cpu: 0 - Intel(R) Xeon(R) CPU E5-2650 v3 @ 2.30GHz
> cpu: 1 - Intel(R) Xeon(R) CPU E5-2650 v3 @ 2.30GHz
> *-memory - capacity: 2GiB
> *-virtio1 - size: 60GiB (64GB)

And a speed test shows the connection to be pretty great as well:

>Testing download speed.................................
> Download: 559.91 Mbit/s
> Testing upload speed...................................
> Upload: 461.47 Mbit/s

That really is a great price for so much speed and power (only $4.05/mo after the lifetime coupon). So I decided to move my current website over and see how everything performed.

## Setup
The process of getting the server ready was pretty straightforward - just `ssh` in and start punching in commands.

First up, I always do an `update` and `upgrade` to make sure the latest security patches are installed. From there I installed `sudo` and setup a new user in the sudo group. I really don't like to be logged in as root all the time as I think it's too easy to make a mistake.

After that I installed `nginx` and `ufw`. I then setup and activated `ufw` to for ssh, http, https, and Postgres (port 5432). The steps were really similar to the ones from my [previous post](https://rthomasv3.xyz/posts/nginx-and-uncomplicated-firewall).

Next up was getting Postgres installed. This is the same as any other Linux system really - just install the packages, setup the users, create databases, and optionally setup the config for remote connections.

Naturally, I don't have the commands for those steps perfectly memorized, so I followed this guide to make sure I got everything right:
https://linuxize.com/post/how-to-install-postgresql-on-debian-10/

And then disaster struck! My home system is Arch, and it turns out pgAdmin 4 is broken... again... Oh well, DBeaver works fine. So I opened a couple connections - one for the database running on my home server and one for the new VM. I then exported my current DB as just straight SQL and got everything copied to the new system.

Next was the setup for `nginx`, which just involves setting it up as a reverse proxy. When it gets a request on port 80, it forwards to my  application running on localhost at the local port. Again, this setup is really similar to my [previous post](https://rthomasv3.xyz/posts/nginx-and-uncomplicated-firewall).

Once that was completed, it was time to get the new server working with dotnet. I installed the dotnet runtime using the [instructions provided by microsoft](https://docs.microsoft.com/en-us/dotnet/core/install/linux-debian). I decided to use this opportunity to update my site to .net 5 and just installed that runtime.

After that I did a new build of my site, copied the files over, and updated the configuration accordingly. I then ran `dotnet <my-site>.dll` and visited the public IP for my new server and boom - the website worked great. I then copied the shell script over from my old server and set everything up to run my site as a service on system startup.

From there it was just a matter of updating the A records to point to the new server and letting `certbot` do its thing for https.

Overall, I'm quite happy with the result. The website is just as responsive and snappy as before, but now my home server is freed up to do some experiments with!

Next up, I'm going to try to get email running off the new server as well...