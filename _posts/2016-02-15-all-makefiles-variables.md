---
id: 83
title: All Makefiles Variables
date: 2016-02-15T18:13:06+02:00
author: Alvaro
layout: post
guid: http://alvarom.com/?p=83
permalink: /2016/02/15/all-makefiles-variables/
ci_cpt_video_url:
  - ""
ci_cpt_show_gallery:
  - ""
categories:
  - Alvaro M
  - Programming
  - Technology
tags:
  - build
  - Makefile
---
Makefiles can be very complicated. Recently I have found on [stackoverflow](http://stackoverflow.com/questions/7117978/gnu-make-list-the-values-of-all-variables-or-macros-in-a-particular-run) a way to print out all variables. My favourite variant was this:<!--more-->

<p style="padding-left: 30px;">
  <code>$(foreach v, $(.VARIABLES), $(info $(v) = $($(v))))</code>
</p>

That I find simple and useful.