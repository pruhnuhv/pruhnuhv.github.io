
---
title: "Testing - GPIO II"
date: 2021-07-11T02:02:26+05:30
draft: true
---

<b> RECAP: </b>
The if...else blocks were just turning out to be a tedious, and not so optimal way of going about the task at hand.
My main task right now was to write a patch for the second bank of GPIO pins, in such a way that I do not add a whole lot of extra complexity to the existing GPIO code of the RPi BSP(s).

<b> The PATCH: </b>
When the pin was on the second bank, the effective pin number reflecting would be it's overhead from 32. ie. pin-32.
Hence, the better way to go about was:
 ```
volatile uint32_t *pin_addr = (uint32_t *) BCM2835_GPIO_REGS_BASE +
-                                             (pin / 10);
+                                           ((pin+bank*32) / 10);
```
so for example, for pin number 43 (ie. bank 1), the 'pin' variable being passed to the function would've been 11 (ie. 43-32), so while calculating the corresponding register's offset from the Base, we add 32 to the 'pin' variable (since bank = 1). (Refer to the 'How the GPIO works' section on the previous blog for more details)

Well, the next thing at hand was tackling these 32 bit registers for each function associated with the GPIO. Each of these 32 bit GPIO function registers were in pairs, one corresponding to pins in the first bank and the other to the second bank.
These register's have a common macro defined in the raspberrypi header file:
```
#define BCM2835_REG(x)           (*(volatile uint32_t *)(x))
```
This meant that, if I were to separately define each of the second bank function registers, I'd have to forcibly use if...else blocks. Ugh.

Again, a quick look at the datasheet, and as expected, these 32 bit function register 'pairs' were stacked one after the other. So, if the GPSET0 was present at (BASE + 0x1C), the GPSET1 would be present at (BASE + 0x20), more importantly, it is (BASE + 0x1C + 0x4). That's right! so, GPSETn would be at (BASE + 0x1C + 0x4*n). Using this knowledge, I modified all the GPIO registers in the header file. It looked something like:

```
-#define BCM2835_GPIO_GPSET0      (BCM2835_GPIO_REGS_BASE + 0x1C)
-#define BCM2835_GPIO_GPCLR0      (BCM2835_GPIO_REGS_BASE + 0x28)
-#define BCM2835_GPIO_GPLEV0      (BCM2835_GPIO_REGS_BASE + 0x34)
-#define BCM2835_GPIO_GPEDS0      (BCM2835_GPIO_REGS_BASE + 0x40)
-#define BCM2835_GPIO_GPREN0      (BCM2835_GPIO_REGS_BASE + 0x4C)
-#define BCM2835_GPIO_GPFEN0      (BCM2835_GPIO_REGS_BASE + 0x58)
-#define BCM2835_GPIO_GPHEN0      (BCM2835_GPIO_REGS_BASE + 0x64)
-#define BCM2835_GPIO_GPLEN0      (BCM2835_GPIO_REGS_BASE + 0x70)
-#define BCM2835_GPIO_GPAREN0     (BCM2835_GPIO_REGS_BASE + 0x7C)
-#define BCM2835_GPIO_GPAFEN0     (BCM2835_GPIO_REGS_BASE + 0x88)
-#define BCM2835_GPIO_GPPUD       (BCM2835_GPIO_REGS_BASE + 0x94)
-#define BCM2835_GPIO_GPPUDCLK0   (BCM2835_GPIO_REGS_BASE + 0x98)
+#define BCM2835_GPIO_GPSET(_bank)    (BCM2835_GPIO_REGS_BASE + 0x1C + 0x4*_bank)
+#define BCM2835_GPIO_GPCLR(_bank)    (BCM2835_GPIO_REGS_BASE + 0x28 + 0x4*_bank)
+#define BCM2835_GPIO_GPLEV(_bank)    (BCM2835_GPIO_REGS_BASE + 0x34 + 0x4*_bank)
+#define BCM2835_GPIO_GPEDS(_bank)    (BCM2835_GPIO_REGS_BASE + 0x40 + 0x4*_bank)
+#define BCM2835_GPIO_GPREN(_bank)    (BCM2835_GPIO_REGS_BASE + 0x4C + 0x4*_bank)
+#define BCM2835_GPIO_GPFEN(_bank)    (BCM2835_GPIO_REGS_BASE + 0x58 + 0x4*_bank)
+#define BCM2835_GPIO_GPHEN(_bank)    (BCM2835_GPIO_REGS_BASE + 0x64 + 0x4*_bank)
+#define BCM2835_GPIO_GPLEN(_bank)    (BCM2835_GPIO_REGS_BASE + 0x70 + 0x4*_bank)
+#define BCM2835_GPIO_GPAREN(_bank)   (BCM2835_GPIO_REGS_BASE + 0x7C + 0x4*_bank)
+#define BCM2835_GPIO_GPAFEN(_bank)   (BCM2835_GPIO_REGS_BASE + 0x88 + 0x4*_bank)
+#define BCM2835_GPIO_GPPUDCLK(_bank) (BCM2835_GPIO_REGS_BASE + 0x98 + 0x4*_bank)
+#define BCM2835_GPIO_GPPUD           (BCM2835_GPIO_REGS_BASE + 0x94)
```

And now, with these changes, making changes to the existing GPIO code got easier too. Moreover, the code consequently becomes more easily readable too.
All of the function definitions just changed in a similar manner to this:

```
 rtems_status_code rtems_gpio_bsp_set(uint32_t bank, uint32_t pin)
 {
- BCM2835_REG(BCM2835_GPIO_GPSET0) = (1 << pin);
+ BCM2835_REG(BCM2835_GPIO_GPSET(bank)) = (1<<pin);
 
  return RTEMS_SUCCESSFUL;
 }
```
Thus, The patch for GPIO was ready, and I used the previously written tests for the same, and Great! It worked!
