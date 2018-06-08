---
layout: post
published: true
author: Alexandre Dumont
last_modified_at: '2018-06-08 23:29 +0200'
title: Input an 8-bit value using a DIP switch
tags: ''
image: 'uploads/dip-switch.png'
---
In this article, I'll show how I connected a DIP switch to input an 8-bit value to the Icezum Alhambra FPGA board (Ice40HX1k)

### Material I used

- a breadboard and some wires
- [8 poles DIP Switch](https://www.mouser.es/ProductDetail/Omron-Electronics/A6E-8104-N?qs=sGAEpiMZZMv%2f%252b2JhlA6ysK8z9VoUe3M4NRXxMPQBGgI%3d)
- [9 pins 3.3KOhm bussed resistor array](https://www.mouser.es/ProductDetail/652-4609X-1LF-3.3K)

### Schematics

The DIP switch is an array of 8 SPST switches. We use the resistor array to connect the dip switch in a Pull-Up configuration.

- When the switch is open, current will flow from Vcc to the PINs.
- When the switch is closed, the pin will be brought down to ground.

![](https://github.com/adumont/Dip-Switch_PullUp/raw/master/images/breadboard.jpg)

![](https://github.com/adumont/Dip-Switch_PullUp/raw/master/images/pins.jpg)

### Sample Circuit in IceStudio

In the circuit, the register enable is connected to "1" (always enabled). It can also be connected to the dbouncer, so that it will load only when button is pressed.

![](https://github.com/adumont/Dip-Switch_PullUp/raw/master/images/circuit-icestudio.png)
