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
  - [Video Demonstration](#video-demonstration)
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

If you prefer running `ftx` directly inside `saturn-docker` without installing host compilation libraries, you can pass host USB devices into the container using device cgroup rules, `/dev` volume mounts, and proper user permissions:

<details markdown="1">
<summary style="cursor: pointer;"><strong>👉 Click here to expand Docker USB Passthrough instructions</strong></summary><br/>

Launch `saturn-docker` with full USB device passthrough:

```bash
docker run -it --rm \
  -e "SRL_INSTALL_ROOT=/saturn/SaturnRingLib" \
  -p 2222:22 \
  --add-host=host.docker.internal:host-gateway \
  --device-cgroup-rule='c 188:0 rmw' \
  --device-cgroup-rule='c 189:* rmw' \
  -v /dev/bus/usb:/dev/bus/usb \
  -v /var/run/dbus:/var/run/dbus \
  -v /dev:/dev \
  -v /home/will/tmp/:/saturn \
  --user 1000:1000 \
  saturn-docker:latest /bin/bash
```

What these flags do:
- `--device-cgroup-rule='c 188:0 rmw'` & `'c 189:* rmw'`: Grants read/write/mknod permissions for USB serial (`ttyUSB0`, major 188) and FTDI USB bus character devices (major 189).
- `-v /dev/bus/usb:/dev/bus/usb` & `-v /dev:/dev`: Mounts host USB device bus trees and device filesystem nodes directly into the container environment.
- `-v /var/run/dbus:/var/run/dbus`: Mounts host D-Bus system socket for daemon communication.
- `-e "SRL_INSTALL_ROOT=/saturn/SaturnRingLib"`: Defines the path to SaturnRingLib inside the container.
- `--add-host=host.docker.internal:host-gateway`: Resolves host IP address inside container for host-container network communication.
- `-p 2222:22`: Forwards host port 2222 to container SSH port 22.
- `--user 1000:1000`: Runs container processes with host user/group IDs to preserve USB device node permissions.
- `-v /home/will/tmp/:/saturn`: Mounts your local Saturn workspace directory into `/saturn`.

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
    TESTING : Starting
    TESTING : Initializing Renderer
    TESTING : Initializing 352x240 Canvas
    TESTING : Starting Render Zone 0: Full Mandelbrot View
    TESTING : Initialized Renderer
    TESTING : render :: Complete for Zone 0 (Full Mandelbrot View)! 5s timer started...
    TESTING : 5s Elapsed! Cleared screen & switching to Zoom Zone 1 (Seahorse Valley)
    TESTING : render :: Complete for Zone 1 (Seahorse Valley)! 5s timer started...
    TESTING : 5s Elapsed! Cleared screen & switching to Zoom Zone 2 (Deep Seahorse Spiral)
    TESTING : render :: Complete for Zone 2 (Deep Seahorse Spiral)! 5s timer started...
    TESTING : 5s Elapsed! Cleared screen & switching to Zoom Zone 3 (Mini Mandelbrot Bulb)
    ```

### Video Demonstration

Here is a real-time demonstration (*SRL mandelbrot demo with traces*) showcasing `SRL-Mandelbrot` running on physical Sega Saturn hardware while `ftx` captures live trace output (`TESTING : ...`) streamed over the USB Gamer's Cartridge FTDI interface:

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; margin-top: 1rem; margin-bottom: 1.5rem;">
  <iframe src="https://www.youtube.com/embed/Fol1nZI7gI4" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

The video demonstrates:
- **Binary Upload & Execution**: Uploading the compiled `Mandelbrot.bin` payload to Saturn RAM and launching `ftx` debug console mode (`./ftx -c`).
- **Hardware Rendering**: The SH-2 CPUs rendering the 352x240 High Color Mandelbrot set and cycling through the 4 fractal zoom zones.
- **Real-time Trace Streaming**: Synchronous hardware log messages (`TESTING : render :: Complete for Zone ...` and `TESTING : 5s Elapsed! ...`) streaming over the FTDI USB FIFO interface in CS0 space (`0x22100001` / `0x22200001`) directly to the host terminal console.

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

You now have real-time hardware trace logging integrated into your Sega Saturn workflow.

Retrieving debug traces directly over the [USB Gamer's Cartridge](https://ppcenter.webou.net/satcart/) with `ftx` gives you host-side visibility when testing on real console hardware, bridging the gap between emulator debugging (with Kronos or Mednafen) and physical hardware validation.

Future parts in this series will cover deeper Saturn subsystems, hardware profiling, and advanced hardware debugging workflows.
