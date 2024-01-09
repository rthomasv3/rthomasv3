---
title: Nginx and UFW
date: 2021-04-17
authors:
  - name: rthomasv3
    link: https://github.com/rthomasv3
    image: https://avatars.githubusercontent.com/u/39268276?v=4
tags:
  - Nginx
  - Uncomplicated Firewall
  - ufw
excludeSearch: false
type: docs
---

## Intro

In the [previous post](/tutorials/proxmox/proxmox-setup-and-vms/) I went over my setup of Proxmox VE, got a few virtual machines created, and installed Ubuntu Server 20.04 onto them. The two VMs relevant to this post are the gateway and blog. The gateway VM will act as a reverse proxy and route incoming requests to the correct server. The blog VM will simply serve this website.

Now all there is to do is `ssh` into the servers and start setting things up.<!--more-->

## Uncomplicated Firewall

Uncomplicated Firewall or `ufw` is a command-line program for managing a Netfilter firewall. Netfilter is a network management framework provided by the Linux kernel. All of this is included in Ubuntu Server by default and shouldn't need to be installed. If, for whatever reason, it's not on your server installation is the same as any other program:

```
sudo apt install ufw
```

What `ufw` allows us to do is quickly and easily setup a host-based firewall for whitelisting incoming and outgoing traffic on our server. There are really only a few ports you want open on a typical web server.

* SSH (port 22) for remote login to the server.
* HTTP (port 80) for serving web pages.
* HTTPS (port 443) for serving web pages securely.

First up is to make sure `ufw` isn't active. You don't want to accidentally lock yourself out of the server by blocking `ssh`.

```
sudo ufw status
```

The above command should return "Status: inactive". If it shows "active" you can run `sudo ufw disable` to temporarily disable it.

First up is to deny all incoming traffic by default.
```
sudo ufw default deny incoming
```

Then we want to whitelist certain traffic, starting with `ssh`.
```
sudo ufw allow ssh
```
> Note: This will allow `ssh` from anywhere. If you want, you can make `ssh` only available from within your internal network. Use `man ufw` and check the "RULE SYNTAX" section  for details.

Next up is to allow HTTP traffic.
```
sudo ufw allow http
```

Then HTTPS traffic.
```
sudo ufw allow https
```

And finally enable the firewall.
```
sudo ufw enable
```

If all went well you should be able to run `sudo ufw status` again and see "Status: active" followed by the list of rules we just setup.

## Nginx

Next up is to install Nginx onto the servers so they know what content to serve when the get a request. Nginx - pronounced "engine X" - is a really powerful free and open source web server. It can be used as a gateway, reverse proxy, load balancer and more. For my purposes I will basically be using it as a reverse proxy (because my ISP - like most - only gives me one free IP address).

Installation is again the same as any other app on Ubuntu:
```
sudo apt install nginx
```

After installation, `nginx` creates a default profile that returns a static html welcome page similar to below.

![Nginx - Welcome Page](/images/971878.jpg)

The overal configuration files for `nginx` can be found in the `/etc/nginx` folder, with website configurations in the `/etc/nginx/sites-available` and `/etc/nginx/sites-enabled` folders. The idea is that you can keep multiple configurations for a website in _sites-available_ and enable them via a symlink to _sites-enabled_. This allows you to create _slots_ like it Azure, serve _maintenance_ pages, and more by just updating the symlink.

To serve my blog, I'm going to need a reverse proxy that forward requests to the blog VM when traffic is going to that domain. To accomplish this I created a new file in _sites-available_ with `sudo nano /etc/nginx/sites-available/rthomasv3.xyz`. You can use whatever editor you want here, but I'm not great with `vim` so I usually just use `nano`.

This is the configuration I ended up with:

```nginx
server {
  server_name rthomasv3.xyz www.rthomasv3.xyz;

  location / {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass http://10.0.0.3:80;
  }
}
```

What this will do is listen for requests for _rthomasv3.xyz_ and forward them on to the local IP address of the blog VM. But first it needs to be linked to _sites-enabled_:
```
ln -s /etc/nginx/sites-available/rthomasv3.xyz /etc/nginx/sites-enabled/rthomasv3.xyz
```

The configuration can then be tested with:
```
sudo nginx -t
```
Output:
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Lastly, `nginx` needs to be restarted with `sudo service nginx restart`. For me, the expected result is actually "502 Bad Gateway" or similar wehn I visit the server's IP because I haven't setup the blog VM that it's forwarding to.

The setup for the blog is very similar to the previous steps - install `nginx`, create and test the configuration, restart the service. The difference is that I want to serve the website running locally on the server instead of forwarding it onto another VM.

Because this website was created as a single page application using Vue and .NET Core, my configuration ended up looking like this:
```nginx
server {
  listen 80;
  listen [::]:80;

  server_name rthomasv3.xyz;

  location / {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass http://127.0.0.1:5000;
  }
}
```

This is actually yet another reverse proxy, but instead is points to the website running on localhost. Now when I visit the blog server's IP I get my website.

Next I opened up HTTP and HTTPS on my router and forwarded all incoming requests to my gateway VM. I also made sure all the VMs I created were given a static internal IP from my router.

The last step was to update the DNS records for _rthomasv3.xyz_ to point to my IP address.

Now, when you visit _rthomasv3.xyz_, it forwards to my IP address. My router lets the request go through to my gateway VM, which then checks the website you wanted and forwards you to the corresponding VM on my local network. That VM then forwards the request again to the website running on its localhost.

Simple, right?

## Conclusion

I was actually surprised how easy everything was to setup and how quickly I got it all up and running. I now have my own server running all the virtual machines I need and I can even serve multiple websites from my one IP address by using `nginx` as a reverse proxy.

For me it's definitely worth it to run my own servers. In my area the electricity usage comes out cheaper than paying for similarly powerful hosting services. I also had a lot of fun setting it all up!

## Further Reading

I left out a lot of the details for getting a .NET Core site up and running on Linux and securing everything, so I thought I would just include the sources I used here:

* [Building an ASP.NET Core web app for a Linux VPS: Part 1](https://www.codeproject.com/Articles/1111663/Building-an-ASP-NET-Core-web-app-for-a-Linux-VPS-P)
* [How To Set Up Nginx Server Blocks (Virtual Hosts) on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-server-blocks-virtual-hosts-on-ubuntu-16-04)
* [How To Secure Nginx with Let's Encrypt on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-20-04)


