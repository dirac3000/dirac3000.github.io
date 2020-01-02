---
id: 45
title: Checkinstall to Create Deb Packages from Source
date: 2014-06-26T12:55:44+02:00
author: Alvaro
layout: post
guid: http://alvarom.com/?p=45
permalink: /2014/06/26/checkinstall-to-create-deb-packages-from-source/
ci_cpt_video_url:
  - ""
ci_cpt_show_gallery:
  - ""
categories:
  - Alvaro M
  - Programming
  - Technology
tags:
  - checkinstall linux install tips
---
I really like my Ubuntu 12.04. I don&#8217;t use the Unity shell, but the system underneath is stable enough and I have plenty of packages.  
From time to time however, I need to install some binary or library that it is not available as package. So I download the source compile and I am ready to install it. The problem is that I want to be able to easily uninstall the package, as easy as I install it!  
That&#8217;s when [checkinstall](http://freecode.com/projects/checkinstall) comes into place. It&#8217;s not a new project but a handy one. You can install it in your Debian or Ubuntu distribution and it allows you to intercept the &#8220;make install&#8221; command and create the Debian package you need. For example, you can do stuff like:

```shell
./configure
make
sudo checkinstall make install
```

You answer few questions about the package description (so later on you will remember what was it for) and then checkinstall will create the package and install it for you. Note that I tried it on Ubuntu, but it will work on any Debian, RPM or Slackware based distribution. You can now delete the original source and later on you will be able to remove the package just by typing:

```
apt-get remove <package>
```

Sweet!