---
author:
  name: "Chris B"
date: 2024-09-30
type:
- post
- posts
title: Seven Things NOT to do While Your Partner is on the Same Network
description: Or how to prevent marital disputes while tinkering at home
weight: 10
series:
- Personal
---

## Introduction
I like to tinker with things. It's been a form of stress relief and a way to satisfy curiosoty about new tools, frameworks, and all things technology for my entire life. Naturally, this led me to setting up a homelab where I can not only play with new ideas I have, but setup tools and services for myself and others in my house.

I also am married to a wonderful and tolerant partner who gives me the time, space and support to do the things I like to do. That being said, sometimes I can make some pretty bad decisions. I've talked about some of these before in random developer talks and trainings, as well as in [Christopher's List of Bad Ideas in IT](https://github.com/devbyaccident/christophers-list), but that's all been in a professional or corporate setting.

This post is going to sum up some of the good ideas gone bad (and just plain bad ideas) I've had at home, and hope to have a bit of a retrospective so I can try to keep someone else from making some of the same mistakes.

Also, a note on the language here, any pronouns used are specific to my living situation. These lessons aren't gendered, and apply to any technical person living with a non-technical person. Now on to the list...

### 1. Don’t Over-Utilize Ad Blockers
I **LOVE** ad blockers. PiHole, Adguard, NextDNS, I have run or currently run them all at home. I know a lot of sites rely on ads nowadays to support for themselves, but I'm not comfortable with my attention span being bombarded with the insatiable capitalistic hunger of the marketing industry just so I can casually browse the internet. Whenever I setup a new ad blocker, I tend to add as many block lists as quickly as I can to maximize the amount of traffic I'm preventing from leaving my home network.

Sometimes that can lead to disaster, like in the case where I implemented the [no-google](https://github.com/nickspaargaren/no-google) blocklist while configuring Adguard Home before leaving to go on-site for a full day of meetings. On a week where my wife had a mid-term paper due for grad school. A paper that was on Google Docs.

I was tired and clicking through the check boxes that Adguard is known and loved for making so easy, and accidentally clicked one too many. Then I left and wasn't able to get back to support it before the damage was done. Which leads me to my next don't for the home network...

### 2. Treat Your Home Network the Same Way You Treat Production
I've talked before about my struggle with insomnia. Some days it's good, and I'll be able to go to sleep and get up at a reasonable hour. Some days it's not, and I either am not able to sleep until around the time the rest of the world is waking up, or I wake up just after the rest of the world has gone to sleep and can't get back there myself.

Those are not the days to make changes to shared parts of the network.

No matter how well the nights go or how awake I feel, there's always some focus lost when sleep doesn't happen easily. On those nights, even if I test and check everything over and over, there's still a potential for error when actually deploying changes, which is what happened when making the Adguard changes described above. 

If I had tried to make a change like that to a system my team at work relies on I would have been out of work the very next day, yet for some reason, I thought it was fine to do on a system my family relies on at home. Then I left for work but while the system was down. If that had been a orod system at work, I woul
d have been rightfully fired for leaving it like that.

The takeaway for this one is best summed up by the wife herself:
> "You're not the only one using the internet at home. You need to treat it like you production at work."

### 3. Backups and Redundancies are Just as Important at Home as They Are at Work
If you're running any kind of homelab, whether it's running on a Raspberry Pi or you have a server rack in your basement, you're probably running a few different services at once. Maybe it's on Docker containers, or maybe you've got a bunch of virtual machines running on Proxmox or VMWare. 

Whatever the case, you might have something on there that you have unintentionally or otherwise made into a critical piece of infrastructure without realizing it. In my case, it was a DHCP server running on a virtual machine in a proxmox cluster through Pihole, (Ad blockers being a theme of this post apparently) and a [Teklager APU](https://teklager.se/en/products/routers/) running OpenWRT.

There was a power outage that forced me to shut down my cluster, and I didn't realize that a fair amount of my DHCP leases were soon going to expire. Once they did, nothing on my network could reach anything else. Even once I turned the cluster back on, getting everything back online was a pain, and I ended up needing to reset the router and use OpenWRT DHCP.

The lesson here is to keep anything that is critical the network as close as possible, ideally on the same hardware. If not, have some kind of redundency in place for when it goes wrong. My current DHCP is on an OpnSense router, but I also have a couple of devices with static IPs just in case I need to get on there when the rest of the network goes down.

In about a year since then, it's failed one other time and then I was able to restore from a backup and get everything working in about two hours.

### 4. Don’t Implement Any Automation Without Telling Your Partner
I love tinkering with Home Assistant, and one of the things I use it for is parental controls. I have automations and dashboards so I can track how long the TV is on while the kids are home, or control it if the remote gets lost, or even turn the volume down if it gets too loud. 

I've even got some automations that will turn the TV back off if it's turned on at a time where my kids shouldn't be watching it, and announces loudly `"It's not time for TV right now"` from the speaker in my living room.

What I didn't have is a way for the TV to tell when my wife was watching it instead of our kids.

To be fair, my wife is a fan of this automation and has even made use of it many times herself to manage the kids screen time. What she wasn't a fan of was when she wanted to cool down at the end of a long day when the kids were out, only to be greated by `"It's not time for TV right now"`

Since then, we've worked on better rules around when to change the TV time (Like automatically allowed after the kids bedtime) but having that conversation before setting up the rules would have saved some headache for both of us.

### 5. Don’t Implement VLAN Segmentation Without Making Sure All Devices Are on the Right VLAN
VLANs are something everyone should have, especially if you have a lot IoT devices from questionable vendors floating around your network. To anyone looking to tinker with your network, it's worth going through [RouterSecurity.org](https://routersecurity.org/) and following their guides.

In my case, I have my primary devices on one VLAN, all my IoT stuff and Home Assistant on another, and my kids devices on a third. None of the devices on any VLAN can communicate directly with devices on any other VLAN, and they all have seperate DNS resolvers. 

It wasn't always setup like this, it took time to configure it all, and the last bit to get setup were the firewall rules to keep everything seperated. It's when those last rules got added that I was informed about an issue: Our media player didn't have access to our media library.

Taking a closer look, I had put the NAS and media server on different VLANs, and by the time I finished setting up my firewall rules, I hadn't realized I needed to put an exception in place for the media player to get the media from the NAS.

Now I've got the firewall configured to allow the traffic, but from my wife's perspective, she had a long day and wanted to sit down and watch TV, but it didn't work. The firewall rules themselves weren't important, but the interrupted functioning of the devices in my house was.

### 6. Buy Good Equipment First or Buy Twice the Equipment Later
When I first started tinkering with my home network, I had grand visions of staying open source at every level. I firmly believe that investments in open source always pay dividends. I started flashing OpenWRT on all my routers and access points, even going so far as to buy open source hardware to install it on. I figured if there were any issues with configuration I could just go and fix it, no harm done.

What I didn't appreciate was that for my wife, technology that doesn't **just work** isn't a configuration issue to be fixed at a later time, it's downtime that's blocking her from using anything.

Quickly it became clear that the radios on the wifi access point wasn't strong enough to reach our attic, where my wife's office is. She could still connect from there, but things like video calls were out of the question. That put her in a situation where if she wanted to work with her graduate school thesis group, she had to do it from our living room, the noisiest room in our house. And if there's anyone in the house streaming video at the same time, it wouldn't work at all.

At first, I tried to add some other access points as repeaters and tweak the QoS settings, but none of that could solve the problem. Meanwhile my wife is getting more and more frustrated, even being unable to work on some projects. It finally came to a breaking point when she was doing a thesis group meeting and one of the kids streamed a game from Steam, which crashed her meeting.

I needed to fix the network quickly, and have something where not only was the signal stronger, but if there were any signal gaps they could be diagnosed and fixed without a lot of fussing with it. 

On the advice of a buddy, I sunk some money into a Ubiquity wireless access point and managed switch. The difference in range and performance was drastic and instantanious, and made me realize that if I had gone with that reccomendation in the first place, I wouldn't have needed to buy the extra hardware first.

Now my network reaches all corners of my house, is easily managed, and I still have my open source OpnSense router defining VLANs and traffic rules using Unifi as an access point. Wife can take her devices and do video calls anywhere in the house, even if the kids are streaming TV, games, or whatever.

Best of all, it **just works**.

### 7. Don’t Setup Something You Think Is Useful Without Making Sure She’ll Use It
I get very excited about setting up new services and tools to use, especially when it's not just for me. Despite the troubles with home automations, ad blockers, and media streaming, my wife has been incredibly happy with many of these services, and has used them extensivley. 

There are also some that are clearly just for me, such as invoicing platforms, development instances, and dashboards. These are just as fun to setup, but also incredibly useful to me when I'm working or doing more tinkering.

Then there are some that I think would be useful to both of us, but wouldn't get used by my wife at all.

For example, one of the first things I setup on our network was an SMB share. It seemed like a great way to share files between our different computers, keep files safe in case of a laptop crashing or hard drive corruption, and have a place to put things if a hard drive filled up.

My wife has never used it. Doesn't know it exists, despite me mapping it to her computer and putting things for her in there, and showing her how to get to the things she asked for in that share.

In fairness, I thought it was a good idea because I have spent the last decade or so in corporate IT and engineering roles. This is something that you see fairly regularly in that kind of environment. I've watched these kinds of file shares save YEARS worth of people's work and livlyhood when something happens to their laptop, and I've seen people who hadn't lose just as much.

My wife has not had those same experiences. It's not that she doesn't see the value of having it, or that she's not interested in having it, but that's not how she uses her computer. Anything that she downloads can always be re-downloaded or saved to something like Google Docs.

For me, it is another thing to manage, which I'm happy to do when it's useful. When it's not, it just feels like another burden that I'm trying to figure out how to find time to keep running.

Now, anytime I am thinking about setting up a new shared project or tool, I ask first "Are **we** going to use this, or am I going to want to manage it?" If the answer is no to both quesions, I don't set it up. It's that simple.