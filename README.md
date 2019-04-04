Yocto BSP meta layer for the UP Board
======================================

This README file contains information on building the meta-up-board BSP
layer.  Please see the corresponding sections below for details.

For more information on the UP board see:
http://up-board.org

UP Board is based on the Intel(R) Atom(TM) x5-Z83xx SoC (formerly Cherry
Trail).

It includes:
* 1/2/4GB RAM
* 16/32/64GB eMMC flash
* 6 USB2.0 ports
* 1 USB 3.0 OTG port
* 1 Gigabit Ethernet
* HDMI and DSI/eDP Graphics ports,
* CSI
* RTC
* 40-pin I/O header.

The form-factor for the board is based on the Raspberry Pi 2, and it can be used with many of the add-on HAT boards
designed for the Raspberry Pi 2.

Table of Contents
=================

1. Prerequisites
2. Building the meta-up-board BSP layer
3. Booting the live USB image
4. Connecitvity firmware
5. Device Notes
6. Additional Resources


Prerequisites
================

Supported Yocto versions
------------------------
* Yocto 2.3 (Pyro)
* Yocto 2.5 (Sumo)

Supported hardware versions
---------------------------
* UP Board V0.4 (DVT/production)
* UP Squared
* UP Core


Building the meta-up-board BSP layer
========================================

Download the latest Sumo release and enter the poky directory:
```
git clone -b sumo git://git.yoctoproject.org/poky.git
cd poky
```
Download the latest Intel BSP layer version for Sumo:

```
git clone -b sumo git://git.yoctoproject.org/meta-intel.git
```

Download the latest collection of layers for OE-core universe for Sumo:
```
git clone -b sumo git://git.openembedded.org/meta-openembedded 
```
Download meta-virtualization and openembedded-core for Docker containers:
```
git clone -b sumo git://git.yoctoproject.org/meta-virtualization
```

```
git clone -b sumo git://git.openembedded.org/openembedded-core
```

Download this UP Board BSP layer for Sumo:

NOTE:
-----
Due to an issue with Linux-Intel kernel for Cherry Trail SoCs,
the layer must be separated for each kernel version:

For UP board and UP Core (Cherry Trail SoCs):
---------------------------------------------

```
git clone -b sumo_linux-yocto https://github.com/emutex/meta-up-board
```

Building your Yocto image for each UP machine
=============================================

UP Board:
---------
From the poky directory:

```
TEMPLATECONF=meta-up-board/conf source oe-init-build-env
bitbake upboard-image-sato
```

UP Core Board:
--------------
From the meta-up-board directory:
```
sudo nano conf/machine/up-board.conf
```
Uncomment required file for up-core boards to enable all features:
```
require conf/machine/include/up-core.inc
```

Then, from the poky directory:

```
TEMPLATECONF=meta-up-board/conf source oe-init-build-env
bitbake upboard-image-sato
```

At the end of a successful build, you should have a live image that
you can boot from a USB flash drive (see instructions on how to do
that below, in the section 'Booting the live USB image').


Booting the live USB image
==============================

This BSP creates bootable live images, which can be used to directly
boot Yocto off of a USB flash drive.  Upon completion of a successful
build, described in the previous section, the images are created in
a sub-folder named ./tmp/deploy/images/up-board/

Under Linux, insert a USB flash drive.  Assuming the USB flash drive
takes device /dev/sdf, use dd to copy the live image to it.  For
example:

```
dd if=core-image-sato-up-board.hddimg of=/dev/sdf
sync
eject /dev/sdf
```

This should give you a bootable USB flash device.  Insert the device
into a bootable USB socket on the target, and power on.  At the
initial BIOS splash screen, press F7 to display a menu of boot options
and select the USB device.  This should result in a system booted to
the Sato graphical desktop.

If you want a terminal, use the arrows at the top of the UI to move to
different pages of available applications, one of which is named
'Terminal'.  Clicking that should give you a root terminal.

If you want to ssh into the system, you can use the root terminal to
ifconfig the IP address and use that to ssh in.  The root password is
empty, so to log in type 'root' for the user name and hit 'Enter' at
the Password prompt: and you should be in.

If you find you're getting corrupt images on the USB (it doesn't show
the syslinux boot: prompt, or the boot: prompt contains strange
characters), try doing this first:

```
dd if=/dev/zero of=/dev/sdf bs=1M count=512
```
Connectivity firmware
======================
Ampak connectivity firmware is included to enable WiFi and Bluetooth
for UPCorePlus boards.

Firmware will be included by defaults for building your image. If you
don't want to include it, just edit conf/machine/up-board.conf file
and disable "Ampak-firmware, SystemD and network tools" parameters.

a. WiFi
--------
Scan your available WiFi networks:

```
iwlist wlan0 scan
```
You will see all the WiFi interfaces in your area.

a. Bluetooth
-------------
Check your Bluetooth devices in your area:

```
systemctl restart firmware-ampak-ap6355.service

rfkill unblock bluetooth

hciconfig hci0

hcitool scan
```

Device Notes
==============

a. GPIO pin mapping (40-pin HAT connector)
------------------------------------------

The UP Board features an external 40-pin header for I/O functions
including GPIO, I2C, UART, SPI, PWM and I2S, similar in layout to the
Raspberry Pi 2.  The Intel X5-Z8350 SoC provides the I/O functions for
these pins at 1.8V logic levels.

Additional buffers and mux switches are used between the SoC and the
I/O pin header to convert between the 1.8V SoC I/O and the 3.3V levels
required at the pin header, with sufficient current source/sink
capability for  LV-TTL/LV-CMOS compatibility.  These buffers and mux
switches require run-time configuration based on the pin function or
GPIO direction selected by the user.

A platform gpio/pinctrl driver has been developed specifically for UP
to manage the complexity of the buffer configuration so that
application code can transparently access the I/O functions on the
external pins through standard kernel interfaces.  It instantiates a
gpio and pinctrl device, and effectively acts as a "shim" between
application code and the underlying SoC GPIO driver.  It creates a
set of logical GPIO pins which are numbered according to the same
layout as the Raspberry Pi 2, to ease portability of I/O-related
application code between the Raspberry Pi and UP platforms.

| Physical Pin | Function | Sysfs GPIO | Notes                |
|--------------|----------|------------|----------------------|
| 1            | 3V3 VCC  |            |                      |
| 2            | 5V VCC   |            |                      |
| 3            | I2C SDA1 |  2         | I2C1 (/dev/i2c-1)    |
| 4            | 5V VCC   |            |                      |
| 5            | I2C SCL1 |  3         | I2C1 (/dev/i2c-1)    |
| 6            | GND      |            |                      |
| 7            | GPIO(4)  |  4         |                      |
| 8            | UART TX  | 14         | UART1 (/dev/ttyS1)   |
| 9            | GND      |            |                      |
| 10           | UART RX  | 15         | UART1 (/dev/ttyS1)   |
| 11           | UART RTS | 17         | UART1 (/dev/ttyS1)   |
| 12           | I2S CLK  | 18         | I2S (PCM Audio)      |
| 13           | GPIO(27) | 27         |                      |
| 14           | GND      |            |                      |
| 15           | GPIO(22) | 22         |                      |
| 16           | GPIO(23) | 23         |                      |
| 17           | 3V3 VCC  |            |                      |
| 18           | GPIO(24) | 24         |                      |
| 19           | SPI MOSI | 10         | SPI2 (/dev/spidev2.x)|
| 20           | GND      |            |                      |
| 21           | SPI MISO |  9         | SPI2 (/dev/spidev2.x)|
| 22           | GPIO(25) | 25         |                      |
| 23           | SPI SCL  | 11         | SPI2 (/dev/spidev2.x)|
| 24           | SPI CS0  |  8         | SPI2 (/dev/spidev2.0)|
| 25           | GND      |            |                      |
| 26           | SPI CS1  |  7         | SPI2 (/dev/spidev2.1)|
| 27           | I2C SDA0 |  0         | I2C0 (/dev/i2c-0)    |
| 28           | I2C SCL0 |  1         | I2C0 (/dev/i2c-0)    |
| 29           | GPIO(5)  |  5         |                      |
| 30           | GND      |            |                      |
| 31           | GPIO(6)  |  6         |                      |
| 32           | PWM0     | 12         | PWM Chip 0 Channel 0 |
| 33           | PWM1     | 13         | PWM Chip 1 Channel 0 |
| 34           | GND      |            |                      |
| 35           | I2S FRM  | 19         | I2S (PCM Audio)      |
| 36           | UART CTS | 16         | UART1 (/dev/ttyS1)   |
| 37           | GPIO(26) | 26         |                      |
| 38           | I2S DIN  | 20         | I2S (PCM Audio)      |
| 39           | GND      |            |                      |
| 40           | I2S DOUT | 21         | I2S (PCM Audio)      |
 -------------------------------------------------------------

b. PWM
------
PWM frequency range is from 293 Hz to 6.4 MHz.  8-bit
resolution is supported for duty-cycle adjustments, but this reduces
for frequencies > 97.6kHz

c. I2C
------
2 I2C channels support standard-mode (100kHz) and fast-mode (400kHz).
Bus frequency can be selected in BIOS settings.  Note that, unlike
Raspberry Pi, the I2C controller issues Repeated-START commands for
combined transactions (e.g. a write-then-read transaction) which may
not be supported by some I2C slave devices.  For such devices, it is
advisable to use separate write and read transactions to ensure that
Repeated-STARTs are not issued.

d. SPI
------
Bus frequencies up to 25MHz are supported, in steps which are less
granular at higher speeds.  E.g. Available speeds include:
 25MHz, 12.5MHz, 8.33MHz, 6.25MHz, 5MHz, 4.167MHz, 3.571MHz, 3.125MHz, etc.
Please be aware that speeds in between those steps will be rounded UP
to the next nearest available speed, and capped at 25MHz.

e. UART
-------
A high-speed UART is available, supporting baud rates up to
support baud rates between 300 and 3686400.  Hardware flow-control
signals are available on pins 11/36 (RTS/CTS).

Additional Resources
=======================
In addition to this README, please see the following URLs for details
and additional documentation:

* http://up-board.org
* https://up-community.org

Please submit any patches against this BSP to the maintainer:
Maintainer: Ubilinux Team <ubilinux@emutex.com>
