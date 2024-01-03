---
title: "GSoC"
date: 2021-08-16T02:02:26+05:30
draft: true
---

<b>GSoC Conclusion :</b>
GSoC'21 officially kicked off on the 7th of June. The ideation, programming, mentorship, and consequently GSoC come to an end today, on the 16th of August.
This blog aims to be a brief report on all that has happened since.

<h1><u>The Raspberry Pi BSP </u></h1>

<b>Before :</b>
 
- The RTEMS 5 branch boots up on a couple of models. SMP doesn't work at all. 
  
- RTEMS 6 branch doesn't boot up on any model.
  
- No idea about the config/firmware to be used, can't test peripherals on this branch.

<b>Current State :</b>

- The RTEMS 5 and 6 branches boot up on all the models except the Pi 4.

- SMP works.

- Support for SPI, I2C and the GPIO pins are tested. We also know the limitations and have example drivers for the same.


<h2> Startup Issues :</h2>

The BSP not booting up had more than one aspect to it, that we later realised.

The configurations in the config.txt file, the fix for which we've added to Docs. 

The correct firmware version, which has been updated in the Docs as well.

The issues with the ARMv6 Board specific Startup code, the patch for which has been applied [here.](https://github.com/pruhnuhv/rki2/tree/rtems6/build/rpi/src)

The yaml files for FDT support missing, which were causing problems with the waf build. The patch for which has been applied [here.](https://github.com/pruhnuhv/rtems/commit/c71e34bee0d6bf47c743fd390cc2f420c67b49ff)



<h2> GPIO :</h2>


We tested the GPIO using some LEDs and switches. The GPIO on the BSP works fine. We had a tentative patch for the GPIO code to add some small functionalities, but we have decided against applying that patch, more details about the same can be found in the GPIO blogs. The polled and interrupt driven tests that we used can be found in the `gpio_cmd` and `gpio_irq_cmd` files [here](https://github.com/pruhnuhv/rki2/tree/rtems6/build/rpi/src)



<h2> I2C :</h2>


We tested I2C on the BSP using a BMP280 Temperature sensor. The I2C works fine. The drivers and the application that we wrote can be found in the `drivers/BMP280` and `bmp280_cmd` files [here](https://github.com/pruhnuhv/rki2/tree/rtems6/build/rpi/src)




<h2> SPI :</h2>


We tested SPI on the BSP using a BMP280 Temperature sensor. The SPI works fine too. The drivers and the application that we wrote can be found in the `drivers/BMP280-SPI` and `bmp280_spi_cmd` files [here](https://github.com/pruhnuhv/rki2/tree/rtems6/build/rpi/src)



<h2> SMP :</h2>


The SMP not working can be attributed to changes to the firmware from the bcm/rpi that the users are not aware about. A simple one line patch to the SMP code got the SMP working again. The patch can be found [here](https://github.com/pruhnuhv/rtems/commit/1c0606819666241b5f9d6470a3f11813a6feddb8)


<h2><b> Further Work :</b></h2>


The Raspberry Pi BSPs are now in a pretty good shape. Further, work is to to be done on adding support for the Pi 4 as well as Adding Libbsd support for the BSP.
