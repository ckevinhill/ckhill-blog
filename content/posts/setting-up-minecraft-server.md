---
title: "Setting Up A Minecraft Server"
date: 2020-07-08T17:25:34+08:00
tags: ["tutorial"]
draft: false
---

The kids wanted to play Minecraft with their cousins as part of our *social distanced* activities.  As we are not all on the same LAN this requires hosting a Minecraft server with a publically accessible address so that everyone can be together in the same World.

### IaaS vs. host my own

A quick google search showed several cloud-based Minecraft hosting services at extremely low costs (e.g. $2/month).  The simplicity and advantages of a hosted server include:
* Management dashboards to setup users, access restrictions, upgrades
* Ability to easily move server across Regions (e.g. easy to move server location from US to Singapore if desired for performance)
* Automated upgrades to match client versions
* Automated back-ups

For the cost, the benefits of using a managed Minecraft hosting service seem attractive.  Reviews indicated fairly common performance but I ended up choosing [Shockbyte](https://shockbyte.com/minecraft-hosting) as they had some of the lowest costs, had strong documentation and were focused on Minecraft server hosting.

### Setup

Initial setup was extremely easy via the Shockbyte portal.  A few questions I needed to answer included:
* What region to host the server in
* What package to choose (basically how much Ram the server would have allocated for concurrent user support)
* What server version I needed

All of these can be changed at a later date as well.  I was looking at about 5 concurrent players so the "Dirt" plan seemed sufficient at $2.50/month.  The server was established in US and I deployed the latest official version of the "Bedrock" server as all of the kids were primarily playing on either tablets or mobile devices.  This version should also support the X-box when we get it back out of storage.

After making initial configuration selections it did take around 10 hours for the server to be allocated which was definitely not the *Instant Setup* proclaimed on the site but worked okay for me.

After server was allocated Shockbyte provides link with access to the the Minecraft Control panel and IP information for the new Minecraft server.  Shockbyte also provides alot of video tutorials (e.g. [Getting Started](https://www.youtube.com/watch?v=cGJ0Dm_50es&list=PLGecCt6miCYJ7LrMY28kBVWvt9tOuo-nj)) if you need additional help.

### Setting up custom DNS

I wanted to make it easier for the kids to configure their tablets for connection to the Server (Minecraft >> Play >> Servers >> Add Server) so I also setup a sub-domain of minecraft.ckhill.com pointing to Minecraft server IP.  You can do this by adding a simple ["A" record](https://www.cloudflare.com/learning/dns/dns-records/dns-a-record/#:~:text=What%20is%20a%20DNS%20A,5.78.) to your Name Server configuration.  DNS for my domain is owned by GoDaddy and you can find A record instructions [here](https://sg.godaddy.com/help/add-an-a-record-19238).

### Results

During quarantine and our move to Singapore this has been the best $2/month I've ever spent.  The kids have enjoyed connecting with each other and their cousins remotely to build a persistent, shared world.  Additionally I have started playing some with them from my mobile phone (they are using Fire tablets).  This is a good way to kill a few hours per day in quarantine or we've even used it while waiting in hotel/government lobbies waiting for paperwork to be completed.  Pretty much anywhere there is a Wifi connection (which is everywhere now-a-days) they can connect and pick up on their builds.

The only real downside is that they keep stealing all of the diamonds and obsidian from my chests ;)h.  