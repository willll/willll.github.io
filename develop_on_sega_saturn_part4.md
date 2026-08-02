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
  - [Pre-built Binary Release](#pre-built-binary-release)
  - [Linux (Debian/Ubuntu)](#linux-debianubuntu)
  - [macOS (Apple Silicon & Intel)](#macos-apple-silicon--intel)
  - [Windows](#windows)
  - [Docker USB Passthrough](#docker-usb-passthrough)
- [Retrieve Logs from Saturn](#retrieve-logs-from-saturn)
- [Use with SRL-Mandelbrot](#use-with-srl-mandelbrot)
- [Troubleshooting](#troubleshooting)
- [Next Steps](#next-steps)

## Overview

After setting up the toolchain ([Part 1](./develop_on_sega_saturn_part1)), configuring VS Code ([Part 2](./develop_on_sega_saturn_part2)), and testing debug traces in emulators ([Part 3](./develop_on_sega_saturn_part3)), this final part focuses on retrieving debug logs from a real Sega Saturn using a [USB Gamer's Cartridge](https://ppcenter.webou.net/satcart/).

In this workflow, logs are read on the host with `ftx`:

- [willll/ftx](https://github.com/willll/ftx)

## How USB Gamer's Cartridge Logging Works

1. Saturn code writes trace bytes to the Gamer's Cartridge USB FIFO data register (`0x22100001` in CS0 space) while checking status flags (`0x22200001`).
2. [USB Gamer's Cartridge](https://ppcenter.webou.net/satcart/) forwards these bytes over USB via its FTDI interface.
3. `ftx` reads the incoming debug stream and prints it on your host terminal.

> **Hardware vs. Emulator Memory Mapping:** Real Gamer's Cartridge hardware maps its USB FIFO registers in CS0 space (`0x22100001` data, `0x22200001` flags). This differs from emulator-based debug cart logging (such as Mednafen or Kronos), which listens on CS1 space (`0x24001000`).

![Hardware Trace Sequence Diagram]({{ '/assets/img/develop_on_sega_saturn_part4/hardware-trace-sequence.svg' | relative_url }})

This is useful for hardware validation when emulator behavior differs from real console behavior.

## Install and Use ftx

You can either download pre-built binaries directly from [willll/ftx Releases](https://github.com/willll/ftx/releases) or build `ftx` from source.

### Pre-built Binary Release

Download the latest executable for your OS from the [GitHub Releases](https://github.com/willll/ftx/releases) page. On Linux and macOS, grant execution permissions after downloading:

```bash
chmod +x ftx
```

---

### Linux (Debian/Ubuntu)

<details markdown="1">
<summary style="cursor: pointer;"><strong>👉 Click here to expand Linux build instructions</strong></summary><br/>

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
<summary style="cursor: pointer;"><strong>👉 Click here to expand macOS build instructions</strong></summary><br/>

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
<summary style="cursor: pointer;"><strong>👉 Click here to expand Windows build instructions</strong></summary><br/>

Alternatively, download `ftx.exe` from [Releases](https://github.com/willll/ftx/releases), or build from source using MSYS2/MinGW or WSL:

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

Inside the container, `ftx` can talk directly to your [USB Gamer's Cartridge](https://ppcenter.webou.net/satcart/).

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

1. Connect [USB Gamer's Cartridge](https://ppcenter.webou.net/satcart/) to the Sega Saturn and host computer.
2. Start `ftx` debug console mode on host (`./ftx -c`).
3. Boot your Saturn program.
4. Watch trace lines appear in real time in the terminal.

## Use with SRL-Mandelbrot

Reference projects:

- [SRL-Mandelbrot](https://github.com/willll/SRL-Mandelbrot) – Mandelbrot rendering demo built with SaturnRingLib.
- [SaturnRingLib (SRL)](https://github.com/ReyeMe/SaturnRingLib) – Easy-to-use C++ wrapper for SGL (Sega Graphic Library).

### What Is SaturnRingLib (SRL)?

[SaturnRingLib (SRL)](https://github.com/ReyeMe/SaturnRingLib) is an object-oriented C++ framework designed to simplify Sega Saturn development by providing high-level abstractions on top of SGL.

### Building SRL-Mandelbrot & Enabling Debug Traces

To build `SRL-Mandelbrot` and capture trace output over USB:

1. Clone the repository:

   ```bash
   git clone https://github.com/willll/SRL-Mandelbrot.git
   cd SRL-Mandelbrot
   ```

2. Configure `makefile` and build with hardware trace flags:

   In `SRL-Mandelbrot/makefile`, trace output behavior is controlled by two key variables:
   - `SRL_LOG_LEVEL = TESTING`: Controls the maximum log level to display.
   - `SRL_LOG_OUTPUT = EMULATOR`: Selects the target log output mechanism (`DEV_CART`, `EMULATOR`, or `NONE`).

   To route log messages directly to the FTDI USB device on real hardware, `SRL_LOG_OUTPUT` must be set to `DEV_CART`. You can pass this flag when running `make`:

   - **Linux (`saturn-docker` / `make`):**
     ```bash
     docker run --rm -i -v $(pwd):/saturn saturn-docker /bin/bash -c "cd /saturn && make SRL_LOG_OUTPUT=DEV_CART"
     ```
     If switching output targets from a previous build, clean prior objects first:
     ```bash
     docker run --rm -i -v $(pwd):/saturn saturn-docker /bin/bash -c "cd /saturn && make clean && make SRL_LOG_OUTPUT=DEV_CART"
     ```
   - **Windows / Batch Helper:**
     ```cmd
     compile.bat release SRL_LOG_OUTPUT=DEV_CART
     ```

   > **Note:** Overwriting `SRL_LOG_OUTPUT=DEV_CART` compiles SaturnRingLib's debug output backend (`SRL::DevCart`) to target the real hardware Gamer's Cartridge USB FIFO registers in CS0 space (`0x22100001` data, `0x22200001` flags) rather than emulator CS1 space (`0x24001000`), allowing `ftx` to capture log streams directly from the FTDI chip.

3. Boot on hardware via [USB Gamer's Cartridge](https://ppcenter.webou.net/satcart/):

   Upload the compiled binary (`BuildDrop/Mandelbrot.bin`) to RAM (`0x06004000`) and launch `ftx` debug console mode (`-c`):

   - **Using `ftx` CLI directly:**
     ```bash
     ./ftx -x BuildDrop/Mandelbrot.bin 0x06004000
     ./ftx -c
     ```
   - **Windows / Batch Helper:**
     Alternatively, run the included `run_on_saturn.bat` script to automate uploading and console logging:
     ```cmd
     run_on_saturn.bat
     ```

4. Confirm expected trace lines appear in your host terminal:

   ```text
   Starting
   Initializing Renderer
   Initializing 352x240 Canvas
   Starting Render Zone 0: Full Mandelbrot View
   Initialized Renderer
   ```

## Troubleshooting

- No output in `ftx`: verify cartridge cable/connection and confirm `SRL_LOG_OUTPUT=DEV_CART` was used when compiling.
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

By utilizing Docker, you've completely abstracted away the pain of setting up the C compiler toolchain. With CMake and Ninja, your builds are fast and reproducible. And by implementing the debug trace output, you can smoothly develop locally with Kronos or Mednafen, and seamlessly take your project to real hardware via the [USB Gamer's Cartridge](https://ppcenter.webou.net/satcart/) when it's time for real validation.

Happy coding!
