---
layout: default
title: "SEGA SATURN Modem PART 1"
permalink: /ssmodem
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

![BOM](./assets/img/ssmodem/PXL_20250426_214222355.MP.png)

## Conexant Modification Steps

### Teardown

Remove the two screws located on each side of the USB cable. Then, using a spudger, separate the upper and lower casing.

![Conexant teardown](./assets/img/ssmodem/PXL_20250426_214605849.MP.png)
<p style="text-align:center"><i>Conexant teardown</i></p>

### Cut RJ11 Pin 3 Trace

Locate and cut the PCB trace connected to pin 3 of the RJ11 port.

![Trace to Cut](./assets/img/ssmodem/PXL_20240119_131417114.MP.png)
<p style="text-align:center"><i>Trace to Cut</i></p>

Use a knife to carefully cut the trace. It should look like this once cut:

![Trace Cut](./assets/img/ssmodem/2020_0101_010321_005.jpg)
<p style="text-align:center"><i>Trace Cut</i></p>

**⚠️ IMPORTANT**: Use a multimeter to verify that there is **no continuity** across the cut trace before continuing.

**Reference**: [GeeksforGeeks – RJ11 Color Code](https://www.geeksforgeeks.org/rj11-color-code/)

### Solder JST Connectors

![JST Wires](./assets/img/ssmodem/PXL_20250426_221529153.MP.jpg)
<p style="text-align:center"><i>JST Wires</i></p>

First, solder one JST connector to the USB 5V power input:

![USB Power](./assets/img/ssmodem/2020_0102_005120_005.jpg)
<p style="text-align:center"><i>USB Power</i></p>

Then solder another connector near the previously cut trace to inject the 9V:

![Power Line Injector Entry Points](./assets/img/ssmodem/2020_0101_013049_005.jpg)
<p style="text-align:center"><i>Power Line Injector Entry Points</i></p>

### Curing

To avoid short circuits, coat exposed traces with acrylic or silicone.

![Coating example with silicone under UV light](./assets/img/ssmodem/2020_0102_020225_005.jpg)
<p style="text-align:center"><i>Coating example with silicone under UV light</i></p>

## Modify the Step-Up Converter

### Configure the Step-Up Converter

Adjust the potentiometer on the step-up converter to output **9V**.  
Use a multimeter to confirm the correct output voltage before connecting to the modem.

![Board Configurations](./assets/img/ssmodem/2020_0101_013807_005.jpg)
<p style="text-align:center"><i>Step-Up Converter Configuration</i></p>

After adjusting, confirm:

![Board Configured](./assets/img/ssmodem/2020_0101_000424_005.jpg)
<p style="text-align:center"><i>Step-Up Converter Set to 9V</i></p>

### Add JST Connector to Step-Up Converter

Solder a JST connector to the input of the step-up converter.

![Board Configured](./assets/img/ssmodem/2020_0102_033201_004.jpg)
<p style="text-align:center"><i>Step-Up Converter input wiring</i></p>

![Wiring diagram](./assets/img/ssmodem/1_diagram.png)
<p style="text-align:center"><i>Wiring diagram</i></p>

![Board Configured](./assets/img/ssmodem/2020_0102_043141_004.jpg)
<p style="text-align:center"><i>XXXX</i></p>


## Related Links

- [Build a Line Voltage Inducer – SegaSaturnShiro](https://www.segasaturnshiro.com/guide-build-a-line-voltage-inducer/)
- [Saturn Community Projects – SegaSaturnShiro](https://www.segasaturnshiro.com/saturn-community-projects/online-play/)
- [Dreamcast-Talk Forum Discussion](https://www.dreamcast-talk.com/forum/viewtopic.php?t=12731)
- [Browsing the Web with Sega Saturn (2022 & Beyond) – Medium](https://jackrafter.medium.com/tutorial-on-browsing-the-web-with-sega-saturn-and-planetweb-browser-in-2022-and-beyond-79dbab82b198)
- [Dreamcast-Talk: Netlink + DreamPi Discussion](https://dreamcast-talk.com/forum/viewtopic.php?t=8453)

## Next Steps (To Be Done)

- Document the connection between the step-up converter and modem.
- Add a wiring diagram for clarity.
- Integrate testing and troubleshooting guide.
- Link to [Netlink GitHub Repository](https://github.com/eaudunord/Netlink/)
