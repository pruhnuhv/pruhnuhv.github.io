---
title: "Testing - I2C"
date: 2021-07-25T00:02:20+05:30
draft: true
---

<b>RTEMS libi2c :</b>
RTEMS provides a standard API to I2C or SPI based drivers. So my driver for the BMP280 as well as the corresponding test is based upon this libi2c API. 

<b> BMP280 :</b>
BMP280 is a widely available Temperature and Pressure sensor, typically used with Arduinos and Raspberry Pi's, hence there is a lot of support and documentation for the same.

<b>The Driver :</b>
In our driver, we initially define the read and write functions to interact with the driver. The `i2c_bus_transfer` function from the RTEMS i2c library, takes the bus control, the messages to transfer along with a count of the number of messsages* being transferred. Here our R/W functions are primarily reading/writing only one byte at a time. (* One message here is one complete transfer of n bytes) 

The I2C Write function of our driver according to the BMP280 datasheet, requires a pair of bytes, ie. the control byte (register address), and a data byte to be transferred, as is standard. Hence we implement this function by passing the appropriate i2c_msg transfer message of length 2 to the `i2c_bus_transfer` function.

```
  i2c_msg msg = {
    .addr = dev->address,
    .flags = 0,
    .len = 2,
    .buf = content
  };
```
Similarly, the I2C Read function for the driver, again like standard, requires one complete message to be sent first with the control byte (register address), followed by another message, this time with the I2C_M_RD flag set, following which the slave sends out the data.

```
  i2c_msg msg[2] = {
    {
    .addr = dev->address,
    .flags = 0,
    .len = 1,
    .buf = &reg
    },
    {
    .addr = dev->address,
    .flags = I2C_M_RD,
    .len = 1,
    .buf = content
    }
  }
```

Once The basic read and write functions are implemented, we just need to set the appropriate configuration as per use, followed by reading the Temperature and calibrating it according to the values returned from the corresponding registers as defined in the datasheet.

<b>Registering the Driver & IOCTL :</b>
Now, assuming you have gone through the code, skipping over the nitty-gritties of how the Temperature is being read and calibrated, we now move to registering the driver and implementing the IOCTL functionality for it. Libi2c provides us with `i2c_dev_alloc_and_init()` that allocates a device control from the heap and initializes it. Further we use this initialized device, and register it using `i2c_dev_register` and that completes our driver.


