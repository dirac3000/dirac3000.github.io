---
id: 76
title: Linux User Space Drivers with Interrupts
date: 2014-12-17T12:08:08+02:00
author: Alvaro
layout: post
guid: http://alvarom.com/?p=76
permalink: /2014/12/17/linux-user-space-drivers-with-interrupts/
ci_cpt_video_url:
  - ""
ci_cpt_show_gallery:
  - ""
categories:
  - Alvaro M
  - Programming
  - Technology
tags:
  - kerne
  - l linux
  - programming
  - uio
---
Writing device drivers in Linux requires some kernel knowledge and some rules must be followed. Writing a user space application that drives the device can be simpler to write and debug. This article explains how to implement a simple user space driver to a memory-mapped device.<!--more-->

Some time ago I needed to port a software that was driving a custom device. The interface between the operating system and the device had changed though. The old software had a custom interface to read/write to the device memory, and to handle the interrupts. The new device is a memory-mapped IP that sits on an ARM based SoC, so in theory the communication layers are simpler to deal with.

My first solution was to write the user space application using _/dev/mem_ to access to physical addresses. This seemed to work fine, but I still had to poll a register value to check for the interrupt.

Then I came across the User I/O (UIO) driver, and a [blog entry](https://fpgacpu.wordpress.com/2013/05/28/how-to-design-and-access-a-memory-mapped-device-part-two/) that explained how it was working. Even better, I figured out that there was a generic interrupt version of the driver, thus making it even easier to use. So here are are the steps I followed.

  1. In you kernel configuration, select UIO\_PDRV\_GENIRQ. I chose it as built-in, but you can build it as a separate module.
  2. In you kernel device tree source, you have to define you UIO device: 
```
mydev@70000000 {
 compatible = "my-uio";
 reg = <0x70000000 0x80000>, /* zone 1 and size */
       <0x80000000 0x40000>; /* zone 2 and size */
 reg-names = "zone1", "zone2"; /* Names of the zones */
 interrupts = <31>; /* interrupt the device generates */
};
```
  3. Add &#8220;_uio\_pdrv\_genirq.of_id=my-uio_&#8221; to the boot parameters, so the _uio\_pdrv\_genirq_ driver will look for the device marked compatible.

Note the &#8220;_reg_&#8221; section, that define the list of memory regions accessible via UIO driver. That&#8217;s all we need to do on the kernel side. If we compile and reboot, you should be able to see the mapped devices on _/sys/devices/70000000.mydev/uio/uio0/maps/mapX/_. A device node will be available on _/dev/uio0_.  
In order to use the device, you need to open it, then map it using the mmap function. For example, this program writes the value 1 at the address 0x8000000, mapped in the zone 2:

<pre>#include &lt;stdio.h&gt;
#include &lt;stdlib.h&gt;
#include &lt;errno.h&gt;
#include &lt;signal.h&gt;
#include &lt;fcntl.h&gt;
#include &lt;sys/mman.h&gt;
#include &lt;unistd.h&gt;

#define PAGE_SIZE 4096UL
#define PAGE_MASK (PAGE_SIZE - 1)

int main(int argc, char **argv) {
 int fd;
 void *map_base, *virt_addr;
 unsigned long writeval = 0x1;
 off_t target = 0x80000000;

 if ((fd = open("/dev/uio0", O_RDWR | O_SYNC)) == -1)
  return -1;
 /* Map one page */
 map_base = mmap(0, PAGE_SIZE, PROT_READ | PROT_WRITE, MAP_SHARED, fd,
            1 * PAGE_SIZE);
 if (map_base == (void *)-1)
   return -1;

 virt_addr = (void *)((off_t)map_base + (target & MAP_MASK));
 *((unsigned long *)virt_addr) = writeval;

 if (munmap(map_base, MAP_SIZE) == -1)
   return -1;
 close(fd);
 return 0;
}</pre>

Note that we map the device with offset 1 * PAGE_SIZE. This is to tell the UIO driver we want to access to the zone 2 (to access zone 1 we would have used offset at 0).

Finally, the interrupts. In order to enable the interrupt check, we need to write a signed 32 bit int value &#8220;1&#8221; to _/dev/uio0_. We can then check for the interrupt using the &#8220;select&#8221; function. When the interrupt kicks in, the UIO driver interrupt handler will notify the poll function that will wake up the select. All we need to do then is to read a signed 32 bit value int from _/dev/uio0_ to acknowledge the event.

### TL; DR

If you want to write a software that drives a memory mapped device form user space, you can use the _uio\_pdrv\_genirq _driver. This will allow you to access the device from user space without writing a single line of C code in kernel space, and using a simple select to wait for the interrupt.