---
layout: default
title: "Develop on Sega Saturn Part 3"
permalink: /develop_on_sega_saturn_part3
---

# Develop on Sega Saturn Part 3

## Table of Contents

- [Develop on Sega Saturn Part 3](#develop-on-sega-saturn-part-3)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [How USB Gamer's Cartridge Logging Works](#how-usb-gamers-cartridge-logging-works)
  - [Install and Use ftx](#install-and-use-ftx)
  - [Retrieve Logs from Saturn](#retrieve-logs-from-saturn)
  - [Use with saturn\_mandelbrot](#use-with-saturn_mandelbrot)
  - [Troubleshooting](#troubleshooting)
  - [Next Steps](#next-steps)

## Overview

This part focuses on retrieving debug logs from a real Sega Saturn using a USB Gamer's Cartridge.

In this workflow, logs are read on the host with `ftx`:

- [willll/ftx](https://github.com/willll/ftx)

## How USB Gamer's Cartridge Logging Works

1. Saturn code writes trace bytes to the debug output address.
2. USB Gamer's Cartridge forwards these bytes over USB.
3. `ftx` reads the incoming debug stream and prints it on your host terminal.

![Hardware Trace Sequence Diagram]({{ '/assets/img/develop_on_sega_saturn_part3/hardware-trace-sequence.svg' | relative_url }})

This is useful for hardware validation when emulator behavior differs from real console behavior.

## Install and Use ftx

### Linux (Debian/Ubuntu)

<details markdown="1">
<summary style="cursor: pointer;"><strong>👉 Click here to expand Linux instructions</strong></summary><br/>

Install dependencies:

```bash
sudo apt update
sudo apt install -y build-essential cmake libboost-program-options-dev libboost-filesystem-dev libftdi1-dev
```

Build `ftx` from source:

```bash
git clone https://github.com/willll/ftx.git
cd ftx
mkdir -p build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make
```

</details>

### macOS (Apple Silicon & Intel)

<details markdown="1">
<summary style="cursor: pointer;"><strong>👉 Click here to expand macOS instructions</strong></summary><br/>

Install dependencies via Homebrew:

```bash
brew install cmake boost libftdi pkg-config
```

Build `ftx` natively from source:

```bash
git clone https://github.com/willll/ftx.git
cd ftx
mkdir -p build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make
```

</details>

### Windows

<details markdown="1">
<summary style="cursor: pointer;"><strong>👉 Click here to expand Windows instructions</strong></summary><br/>

Install dependencies via MSYS2/MinGW or use WSL. The build steps are identical to Linux/macOS:

```bash
git clone https://github.com/willll/ftx.git
cd ftx
mkdir -p build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make
```

</details>

Start debug console mode from the build folder:

```bash
./ftx -c
```

If needed, specify USB VID/PID explicitly:

```bash
./ftx -v 0x0403 -p 0x6001 -c
```

## Retrieve Logs from Saturn

1. Connect USB Gamer's Cartridge to the Sega Saturn and host computer.
2. Start `ftx` debug console mode on host (`./ftx -c`).
3. Boot your Saturn program.
4. Watch trace lines appear in real time in the terminal.



## Use with saturn_mandelbrot

Reference project:

- [saturn_mandelbrot](https://github.com/willll/saturn_mandelbrot)

As of tag [v1.0](https://github.com/willll/saturn_mandelbrot/releases/tag/v1.0), `saturn_mandelbrot` ships with built-in trace output. No code changes are needed to see logs.

Typical trace flow:

1. Build `saturn_mandelbrot` v1.0 (from Part 2 workflow).
2. Turn on the Sega Saturn with the USB Gamer's Cartridge inserted.
3. Use `ftx` to upload the raw binary to RAM (`0x06004000`) and execute it. Then, start the debug console (`-c`):

   ```bash
   ./ftx -x path/to/build/mandelbrot.bin 0x06004000
   ./ftx -c
   ```

4. Confirm expected lines appear in your terminal:

   ```text
   trace: main start
   trace: video on, entering render loop
   ```

## Troubleshooting

- No output in `ftx`: verify cartridge cable/connection and confirm you are running the v1.0 build (or a build with `debug_print(...)` calls).
- Garbled output: check cartridge USB connection stability and retry with explicit VID/PID.
- Intermittent logs: reduce trace volume in tight loops and keep messages short.
- Permission denied / device access error on Linux: this is the most common issue when working with FTDI USB devices on Linux. You need to configure a `udev` rule to grant your user access. To fix this permanently, run these commands:

```bash
echo 'SUBSYSTEM=="usb", ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6001", MODE="0666"' | sudo tee /etc/udev/rules.d/99-ftdi.rules
sudo udevadm control --reload-rules
sudo udevadm trigger
```

Quick Linux device check:

```bash
lsusb | grep -i "0403:6001"
```

## Next Steps

Congratulations! You now have a complete, cross-platform Sega Saturn development environment. 

By utilizing Docker, you've completely abstracted away the pain of setting up the C compiler toolchain. With CMake and Ninja, your builds are fast and reproducible. And by implementing the debug trace output, you can smoothly develop locally with Kronos or Mednafen, and seamlessly take your project to real hardware via the USB Gamer's Cartridge when it's time for real validation.

Happy coding!
