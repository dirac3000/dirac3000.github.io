---
id: 171
title: Docker on Windows/Cygwin
date: 2017-09-19T14:05:23+02:00
author: Alvaro
layout: post
guid: http://alvarom.com/?p=171
permalink: /2017/09/19/docker-on-windowscygwin/
ci_cpt_video_url:
  - ""
ci_cpt_show_gallery:
  - ""
categories:
  - Notes
  - Programming
  - Technology
---
Sometimes I need to work on Windows, and when I tinker a bit I often end up the Cygwin based [MobaXterm](http://mobaxterm.mobatek.net/), the excellent _unixish_ terminal + X server for Windows. Few days I started also playing with Docker on windows, and when I tried to execute it from the shell I ended up with this message:

`$ docker run -it ubuntu bash<br />
the input device is not a TTY. If you are using mintty, try prefixing the command with 'winpty'`

You obviously couldn&#8217;t just apt-get winpty, that would have been too easy, but you can quickly download [Winpty](https://github.com/rprichard/winpty) from their [release page](https://github.com/rprichard/winpty/releases/). I picked the 0.4.3 cygwin x64 version and put the binary in my $PATH, so it was easy to prefix the previous command:

`$ winpty docker run -it ubuntu bash<br />
root@5af561cc25b8:/# uname -a<br />
Linux 5af561cc25b8 4.9.41-moby #1 SMP Wed Sep 6 00:05:16 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux`