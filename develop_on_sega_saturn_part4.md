---
layout: default
title: "Develop on Sega Saturn Part 4"
permalink: /develop_on_sega_saturn_part4
---

# Develop on Sega Saturn Part 4

## Table of Contents

- [Overview](#overview)
- [How USB Gamer's Cartridge Logging Works](#how-usb-gamers-cartridge-logging-works)
- [Install and Use ftx](#install-and-use-ftx)
  - [Linux (Debian/Ubuntu)](#linux-debianubuntu)
  - [macOS (Apple Silicon & Intel)](#macos-apple-silicon--intel)
  - [Windows](#windows)
  - [Docker USB Passthrough](#docker-usb-passthrough)
- [Retrieve Logs from Saturn](#retrieve-logs-from-saturn)
- [Use with SRL-Mandelbrot](#use-with-srl-mandelbrot)
- [Troubleshooting](#troubleshooting)
- [Next Steps](#next-steps)

## Overview

After setting up the toolchain ([Part 1](./develop_on_sega_saturn_part1)), configuring VS Code ([Part 2](./develop_on_sega_saturn_part2)), and testing debug traces in emulators ([Part 3](./develop_on_sega_saturn_part3)), this final part focuses on retrieving debug logs from a real Sega Saturn using a USB Gamer's Cartridge.

In this workflow, logs are read on the host with `ftx`:

- [willll/ftx](https://github.com/willll/ftx)

## How USB Gamer's Cartridge Logging Works

1. Saturn code writes trace bytes to the debug output address.
2. USB Gamer's Cartridge forwards these bytes over USB.
3. `ftx` reads the incoming debug stream and prints it on your host terminal.

![Hardware Trace Sequence Diagram]({{ '/assets/img/develop_on_sega_saturn_part4/hardware-trace-sequence.svg' | relative_url }})

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

> **Important Driver Note for Windows:** Windows installs the FTDI Virtual COM Port driver by default. You must replace it with the generic **WinUSB** driver using [Zadig](https://zadig.akeo.ie/):
> 1. Launch Zadig, select **Options -> List All Devices**.
> 2. Select **FT245R USB FIFO** (VID `0403`, PID `6001`).
> 3. Select **WinUSB** as target driver and click **Replace Driver**.

</details>

### Docker USB Passthrough

If you prefer running `ftx` directly inside `saturn-docker` without installing host compilation libraries, pass host USB devices into the container using device cgroup rules and volume mounts:

<details markdown="1">
<summary style="cursor: pointer;"><strong>👉 Click here to expand Docker USB Passthrough instructions</strong></summary><br/>

Launch `saturn-docker` with USB device access:

```bash
docker run -it --rm \
  -v $(pwd):/saturn \
  -v /dev/bus/usb:/dev/bus/usb \
  -v /dev:/dev \
  --device-cgroup-rule='c 188:* rmw' \
  --device-cgroup-rule='c 189:* rmw' \
  --user $(id -u):$(id -g) \
  saturn-docker /bin/bash
```

What these flags do:
- `-v /dev/bus/usb:/dev/bus/usb` & `-v /dev:/dev`: Mounts host USB device nodes into the container environment.
- `--device-cgroup-rule='c 188:* rmw'` & `'c 189:* rmw'`: Grants read/write/mknod permissions for USB serial and FTDI character devices (major `188` for `ttyUSB`, major `189` for USB bus nodes).
- `--user $(id -u):$(id -g)`: Runs container processes as your host user account to preserve USB file permissions.

Inside the container, `ftx` can talk directly to your USB Gamer's Cartridge.

</details>

Start debug console mode (`-c` for read-only log view, or `-t` for interactive bidirectional terminal):

```bash
./ftx -c
```

If needed, specify USB VID/PID explicitly using `--vid` and `--pid`:

```bash
./ftx --vid 0x0403 --pid 0x6001 -c
```

## Retrieve Logs from Saturn

1. Connect USB Gamer's Cartridge to the Sega Saturn and host computer.
2. Start `ftx` debug console mode on host (`./ftx -c`).
3. Boot your Saturn program.
4. Watch trace lines appear in real time in the terminal.

## Use with SRL-Mandelbrot

Reference projects:

- [SRL-Mandelbrot](https://github.com/willll/SRL-Mandelbrot) – Mandelbrot rendering demo built with SaturnRingLib.
- [SaturnRingLib (SRL)](https://github.com/ReyeMe/SaturnRingLib) – Easy-to-use C++ wrapper for SGL (Sega Graphic Library).

### What Is SaturnRingLib (SRL)?

[SaturnRingLib (SRL)](https://github.com/ReyeMe/SaturnRingLib) is an object-oriented C++ framework designed to simplify Sega Saturn development by providing high-level abstractions on top of SGL:

- **Graphics & Bitmaps:** Simplifies VDP1 sprite/texture creation and palette management (`SRL::Bitmap::IBitmap`).
- **Slave SH-2 Offloading:** Easily offloads heavy computational loops (such as Mandelbrot pixel calculation) to the secondary SH-2 CPU via `SRL::Slave::ExecuteOnSlave()`.
- **Integrated Memory & Debugging:** Provides structured memory allocation helpers alongside built-in debug logging channels.

### Building SRL-Mandelbrot & Enabling Debug Traces

To build `SRL-Mandelbrot` and capture trace output over USB:

1. Clone the repository:

   ```bash
   git clone https://github.com/willll/SRL-Mandelbrot.git
   cd SRL-Mandelbrot
   ```

2. Build with Debug flags:

   - **Linux (`saturn-docker` / `make`):**
     ```bash
     docker run --rm -i -v $(pwd):/saturn saturn-docker /bin/bash -c "cd /saturn && make DEBUG=1"
     ```
   - **Windows / Batch Helper:**
     ```cmd
     compile.bat debug
     ```

   > **Build Flags Note:** Compiling with `DEBUG=1` (or `compile.bat debug`) defines the `-DDEBUG` preprocessor flag, enabling trace output logging and compiling the final binaries into `BuildDrop/`.

3. Boot on hardware via USB Gamer's Cartridge:

   Upload the compiled binary (`BuildDrop/mandelbrot.bin`) to RAM (`0x06004000`) and launch `ftx` debug console mode (`-c`):

   ```bash
   ./ftx -x BuildDrop/mandelbrot.bin 0x06004000
   ./ftx -c
   ```

4. Confirm expected trace lines appear in your host terminal:

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
