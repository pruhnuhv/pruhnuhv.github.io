---
title: "Waf And Autoconf"
date: 2021-06-30T00:00:50+05:30
draft: true
---

<b> Autoconf: </b>
The Autoconf Build for the master branch was failing because of the absence of the rtems_bsdnet.h file and I had to disable networking (--disable-networking) while building the BSP, after which I was able to build the master Branch BSP successfully.

Well, funny enough, using our temporary patch from my previous post, I was able to get the BSP to boot and bring the console up when built with Autoconf. I had spent the past few days trying to fix and patch the console, vector tables, mmu and the startup code, in the end the issue turned out to be related to the Build System :)



<b>Waf: </b>
We were so close yet so far, to get the Basic RPi BSPs to boot it was still essential to get the waf Build of the same to work. The waf build wasn't failing, I tried out a lot of options in the config.ini, the build was working, but the BSP itself wasn't booting on actual Hardware. 

Eventually, I compared the bspopts.h and cpuopts.h files in the builds of both of the BSPs. 

```
#define BSP_FDT_BLOB_COPY_TO_READ_ONLY_LOAD_AREA 1
 
#define BSP_FDT_BLOB_READ_ONLY 1

#define BSP_START_COPY_FDT_FROM_U_BOOT 1

#define BSP_FDT_BLOB_SIZE_MAX 262144

```
These were the 4 defines that were missing from the waf build but were present in the Autoconf build. 
These yaml files were to be added to resolve this issue:

```
- role: build-dependency
  uid: optfdtcpyro
- role: build-dependency
  uid: optfdtmxsz
- role: build-dependency
  uid: optfdtro
- role: build-dependency
  uid: optfdtuboot
  ```

The files just had to be moved out from the BSP specific directories to the shared directory. 


<b></b>
<b><b> Finally, both the Raspberry Pi BSPs in the master branch are working. </b></b>
