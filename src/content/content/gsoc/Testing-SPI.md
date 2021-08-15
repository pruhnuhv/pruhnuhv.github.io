---
title: "Testing - SPI"
date: 2021-08-10T00:00:10+05:30
draft: true
---

<b> RTEMS libi2c :</b>
The RTEMS libi2c API also supports SPI devices, the SPI on the raspberry Pi BSP is based on this libi2c API instead of the separate /dev/spi API that is used in a few other RTEMS BSPs.

<b>The Driver :</b>
In our driver, like we did for the I2C driver, we first define the read and write functions to interact with the sensor. According to the datasheet, writing is done by lowering the CSB and sending control bytes and register data pairs. The control bytes consist of the register address with the bit7 as 0. We pass the void* variable to our read/write functions and utilise the `rtems_libio_rw_args_t` parameter block for reading/writing.
```
   typedef struct {
     rtems_libio_t          *iop;
     off_t                   offset;
     char                   *buffer;
     uint32_t                count;
     uint32_t                flags;
     uint32_t                bytes_moved;
   } rtems_libio_rw_args_t;
   
```
We can access the offset, count, number of bytes moved, and a corresponding buffer for our spi read/write operations. Like a standard SPI operation, we first start the bus and then set the transfer mode. We use the `rtems_libi2c_tfr_mode_t` structure for setting the transfer mode (bit rate, clock phase and polaritiy, bits per char...).
```
  static rtems_libi2c_tfr_mode_t tfr_mode = {
    .baudrate = 100000,
    .bits_per_char = 8,
    .lsb_first = FALSE,
    .clock_inv = TRUE,
    .clock_phs = FALSE
  };

  /* Set transfer mode */
      sc = rtems_libi2c_ioctl(minor, RTEMS_LIBI2C_IOCTL_SET_TFRMODE, &tfr_mode);

      if ( sc != RTEMS_SUCCESSFUL ) {
        return sc;
      }
```

Then we address the device and can send our write instruction and address (in this case, the control byte), followed by the data to be written and consequently terminate the transfer.

Similary, for the read operation, the steps remain the same, except for the fact that we change the control byte, as this time, according to the datasheet, we need to set the bit7 to 1 for the read operation.

<b>Gettting The Temperature :</b>
Once The basic read and write functions are implemented, we just need to set the appropriate configuration as per use, followed by reading the Temperature and calibrating it according to the values returned from the corresponding registers as defined in the datasheet. However, one important detail to make note of here is that we need to declare and allocate memory for a rtems libio read/write args struct, which we will be passing to the read/write functions we just defined.

<b>Registering the driver and IOCTL :</b>
Yet again, assuming that you have gone through the code itself, I'll be skipping over the details of how the Temperature is being read and calibrated (all of it is also clearly stated in the datasheet). We now move to registering the driver and implementing the IOCTL for it. One mistake that I was making initially was using `spi_dev_alloc_and_init()` which isn't even a part of the libi2c API. (It is a part of the /dev/spi implementation of the SPI which the Raspberry Pi BSP doesn't use). We use `rtems_libi2c_register_drv()` 
```
  rtems_libi2c_register_drv(
             "BMP280-SPI",
             &spi_bmp280_rw_drv_t,
             spi_bus,
              0x00
           );
```

With this call, libi2c is informed, that: a device is attached to the given "bus" number.
  The device is managed by a driver, who's entry functions are listed
  in "drvtbl"
  the device should be registered with the given "name"(BMP280-SPI) in the device
  tree of the filesystem.
And that completes our SPI Driver.
