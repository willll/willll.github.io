---
layout: default
title: "SEGA SATURN Modem PART 1"
permalink: /ssmodem_part1
---

# SEGA SATURN Modem PART 1

## Overview

This guide explains how to connect the Sega Saturn to the internet using a Conexant RD02-D400 RJ11 to USB adapter.  
To do so, the Conexant modem must be modified by adding a low-cost step-up converter that uses the USB 5V and injects 9V into the phone line.

## Modify RJ11 to USB Adapter

### Goal

> 3 to 9 volt range when a telephone on the line goes off-hook. An off-hook telephone typically draws about 20 milliamps of DC current to operate, at a DC resistance around 180 ohms. The remaining voltage drop occurs over the copper wire path and over the telephone company circuits where there is usually 200 to 400 ohms of series resistance to protect from short circuits and decouple the audio circuits.

**Source**: [jkaudio.com](https://www.jkaudio.com/article_03.htm)

### Bill of Materials (BOM)

| BOM Level | Part Number | Part Name                         | Part Description                      | Quantity |
|:----------|:------------|:----------------------------------|:--------------------------------------|:--------:|
| 1         | 1           | Conexant RD02-D400                | Conexant RD02-D400 Modem              |    1     |
| 1         | 2           | 0.47µF 50V Capacitor              | 0.47µF 50V Electrolytic Capacitor     |    1     |
| 1         | 3           | 330–380 Ohm ±1% Resistor          | 330–380 Ohm ±1% Tolerance Resistor    |    1     |
| 1         | 4           | DC-DC 6W 5V to 9V Converter       | Step-up Converter 2.5–5V to 5V/8V/12V |    1     |
| 1         | 5           | JST 2-pin Wire Connector (Male)   | Male JST 2-pin Wire Connector         |    2     |
| 1         | 6           | JST 2-pin Wire Connector (Female) | Female JST 2-pin Wire Connector       |    2     |

<p style="text-align:center"><i>BOM</i></p>

![BOM](./assets/img/ssmodem_part1/PXL_20250426_214222355.MP.png)

## Conexant Modification Steps

### Teardown

Remove the two screws located on each side of the USB cable. Then, using a spudger, separate the upper and lower casing.

![Conexant teardown](./assets/img/ssmodem_part1/PXL_20250426_214605849.MP.png)
<p style="text-align:center"><i>Conexant teardown</i></p>

### Cut RJ11 Pin 3 Trace

Locate and cut the PCB trace connected to pin 3 of the RJ11 port.

![Trace to Cut](./assets/img/ssmodem_part1/PXL_20240119_131417114.MP.png)
<p style="text-align:center"><i>Trace to Cut</i></p>

Use a knife to carefully cut the trace. It should look like this once cut:

![Trace Cut](./assets/img/ssmodem_part1/2020_0101_010321_005.jpg)
<p style="text-align:center"><i>Trace Cut</i></p>

**⚠️ IMPORTANT**: Use a multimeter to verify that there is **no continuity** across the cut trace before continuing.

**Reference**: [GeeksforGeeks – RJ11 Color Code](https://www.geeksforgeeks.org/rj11-color-code/)

### Solder JST Connectors

![JST Wires](./assets/img/ssmodem_part1/PXL_20250426_221529153.MP.jpg)
<p style="text-align:center"><i>JST Wires</i></p>

First, solder one JST connector to the USB 5V power input:

![USB Power](./assets/img/ssmodem_part1/2020_0102_005120_005.jpg)
<p style="text-align:center"><i>USB Power</i></p>

Then solder another connector near the previously cut trace to inject the 9V:

![Power Line Injector Entry Points](./assets/img/ssmodem_part1/2020_0101_013049_005.jpg)
<p style="text-align:center"><i>Power Line Injector Entry Points</i></p>

### Curing

To avoid short circuits, coat exposed traces with acrylic or silicone.

![Coating example with silicone under UV light](./assets/img/ssmodem_part1/2020_0102_020225_005.jpg)
<p style="text-align:center"><i>Coating example with silicone under UV light</i></p>

## Modify the Step-Up Converter

### Configure the Step-Up Converter

Adjust the potentiometer on the step-up converter to output **9V**.  
Use a multimeter to confirm the correct output voltage before connecting to the modem.

![Board Configurations](./assets/img/ssmodem_part1/2020_0101_013807_005.jpg)
<p style="text-align:center"><i>Step-Up Converter Configuration</i></p>

After adjusting, confirm:

![Board Configured](./assets/img/ssmodem_part1/2020_0101_000424_005.jpg)
<p style="text-align:center"><i>Step-Up Converter Set to 9V</i></p>

### Add JST Connector to Step-Up Converter

Solder a JST connector to the input of the step-up converter.

![Board Configured](./assets/img/ssmodem_part1/2020_0102_033201_004.jpg)
<p style="text-align:center"><i>Step-Up Converter input wiring</i></p>

![Wiring diagram](./assets/img/ssmodem_part1/1_diagram.png)
<p style="text-align:center"><i>Wiring diagram</i></p>

Bend the capacitor and the resistor over the board. Then, twist the capacitor’s negative leg together with the free leg of the resistor.  
**⚠️ IMPORTANT**: Pay close attention to the capacitor’s polarity!

![Capacitor and resistor assembly](./assets/img/ssmodem_part1/2020_0102_043141_004.jpg)
<p style="text-align:center"><i>Capacitor and resistor assembly</i></p>

Use heat shrink tubing to cover the twisted legs. Then, solder the JST cable to both the Step-Up's positive pad and the twisted legs:

![Board Configured](./assets/img/ssmodem_part1/2020_0118_222140_007.jpg)
<p style="text-align:center"><i>Step-Up JST cable assembly</i></p>

As an extra precaution, I chose to solder both negative pads together using the resistor’s leg—though I’m not certain this is strictly necessary.

![Step-Up grounding](./assets/img/ssmodem_part1/2020_0118_222437_008.jpg)
<p style="text-align:center"><i>Step-Up grounding</i></p>

### Curing

Again, applying a protective coating is a good idea. I used MG Chemicals 422C, which fluoresces under UV-A light:

![Step-Up board coating](./assets/img/ssmodem_part1/PXL_20250628_174654179.jpg)
<p style="text-align:center"><i>Step-Up board coating</i></p>

## Validation

At this point, all parts are ready to be assembled:

![Parts](./assets/img/ssmodem_part1/PXL_20250628_190647493.jpg)
<p style="text-align:center"><i>Parts</i></p>

Connect the Step-Up converter to the modem using the JST connectors. Once connected, it's time to test.

A good first test is to inject 5V into the USB port and confirm that the Step-Up board powers on. Then, use a multimeter to read the current draw. During my test, I measured 4.85V at 0.07A.

![Validation](./assets/img/ssmodem_part1/PXL_20250628_191111627.jpg)
<p style="text-align:center"><i>Validation</i></p>

Next, connect the modem to a Raspberry Pi and confirm that all three LEDs are lit and the device is recognized correctly.

```text
usb 1-1.4: new full-speed USB device number 3 using xhci_hcd
usb 1-1.4: New USB device found, idVendor=0572, idProduct=1324, bcdDevice= 1.00
usb 1-1.4: New USB device strings: Mfr=1, Product=2, SerialNumber=3
usb 1-1.4: Product: USB Modem
usb 1-1.4: Manufacturer: Conexant
usb 1-1.4: SerialNumber: 24680246
cdc_acm 1-1.4:1.0: ttyACM0: USB ACM device
usbcore: registered new interface driver cdc_acm
cdc_acm: USB Abstract Control Model driver for USB modems and ISDN adapters
```

## Final Assembly

![Folding all within the case](./assets/img/ssmodem_part1/PXL_20250628_194322102.jpg)
<p style="text-align:center"><i>Folding everything within the case</i></p>

## Result

![Result](./assets/img/ssmodem_part1/PXL_20250628_194648052.MP.jpg)
<p style="text-align:center"><i>Result</i></p>

## Related Links

- [Build a Line Voltage Inducer – SegaSaturnShiro](https://www.segasaturnshiro.com/guide-build-a-line-voltage-inducer/)
- [Saturn Community Projects – SegaSaturnShiro](https://www.segasaturnshiro.com/saturn-community-projects/online-play/)
- [Dreamcast-Talk Forum Discussion](https://www.dreamcast-talk.com/forum/viewtopic.php?t=12731)
- [Browsing the Web with Sega Saturn (2022 & Beyond) – Medium](https://jackrafter.medium.com/tutorial-on-browsing-the-web-with-sega-saturn-and-planetweb-browser-in-2022-and-beyond-79dbab82b198)
- [Dreamcast-Talk: Netlink + DreamPi Discussion](https://dreamcast-talk.com/forum/viewtopic.php?t=8453)