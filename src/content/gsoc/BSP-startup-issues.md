---
title: "BSP Startup issues"
date: 2021-06-15T02:02:26+05:30
draft: true
---

<b>Some context, to the blog post ahead: </b>
It's been a week into the coding period of the GSoC'21 at RTEMS, more than a week since I actually started working, and a few meets and emails have been exchanged with the mentors and the ever-so-helpful RTEMS community.

<b>Initial Hiccups:</b>
I started off trying to build the RTEMS 6 (henceforth referred to as Master) Raspberry Pi BSPs. There's two(rather, three) ways to build the BSPs. You could use the RTEMS-Source-Builder(we'll call it the RSB), you can use the (although now inching towards it demise) Autoconf-make build (Refer the 'Manually Build a BSP' section in the RTEMS 5 docs), and the 3rd option is the waf build. I wasn't able to build the Raspberry Pi BSPs using the first two, basically, the RSB/the Autoconf build failed, and being new to the entire RTEMS toolchain and directory structure, if I say that I wasn't intimidated by it, I'd be lying, luckily for me, the waf build went through without a problem and so I ignored the first two builds failing. (More on this later)

Sigh! The Rpi BSPs didn't boot on their respective hardware(s), A lot of SD card mount-unmount later, I moved to the RTEMS 5 BSPs to see if I could make them work.

<b>The SD card:</b>
The SD card has the bootfiles necessary for the Pi to boot up, as the BSPs weren't even starting up, tftp was out of question at the moment. A look at the Rpi Docs will tell you that you need to get the bootcode.bin, start.elf (atleast these two), and the other files from the official [Raspberry Pi Repository](https://github.com/raspberrypi/firmware/tree/master/boot)
Additionally, we add a config.txt to use/modify the config parameters that will be used to set up the Pi.

<b>Bisecting:</b> 
Git bisect is an amazing tool to go through the commits to find the commit that broke the application. The RTEMS 5 Rpi BSPs were working with an older version of the official Rpi firmware but weren't working with the latest/master branch firmware. Hence, we were able to narrow it down to a certain [version](https://github.com/raspberrypi/firmware/tree/1.20200601/boot) of the official firmware, after which the BSP doesn't boot. Without a JTAG/other appropriate debug options, I wasn't able to debug this problem and we decided on updating the docs to point to the older firmware instead of spending time debugging it (for now).

<b>What works, What doesn't:</b>
So, the RTEMS 5 branch BSPs, now worked with the older firmware, the RTEMS 6 branch BSPs still don't work.

<b>What commit broke it, again:</b>
Again, going through the commits, we could find the commit that broke the BSP!

The Commit 272534eb725f2486b7a32b39d998202a101bd36e on the RTEMS 6 branch, removed the following statement from the bspstarthooks.c file and it had to be re-added to get the BSPs to work:

```
 /* Clear Secure or Non-secure Vector Base Address Register */
  arm_cp15_set_vector_base_address(bsp_vector_table_begin);
```

An Important Thing to note here: To build this version of the BSP, we had to revert to an older version of the RSB (d3dc0bc3861362978cdf65725e4ba2b64e283d32).
When we added the function from the commit that broke the BSP, back to the latest RTEMS master, we had to build the BSP using the waf build system, as the Build with the Autoconf Build System Failed. 


