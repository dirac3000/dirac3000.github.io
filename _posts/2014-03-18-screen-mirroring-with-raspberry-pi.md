---
id: 24
title: Screen Mirroring with Raspberry Pi
date: 2014-03-18T01:14:09+02:00
author: Alvaro
layout: post
guid: http://alvarom.com/?p=24
permalink: /2014/03/18/screen-mirroring-with-raspberry-pi/
ci_cpt_video_url:
  - ""
ci_cpt_show_gallery:
  - ""
image: /http://alvarom.com/wp-content/uploads/2014/03/rplay-200x200.png
categories:
  - Technology
---
A while ago I bought a [Raspberry Pi](http://www.raspberrypi.org/ "Raspberry Pi website") (model B). It is quite a nice gadget that is rather popular, so you can easily use it for whatever project you might have in mind that needs a low-powered embedded computer. I use it to share my external USB drive contents via samba, as bittorrent client and as a DLNA server. I followed [this guide](http://raspberry.io/projects/view/naspberry-pi/), and it works like a charm. But I wanted to do more&#8230;<!--more-->Looking at the nice quality of the videos on my DLNA-capable TV, I wondered if it was possible to use the TV as a mirrored or second screen for my MacBook. Screen mirroring via wifi is actually trickier than it seems! There aren&#8217;t many established protocols so far. Here&#8217;s what I found out:

  1. Even if I thought it would be possible to stream the content of my screen via **DLNA**, apparently it isn&#8217;t that easy. I don&#8217;t know the protocol well enough to know what are the exact limitations, but I couldn&#8217;t find a solution ready out there.
  2. I thought about installing a **VNC** server on my Mac and a client on the Raspberry Pi. The problem is that I don&#8217;t run X on the RPi. I tried the [DirectVNC](http://freecode.com/projects/directvnc) client, but it was slow and i was not able to run it in full HD. I thought about trying to install X and a fancier  client, but I abandoned the idea because it couldn&#8217;t work as a second monitor anyway, and sound would not have been transmitted.
  3. The most obvious solution would be Apple&#8217;s **AirPlay**. It does exactly what I want, but the receiver should be an Apple TV or an AirPlay-ready device. And I don&#8217;t own one.

The pain with AirPlay is that it is a closed source protocol. I found out about few projects that have succeeded in making it work for video (but not screen mirroring), for example with [Airplayer](https://github.com/PascalW/Airplayer/), used in Plex.

Then I found out about a beta software called [rPlay](http://www.vmlite.com/index.php?option=com_kunena&func=view&catid=23&id=10991), created by a company called VMLite. I asked to participate to the beta, so I received an email and received a license key. That means that you will have to install some proprietary software in your Raspberry Pi. I tried anyway, following the instructions from [this website](http://adventuresandwhathaveyou.wordpress.com/2013/09/02/airplay-mirroring-on-raspberry-pi-with-rplay/).

It did work out!

<img class="alignnone" alt="" src="https://lh4.googleusercontent.com/-46EfmQPJKf4/Ut79yET3N3I/AAAAAAAALYY/itm8VYv9I98/w1050-h777-no/IMG_20140122_000735.jpg" width="945" height="699" /> 

The resolution is good and the latency is acceptable. And it was easy to set up!

One problem was the instability of the server. In order to activate the mirroring or the second screen, you need to have access to a small menu on the menu bar:

<a href="http://alvarom.com/wp-content/uploads/2014/03/rplay.png" rel="fancybox[24]"><img class="alignnone size-full wp-image-26" alt="rplay" src="http://alvarom.com/wp-content/uploads/2014/03/rplay.png" width="269" height="245" /></a>

&nbsp;

The problem is that the menu does not show up all the time, and it is quite unreliable.  
The other problem was that some applications apparently make the external monitor freeze, and sometimes rPlay crashes completely. I imagine that the fact I was running a beta version justifies these problems. But the project is not open source and it isn&#8217;t trivial to get support for it. This can be frustrating.

So for now I abandoned rPlay. I might try other options, for example contacting the [AirTame](http://airtame.com/) guys and ask them about the sources of their project, or to try to reproduce what this guy [has done](http://realmike.org/blog/2011/02/09/live-desktop-streaming-via-dlna-on-gnulinux/) with a Linux machine and UPNP.