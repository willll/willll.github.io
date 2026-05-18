---
layout: default
title: "Develop on Sega Saturn Part 2"
permalink: /develop_on_sega_saturn_part2
---

# Develop on Sega Saturn Part 2

## Table of Contents

- [Develop on Sega Saturn Part 2](#develop-on-sega-saturn-part-2)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [What Is a Debug Trace?](#what-is-a-debug-trace)
  - [How to Retrieve Traces](#how-to-retrieve-traces)
    - [Kronos](#kronos)
    - [Mednafen](#mednafen)
  - [Example Project: saturn\_mandelbrot](#example-project-saturn_mandelbrot)
    - [Where Debug Output Is Implemented](#where-debug-output-is-implemented)
    - [Add Trace Calls in the Program](#add-trace-calls-in-the-program)
    - [Build with saturn-docker](#build-with-saturn-docker)
    - [Run It](#run-it)
    - [Expected Success Output](#expected-success-output)
    - [Retrieve Traces with Emulators](#retrieve-traces-with-emulators)
  - [Next Steps](#next-steps)

## Overview

Part 1 focused on setting up the toolchain and running your first program.
This part focuses on debug traces: how to add them, how to read them, and how to use them to understand program behavior.

## What Is a Debug Trace?

A debug trace is a short runtime message emitted by your program to report what it is doing.
Typical examples:

- "Init done"
- "Entered render loop"
- "x=120 y=40 iteration=27"
- "Error: file not found"

Why traces are useful:

- They show execution order.
- They confirm whether a function is reached.
- They help locate hangs, crashes, and wrong states.
- They are lightweight compared to full interactive debugging.

On Sega Saturn, traces are especially useful because development often mixes emulator and hardware workflows, where stepping with a debugger is not always convenient.

## How to Retrieve Traces

The main principle used in this workflow is:

1. Saturn code writes trace bytes to a known debug output address.
2. Emulator or hardware captures those bytes.
3. You read them on the host side (console/log/serial terminal).

### Kronos

How it works:

- Your program writes bytes to the debug trace output channel.
- Kronos (with the relevant debug support path) captures this stream and prints it to host-visible logs/console.

How you use it:

- Launch your `.cue` image with Kronos.
- Keep terminal/log output visible while the program runs.
- Correlate trace lines with on-screen behavior.

When trace output is working, the Kronos log panel should look like this:

![Kronos log output showing trace messages]({{ '/assets/img/develop_on_sega_saturn_part2/kronos-logs-display.png' | relative_url }})

This screenshot shows the emulator window running the Mandelbrot demo while the log panel receives trace messages such as `trace: main start` and `trace: video on, entering render loop`.

Kronos settings reference (Cart/Memory tab):

![Kronos Cart/Memory settings with Development Extension]({{ '/assets/img/develop_on_sega_saturn_part2/kronos-cart-memory-development-extension-settings.png' | relative_url }})

Meaning of each setting shown:

- `Cartridge = Development Extension`: enables the development/debug cartridge mode used for trace-oriented workflows.
- `Memory` path (for example `bkram.bin`): file used by Kronos to persist backup RAM data between runs.
- `Mpeg ROM`: optional MPEG card ROM path; not required for standard debug trace workflows.
- `Extend Internal backup memory to 8MB`: expands backup memory size; useful for storage-heavy scenarios but not required for basic trace capture.

Recommended for trace work:

- Keep `Cartridge` on `Development Extension`.
- Keep a valid `bkram.bin` path configured.
- Leave `Mpeg ROM` empty unless you specifically need MPEG card features.

### Mednafen

How it works:

- Same trace write path from Saturn code.
- This workflow requires the `mednafenSSDev` fork: [willll/mednafenSSDev](https://github.com/willll/mednafenSSDev).
- With the debug cart enabled, writes to debug cart space are captured and forwarded to host output.

How you use it:

1. Use the `mednafenSSDev` binary (not stock Mednafen).
2. Enable debug cart in `mednafen.cfg`:

```ini
; Expansion cart
ss.cart debug
```

3. Launch your `.cue` image with that binary.

Where traces are written:

- Standard output (terminal where Mednafen is launched).
- ImGui logs panel in dev builds (the debug cart path adds messages to the log view).

Implementation note:

- In `mednafenSSDev`, debug cart handling is in `src/ss/cart/debug.cpp`.
- The debug write handler prints characters with `fputc(..., stdout)` and flushes output.

For real hardware USB cartridge logging, continue with Part 3:

- [Develop on Sega Saturn Part 3](./develop_on_sega_saturn_part3)

## Example Project: saturn_mandelbrot

Reference project:

- [saturn_mandelbrot](https://github.com/willll/saturn_mandelbrot)

This project is a good trace example because it has a long-running render loop where progress/freeze visibility is valuable.

As of tag [v1.0](https://github.com/willll/saturn_mandelbrot/releases/tag/v1.0), `saturn_mandelbrot` ships with `debug_print(...)` calls built in. You can run the project as-is and see trace output immediately.

### Where Debug Output Is Implemented

In `src/debug.h`, the project defines `debug_print`:

```c
#define CS1(x) (0x24000000UL + (x))

void debug_print(const char *message) {
  volatile Uint8 *addr = (volatile Uint8 *)CS1(0x1000);
  const char *s = message;
  while (*s)
    *addr = (Uint8)*s++;

  if ((Uint8)*(s - 1) != '\n') {
    *addr = '\n';
  }
}
```

What this means:

- Traces are emitted byte-by-byte to address `0x24001000`.
- Emulator/hardware trace receivers can hook that write path and expose it on host side.

### Trace Calls in the Program

As of v1.0, `src/mandelbrot.c` already contains `debug_print(...)` calls at key points. You do not need to add them manually.

Actual trace calls in v1.0:

```c
debug_print("trace: main start");
// ... init ...
debug_print("trace: slInitBitMap failed");  // error path only
// ...
debug_print("trace: video on, entering render loop");
```

You can still add your own calls in `main` or loop bodies for deeper visibility. `src/mandelbrot.c` includes `debug.h`, so `debug_print(...)` is always available.

Tip:

- Keep trace lines short.
- Use consistent prefixes to grep quickly.

### Build with saturn-docker

Clone and build (tag v1.0):

```bash
git clone --branch v1.0 https://github.com/willll/saturn_mandelbrot.git
cd saturn_mandelbrot
```

Release build (ninja):

```bash
docker run -it --rm -v $(pwd):/saturn saturn-docker /bin/sh -c "
  mkdir -p /saturn/build && \
  cd /saturn/build && \
  rm -rf * && \
  cmake -DCMAKE_TOOLCHAIN_FILE=$SATURN_CMAKE/sega_saturn.cmake \
        -DCMAKE_INSTALL_PREFIX=/saturn/ \
        .. -G Ninja && \
  ninja && \
  ninja install
"
```

Debug build (ninja):

```bash
docker run -it --rm -v $(pwd):/saturn saturn-docker /bin/sh -c "
  mkdir -p /saturn/build && \
  cd /saturn/build && \
  rm -rf * && \
  cmake -DCMAKE_BUILD_TYPE=Debug \
        -DCMAKE_TOOLCHAIN_FILE=$SATURN_CMAKE/sega_saturn.cmake \
        -DCMAKE_INSTALL_PREFIX=/saturn/ \
        .. -G Ninja && \
  ninja && \
  ninja install
"
```

Important artifacts:

- `mandelbrot/mandelbrot.elf`
- `build/mandelbrot.bin`
- `build/IP.BIN`
- `mandelbrot/mandelbrot.iso`
- `mandelbrot/mandelbrot.cue`

### Run It

Kronos:

```bash
kronos -a ./mandelbrot/mandelbrot.cue
```

Mednafen:

```bash
mednafen ./mandelbrot/mandelbrot.cue
```

### Expected Success Output

When running the v1.0 build with traces enabled, you should see:

```text
trace: main start
trace: video on, entering render loop
```

If `slInitBitMap` fails (error path), you will also see:

```text
trace: slInitBitMap failed
```

### Retrieve Traces with Emulators

Kronos path:

1. Run the image in Kronos.
2. Keep host terminal/log output visible.
3. Verify lines such as `trace: main start` and `trace: video on, entering render loop` appear.

Mednafen path:

1. Make sure you are using `mednafenSSDev`.
2. Set `ss.cart debug` in `mednafen.cfg`.
3. Run the image with Mednafen.
4. Read trace lines from terminal standard output (and ImGui logs panel if enabled in your build).
5. Verify `trace: main start` and `trace: video on, entering render loop` appear in order.

## Next Steps

Continue with hardware logging in Part 3:

- [Develop on Sega Saturn Part 3](./develop_on_sega_saturn_part3)
