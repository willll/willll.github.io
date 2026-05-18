---
layout: default
title: "Develop on Sega Saturn Part 3"
permalink: /develop_on_sega_saturn_part3
---

# Develop on Sega Saturn Part 3

## Table of Contents

- [Overview](#overview)
- [How USB Gamer's Cartridge Logging Works](#how-usb-gamers-cartridge-logging-works)
- [Install and Use ftx](#install-and-use-ftx)
- [Retrieve Logs from Saturn](#retrieve-logs-from-saturn)
- [Use with saturn_mandelbrot](#use-with-saturn_mandelbrot)
- [Troubleshooting](#troubleshooting)

## Overview

This part focuses on retrieving debug logs from a real Sega Saturn using a USB Gamer's Cartridge.

In this workflow, logs are read on the host with `ftx`:

- [willll/ftx](https://github.com/willll/ftx)

Note: the setup steps in this page are Linux-focused.

## How USB Gamer's Cartridge Logging Works

1. Saturn code writes trace bytes to the debug output address.
2. USB Gamer's Cartridge forwards these bytes over USB.
3. `ftx` reads the incoming debug stream and prints it on your host terminal.

This is useful for hardware validation when emulator behavior differs from real console behavior.

## Install and Use ftx

Prerequisites (Linux):

- C++ compiler (`g++`)
- `cmake`
- Boost development packages
- `libftdi1` development package

Example install command (Debian/Ubuntu):

```bash
sudo apt update
sudo apt install -y build-essential cmake libboost-program-options-dev libboost-filesystem-dev libftdi1-dev
```

Build `ftx` from source:

```bash
git clone https://github.com/willll/ftx.git
cd ftx
mkdir -p build
cd build
cmake ..
make
```

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

Expected success example:

```text
[BOOT] mandelbrot start
[INIT] slInitSystem done
[LOOP] frame done
```

## Use with saturn_mandelbrot

Reference project:

- [saturn_mandelbrot](https://github.com/willll/saturn_mandelbrot)

Important: upstream `saturn_mandelbrot` does not print trace messages by default. Add `debug_print(...)` calls in the code (as shown in Part 2), then rebuild before testing on hardware.

Typical trace flow:

1. Build `saturn_mandelbrot` (from Part 2 workflow).
2. Run the program on real hardware.
3. Start `ftx -c` on host.
4. Confirm expected trace prefixes such as `[BOOT]`, `[INIT]`, and `[LOOP]`.

## Troubleshooting

- No output in `ftx`: verify cartridge cable/connection and confirm program is emitting trace messages.
- No output in `ftx`: verify cartridge cable/connection and confirm you are running a build that includes `debug_print(...)` trace calls.
- Garbled output: check cartridge USB connection stability and retry with explicit VID/PID.
- Intermittent logs: reduce trace volume in tight loops and keep messages short.
- Permission denied / device access error on Linux: check user USB permissions (udev/group), or test once with elevated privileges to confirm it is a permissions issue.

Quick Linux device check:

```bash
lsusb | grep -i "0403:6001"
```
