---
id: 52
title: Profiling Applications on Linux
date: 2014-08-25T23:28:53+02:00
author: Alvaro
layout: post
guid: http://alvarom.com/?p=52
permalink: /2014/08/25/profiling-applications-on-linux/
ci_cpt_video_url:
  - ""
ci_cpt_show_gallery:
  - ""
image: /http://alvarom.com/wp-content/uploads/2014/08/profile_graph-200x200.png
categories:
  - Alvaro M
  - Programming
  - Technology
tags:
  - calgrind
  - gprof
  - linux
  - oprofile
  - perf
  - pprof
  - profile
  - programming
  - vtune
---

{:refdef: style="text-align: center;"}
![Profile graph](http://alvarom.com/wp-content/uploads/2014/08/profile_graph.png)
{: refdef}

A while ago we developed an application that runs on Linux. I wanted to profile it in order to understand if we could make it run faster. That&#8217;s why I spent some time looking at some profilers, and here&#8217;s a quick round of what I have learned.  
I created a [multithread test program](https://gist.github.com/dirac3000/28db406b487d63b67e40) as an example that I want to analyse to understand where the time is spent.

### In The Beginning There Was Gprof

When I said I wanted to profile an application, the quickest answer to the problem seemed to be &#8220;use gprof&#8221;. To use this profiler, you need to compile your code with the &#8220;-pg&#8221; flag in order to have gcc to instrument your code. You then run the binary and it produces a log, gmon.out. You can then analyse the output with gprof. That&#8217;s what you get:

<pre>Flat profile:

Each sample counts as 0.01 seconds.
%         cumulative   self  self  total 
time       seconds   seconds calls Ts/call Ts/call name 
57.51      0.04         0.04                       compare
14.38      0.05         0.01                       action
14.38      0.06         0.01                       frame_dummy
</pre>

We can see where we spend 86.27% of the time, that _compare_ takes quite a while. That&#8217;s good for a start, but it doesn&#8217;t really point the finger on what to look at. Searching on the internet I also realized that:

  1. The gprof method is a statistical method that only takes 100 samples per second.
  2. It was designed in the &#8217;80s, with no thread programming in mind.
  3. It takes many [assumptions](https://sourceware.org/binutils/docs/gprof/Assumptions.html#Assumptions) on the collected data, often leading to wrong conclusions.

Time to look for another profiler.

### Valgrind and Callgrind

[Valgrind](http://valgrind.org/) is a tool well known for finding memory leaks and other memory issues in C/C++ code. Valgrind has a tool called callgrind that allows to record the call history of a program&#8217;s run. The application will run very slowly, but it will generate a file containing detailed call information.

<pre class="brush: plain; title: ; notranslate" title="">valgrind --tool=callgrind ./testoprof</pre>

Note that it makes your program run very very slow.  
You can then use _callgrind_annotate_ to read the results, here you can see an example output:

<pre class="brush: plain; title: ; notranslate" title="">--------------------------------------------------------------------------------
Profile data file 'callgrind.out.26274' (creator: callgrind-3.8.1)
--------------------------------------------------------------------------------

[...]
--------------------------------------------------------------------------------
Ir file:function
--------------------------------------------------------------------------------
1,292,858,953 /build/buildd/eglibc-2.15/stdlib/random_r.c:random_r [/lib/x86_64-linux-gnu/libc-2.15.so]
1,002,000,013 /build/buildd/eglibc-2.15/string/strfry.c:strfry [/lib/x86_64-linux-gnu/libc-2.15.so]
925,841,104 /build/buildd/eglibc-2.15/string/../sysdeps/x86_64/multiarch/strcmp-sse42.S:__strcmp_sse42 [/lib/x86_64-linux-gnu/libc-2.15.so]
629,738,640 /build/buildd/eglibc-2.15/stdlib/msort.c:msort_with_tmp.part.0'2 [/lib/x86_64-linux-gnu/libc-2.15.so]
387,418,592 /home/users/amoran/tmp/testoprof/testoprof.c:compare [/home/users/amoran/Perso/tmp/testoprof/testoprof]
299,307,940 /build/buildd/eglibc-2.15/malloc/malloc.c:_int_malloc [/lib/x86_64-linux-gnu/libc-2.15.so]
203,654,096 /build/buildd/eglibc-2.15/string/../sysdeps/x86_64/multiarch/../memcpy.S:__GI_memcpy [/lib/x86_64-linux-gnu/libc-2.15.so]
108,000,000 /build/buildd/eglibc-2.15/string/../sysdeps/x86_64/multiarch/../strlen.S:__GI_strlen [/lib/x86_64-linux-gnu/libc-2.15.so]
104,000,272 /build/buildd/eglibc-2.15/malloc/malloc.c:malloc [/lib/x86_64-linux-gnu/libc-2.15.so]
34,000,000 /build/buildd/eglibc-2.15/string/strdup.c:strdup [/lib/x86_64-linux-gnu/libc-2.15.so]
30,998,796 /build/buildd/eglibc-2.15/stdlib/msort.c:msort_with_tmp.part.0 [/lib/x86_64-linux-gnu/libc-2.15.so]
</pre>

Note that Callgrind can be very slow. According to the documentation Callgrind is based on Cachegrind, and the [latter](http://valgrind.org/docs/manual/cg-manual.html) &#8220;simulates how your program interacts with a machine&#8217;s cache hierarchy and (optionally) branch predictor&#8221;. While this is a great thing, it also means that it simulates some hardware, thus slowing down the execution and sometimes it might give different results of what is expected.  
Here we clearly see that the program actually spends a lot of time executing the function strfry (a function that randomizes the content of a string). That actually makes sense. But I still have a hard way to read the output.

I looked for a way to better visualize the callgrind data. KCachegrind is a tool for KDE that does that very well, however I just compiled the slimmer QCachegrind from sources to avoid the hassle of installing all the KDE dependencies:

<pre class="brush: plain; title: ; notranslate" title="">git clone git://anongit.kde.org/kcachegrind
 cd kcachegrind/qcachegrind
 qmake
 make
</pre>

Here&#8217;s a view of QCachegrind:

<img class="alignnone" src="https://lh5.googleusercontent.com/-ttlWSAnJcP0/U_pbpeicptI/AAAAAAAAkFk/jx-YE1dy5EY/w849-h582-no/qcachegrind.png" alt="" width="849" height="582" /> 

### Google Performance Tools

Google developed some tools to enhance the performance in their application, The [Google Performance Tools](https://code.google.com/p/gperftools/wiki/GooglePerformanceTools) (GPT). They say that &#8220;Perftools is a collection of a high-performance multi-threaded malloc() implementation, plus some pretty nifty performance analysis tools&#8221;. I focused mainly on the performance analysis tools. The good thing is that it doesn&#8217;t need instrumentation, and even if some people say it is not as accurate as callgrind, it is very fast, thus making it worth using during the development.

In order to use GPT, they recommend to link it together with the library. I preferred just to use an environmental variable at launch to enable the profiler,

<pre class="brush: plain; title: ; notranslate" title="">LD_PRELOAD=/usr/lib/libprofiler.so.0.1.0 CPUPROFILE_FREQUENCY=130000 \
CPUPROFILE=./cpuprofile.log ./testoprof
</pre>

The CPUPROFILE_FREQUENCY variable will set the number of probes performed every second.  
In order to visualize profiling information, you can use google-pprof.

<pre class="brush: plain; title: ; notranslate" title="">google-pprof -gv ./testoprof cpuprofile.log
</pre>

You can also convert it to other formats, notably to pdf or callgrind (so later you can visualize it with QCachegrind).

### Perf

[Perf](https://perf.wiki.kernel.org) is a tool to perform lightweight system-wide profiling using the Linux kernel counters and tracepoints. It will probe the program every time an event happens. It is available in the _linux-tools_ package on Ubuntu. To profile an application, you can use the perf record and perf report comands:

<pre class="brush: plain; title: ; notranslate" title="">perf record -g ./testoprof
 perf report --stdio -i perf.data
</pre>

The _-g_ flag in record will enable the call graph record. The report will then look like this:

<pre class="brush: plain; title: ; notranslate" title=""># Overhead Command Shared Object Symbol
# ........ ......... ................. .............................
#
61.40% testoprof libc-2.15.so [.] __random_r
|
--- __random_r

16.49% testoprof libc-2.15.so [.] strfry
|
--- strfry
|
--100.00%-- start_thread

10.96% testoprof libc-2.15.so [.] malloc
|
--- malloc
|
--100.00%-- (nil)
...
</pre>

A somewhat clearer view could be obtained using perf script and a very handy script called [gprof2dot](https://code.google.com/p/jrfonseca/wiki/Gprof2Dot):

<pre class="brush: plain; title: ; notranslate" title="">perf script -i perf.data | gprof2dot.py -w -f perf | dot -Tsvg &gt; perf_out.svg
</pre>

What I see though, is that it is a bit harder to see the relationship between _\_random\_r and strfry. This might be due to the implementation of strfry, but even looking at [that](http://fossies.org/dox/glibc-2.19/strfry_8c_source.html) I couldn&#8217;t really understand why I couldn&#8217;t get a clear explanation.  
EDIT: I noticed that perhaps the call graph is inverted (the branches being the callers), though I am not sure about it.

### Oprofile

[Oprofile](http://oprofile.sourceforge.net) is another system-wide profiler that you can use to analyse the performance of an application on Linux. It uses hardware counters exposed by the kernel (so in this way it should not be very different from perf).  For some reason, it appears that the package is not available on Ubuntu 12.04. I recommend you to get the git sources, as the last release (1.0.0 to be released soon) appears to have few problems that have been fixed upstream. Follow these steps to build it:

<pre class="brush: plain; title: ; notranslate" title="">git clone git://git.code.sf.net/p/oprofile/oprofile oprofile-oprofile
 cd oprofile-oprofile
 ./autogen.sh
 ./configure
 make
 sudo make install
</pre>

(If you prefer you can use [checkinstall](http://alvarom.com/2014/06/26/checkinstall-to-create-deb-packages-from-source/) instead).

To profile your application with oprofile, you can use the command operf:

<pre class="brush: plain; title: ; notranslate" title="">operf -g --events=CPU_CLK_UNHALTED:130000 -t  ./testoprof
</pre>

The _-g_ tells operf to generate a call graph. I also increased the _CPU\_CLK\_UNHALTED_ default value to avoid missing data.

<pre class="brush: plain; title: ; notranslate" title="">opreport -c -g
</pre>

This will give you a big report of the activity of your program, annotated with the available symbols. Here is an output example:

<pre class="brush: plain; title: ; notranslate" title="">samples % linenr info image name symbol name
-------------------------------------------------------------------------------
15 100.000 pthread_create.c:231 libpthread-2.15.so start_thread
89977 63.0515 random_r.c:367 libc-2.15.so random_r
89977 99.4771 random_r.c:367 libc-2.15.so random_r [self]
175 0.1935 (no location information) kallsyms invalidate_interrupt0
80 0.0884 (no location information) kallsyms invalidate_interrupt1
59 0.0652 (no location information) kallsyms invalidate_interrupt3
57 0.0630 (no location information) kallsyms invalidate_interrupt2
51 0.0564 (no location information) kallsyms apic_timer_interrupt
18 0.0199 (no location information) kallsyms retint_careful
12 0.0133 (no location information) kallsyms ret_from_intr
8 0.0088 (no location information) kallsyms reschedule_interrupt
5 0.0055 (no location information) kallsyms system_call
3 0.0033 (no location information) kallsyms system_call_after_swapgs
2 0.0022 (no location information) kallsyms smp_invalidate_interrupt
2 0.0022 (no location information) kallsyms restore_args
1 0.0011 (no location information) kallsyms retint_swapgs
-------------------------------------------------------------------------------
118 100.000 pthread_create.c:231 libpthread-2.15.so start_thread
22523 15.7830 strfry.c:26 libc-2.15.so strfry
22523 99.2640 strfry.c:26 libc-2.15.so strfry [self]
....
</pre>

NOTE: If you run operf with root privileges, the data will be more relevant, as it will be able to gather some information from the kernel symbols (as shown above).  
In this case, we figure out that the program spends a long time on _random_r_ and _strfry_ functions. In order to better visualize the output, You can convert the output data with _gprof2dot_:

<pre class="brush: plain; title: ; notranslate" title="">opreport -c -g &gt; opreport.out
 gprof2dot.py -f oprofile opreport.out | dot -Tsvg &gt; profile_graph.svg
</pre>

Here you can see the output:

<a href="http://alvarom.com/wp-content/uploads/2014/08/profile_graph.png" rel="fancybox[52]"><img class="alignnone size-medium wp-image-53" src="http://alvarom.com/wp-content/uploads/2014/08/profile_graph-300x247.png" alt="profile_graph" width="300" height="247" srcset="http://alvarom.com/wp-content/uploads/2014/08/profile_graph-300x247.png 300w, http://alvarom.com/wp-content/uploads/2014/08/profile_graph.png 400w" sizes="(max-width: 300px) 100vw, 300px" /></a>

### VTune Amplifier

All the profilers mentioned so far are open source and freely available. A colleague told me about Intel VTune Amplifier and I decided to give it a go. VTune is a commercial software provided by Intel, it cost around $899 for a license. I downloaded the trial version, had a bit of trouble installing it but in the end I made it work with the help of their support team.  
VTrune installs a kernel module called _vtsspp_ that uses _kprobe _(a tracing points in the kernel). I also assume it works in a similar way that _perf_ and _oprofile_ work, but the huge advantage is the very clear and easy to use graphical user interface, especially for multithreaded applications.

<a href="http://alvarom.com/wp-content/uploads/2014/08/vtune.png" rel="fancybox[52]"><img class="alignnone size-medium wp-image-64" src="http://alvarom.com/wp-content/uploads/2014/08/vtune-300x165.png" alt="vtune" width="300" height="165" srcset="http://alvarom.com/wp-content/uploads/2014/08/vtune-300x165.png 300w, http://alvarom.com/wp-content/uploads/2014/08/vtune-1024x565.png 1024w, http://alvarom.com/wp-content/uploads/2014/08/vtune-920x508.png 920w, http://alvarom.com/wp-content/uploads/2014/08/vtune.png 1215w" sizes="(max-width: 300px) 100vw, 300px" /></a>  <a href="http://alvarom.com/wp-content/uploads/2014/08/vtune2.png" rel="fancybox[52]"><img class="alignnone size-medium wp-image-65" src="http://alvarom.com/wp-content/uploads/2014/08/vtune2-300x165.png" alt="vtune2" width="300" height="165" srcset="http://alvarom.com/wp-content/uploads/2014/08/vtune2-300x165.png 300w, http://alvarom.com/wp-content/uploads/2014/08/vtune2-1024x565.png 1024w, http://alvarom.com/wp-content/uploads/2014/08/vtune2-920x508.png 920w, http://alvarom.com/wp-content/uploads/2014/08/vtune2.png 1215w" sizes="(max-width: 300px) 100vw, 300px" /></a>

### Conclusions / TL;DR

Profiling an application can be very tricky. Using the simple test mentioned, different profilers pointed to different directions. In the end I found that

  1. _oprofile_ and _VTune_ were probably the tools that gave the clearest answers.
  2. _Perf_ looks like a great system-wide profiler that give a lot of information on the kernel too, but I found it harder to use it to profile a user space application (perhaps it&#8217;s just lack of experience with this tool).
  3. Google&#8217;s _pprof_ is so fast that it&#8217;s worth using during development, even if the results are not as accurate as the previously mentioned tools.
  4. _Callgrind_ gave me a good idea of the dependencies (call graph), but it&#8217;s very very slow.
  5. I would avoid using _gprof_, given that the other profilers give a much clearer idea of what&#8217;s going on.

### Further Notes

Profilers can be difficult to use and understand, to the point that [some people say they lie](http://yosefk.com/blog/how-profilers-lie-the-cases-of-gprof-and-kcachegrind.html). I really think the tricky thing in profiling is understanding two things:

  1. **What you are looking for.** When you decide to profile a program, what are you really looking for? If your program has many waits or if it accesses the disk frequently, you might want to focus on debugging these kind of operations when they happen. A profiler might give you some information about how the processor do during a run of a given binary, but it might not answer directly to the problem you are trying to solve.
  2. **How the profiler works. **As you can see from the example above, the profiler might point in different directions when you try to understand where time is spent. Even if it can be boring, it is important to understand what does the profiler do and how to read the collected information. Reading the documentation can really save you time.

&nbsp;