---
id: 19
title: Mac and gcc
date: 2014-01-25T21:15:23+02:00
author: Alvaro
layout: post
guid: http://alvarom.com/?p=19
permalink: /2014/01/25/mac-and-gcc/
ci_cpt_video_url:
  - ""
ci_cpt_show_gallery:
  - ""
categories:
  - Programming
  - Technology
---
I wanted to compile a little test program I wrote in C. The problem is that, in order to compile on a Mac using the command line, you need to configure the XCode command line tools. You can find this information on the web easily, but I couldn&#8217;t get the thing working properly.

In my case, I had XCode installed and at some point I used to have the command lines tools. Then I updated to Mavericks (OSX 9). So gcc could not find the standard libc includes.

> <pre>amoran@Cuzco:~/dev/test-rdl$ gcc -o test test.c
test.c:1:19: error: stdio.h: No such file or directory</pre>

I found out there is a tool called _xcode-select_ that should be able to &#8220;print or change the path to the active developer directory&#8221;. I didn&#8217;t understand exactly what it is for (I suspect it is to handle several Xcode versions), but it looked like the tool I was looking for. In the end the solution was just to reinstall the Xcode command line tools, by using an undocumented option:

> <pre>xcode-select --install</pre>

Once you finish installing, you can use gcc as in any standard Linux box.