---
layout: default
title: "Develop on Sega Saturn Part 1"
permalink: /develop_on_sega_saturn_part1
---

# Develop on Sega Saturn Part 1

## Table of Contents

- [Develop on Sega Saturn Part 1](#develop-on-sega-saturn-part-1)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Install the Development Environment](#install-the-development-environment)
    - [Before You Start](#before-you-start)
    - [Windows](#windows)
    - [macOS](#macos)
    - [Linux](#linux)
    - [First Successful Run Checklist](#first-successful-run-checklist)
    - [Common Beginner Issues](#common-beginner-issues)
    - [Typical Questions (saturn-docker README)](#typical-questions-saturn-docker-readme)
    - [What This Brings](#what-this-brings)
  - [What saturn-docker Installs](#what-saturn-docker-installs)
  - [Use Visual Studio Code for Saturn Development](#use-visual-studio-code-for-saturn-development)
    - [Recommended Extensions](#recommended-extensions)
    - [Workflow A (Easy): Build with Local Docker from VS Code Tasks](#workflow-a-easy-build-with-local-docker-from-vs-code-tasks)
    - [Workflow B: SSH into Container/Remote Host](#workflow-b-ssh-into-containerremote-host)
    - [Dev Containers Option](#dev-containers-option)
    - [CMake and Build/Run Tasks](#cmake-and-buildrun-tasks)
    - [Minimal Beginner Loop in VS Code](#minimal-beginner-loop-in-vs-code)
  - [Hello World](#hello-world)
    - [Understand the Source Code](#understand-the-source-code)
    - [Build It with the Docker Image](#build-it-with-the-docker-image)
    - [Build Artifacts (What You Get)](#build-artifacts-what-you-get)
    - [Run with Kronos](#run-with-kronos)
    - [Run with Mednafen](#run-with-mednafen)
    - [Quick Troubleshooting](#quick-troubleshooting)
  - [Next Steps](#next-steps)

## Overview

This series focuses on Sega Saturn development and debugging.
Part 1 is emulator-first and explains how to install a development environment and build a first program.
Part 2 will focus on hardware debugging.
More parts will be added in the future as my development environment evolves.

## Install the Development Environment

The recommended setup is based on this project:

- [saturn-docker](https://github.com/willll/saturn-docker)

This project provides a containerized Sega Saturn toolchain so you can start quickly without manually installing every dependency on your host machine.
In simple terms, Docker gives you a preconfigured Linux box that always behaves the same way on every computer.
That means fewer "works on my machine" problems and easier debugging.

### Before You Start

You need:

- A GitHub account and internet access.
- Git installed on your machine.
- Docker installed and running.
- At least 20 GB free disk space (first build can be large).

Quick checks:

```bash
git --version
docker --version
docker info
docker run hello-world
```

If these commands work, you are ready.

### Windows

<details markdown="1">
<summary>View Windows Instructions</summary>

Use one of these free alternatives:

- [Rancher Desktop](https://rancherdesktop.io/) (recommended for beginners)
- [Podman Desktop](https://podman-desktop.io/)
- WSL2 + Docker Engine inside your Linux distribution

Recommended setup (Rancher Desktop):

1. Install Rancher Desktop.
2. In settings, enable the Docker-compatible CLI so `docker` commands work.
3. Open PowerShell or WSL and verify:

```bash
docker --version
docker info
docker run hello-world
```

4. Clone the project:

```bash
git clone https://github.com/willll/saturn-docker.git
cd saturn-docker
```

5. Build the image (first time can take a while):

```bash
docker build -t saturn-docker . --file ./Dockerfile
```

6. Start the container shell:

```bash
docker run -it --rm -v $(pwd):/saturn saturn-docker /bin/bash
```

If `$(pwd)` does not work in PowerShell, run the same commands in WSL.

</details>

### macOS

<details markdown="1">
<summary>View macOS Instructions</summary>

Use one of these free alternatives:

- [Colima](https://github.com/abiosoft/colima) + Docker CLI (recommended)
- [Rancher Desktop](https://rancherdesktop.io/)
- [Podman Desktop](https://podman-desktop.io/)

Recommended setup (Colima):

1. Install tools:

```bash
brew install docker colima
```

2. Start Colima:

```bash
colima start
```

3. Verify Docker-compatible CLI works:

```bash
docker --version
docker info
docker run hello-world
```

4. Open Terminal, then run:

```bash
git clone https://github.com/willll/saturn-docker.git
cd saturn-docker
docker build -t saturn-docker . --file ./Dockerfile
docker run -it --rm -v $(pwd):/saturn saturn-docker /bin/bash
```

The environment is the same as Windows and Linux because everything runs inside the container.

</details>

### Linux

<details markdown="1">
<summary>View Linux Instructions</summary>

1. Install Docker Engine (or Docker Desktop).
2. Ensure your user can run Docker commands.
3. Open a terminal and run:

```bash
git clone https://github.com/willll/saturn-docker.git
cd saturn-docker
docker build -t saturn-docker . --file ./Dockerfile
docker run -it --rm -v $(pwd):/saturn saturn-docker /bin/bash
```

Linux is usually the simplest path because Docker integration is native.

</details>

### First Successful Run Checklist

Inside the container, verify that you reached a Linux shell prompt and can run basic commands. For example, test that the SH2 compiler is installed and check the core SDK directories:

```bash
whoami
sh-elf-gcc --version
ls -la /opt/saturn
```

The output of `ls -la /opt/saturn` should reveal key folders such as `sgl`, `sbl`, `joengine`, `yaul`, `SaturnRingLib`, and the `toolchain` itself.

If that works, your Saturn development environment is installed correctly.

### Common Beginner Issues

- Docker-compatible runtime is installed but not running: start Rancher Desktop/Podman/your container service and retry.
- Permission errors on Linux: add your user to the docker group, then log out and back in.
- Slow first build: normal, because toolchains and dependencies are downloaded.
- Volume/path issues on Windows: prefer WSL if PowerShell path mapping causes trouble.
- Volume/path issues on macOS: retry from a local folder (for example under your home directory) and confirm Colima/Rancher is running before `docker run`.

### Typical Questions (saturn-docker README)

If you need more details, use these direct links to the README:

- How do I build the Docker image?: [Build it](https://github.com/willll/saturn-docker#build-it)
- How do I run the container?: [Run it](https://github.com/willll/saturn-docker#run-it)
- How do I map USB/serial devices into the container?: [Map a device from host to container](https://github.com/willll/saturn-docker#map-a-device-from-host-to-container)
- How do I use it from an IDE?: [Use it in an IDE](https://github.com/willll/saturn-docker#use-it-in-an-ide)
- Can I change the SH2 GCC version?: [Change GCC Version for SH2](https://github.com/willll/saturn-docker#change-gcc-version-for-sh2)
- Is there a prebuilt image?: [Docker Hub](https://github.com/willll/saturn-docker#docker-hub)
- What do all build variables mean?: [List of variables](https://github.com/willll/saturn-docker#list-of-variables)

### What This Brings

- A consistent and portable build environment.
- Faster onboarding for new contributors.
- Easier debugging because all developers run the same tools.
- Cleaner host machines with fewer Saturn-specific packages installed globally.

## What saturn-docker Installs

saturn-docker installs a full Sega Saturn development stack.
The exact enabled set can evolve over time, so use the project README as the source of truth:

- [saturn-docker tools/build status](https://github.com/willll/saturn-docker#tools)

Main components and their purpose:

- **GCC for SH2**: cross-compiler used to build code for Saturn SH2 CPUs. Project: [Saturn-SDK-GCC-SH2](https://github.com/willll/Saturn-SDK-GCC-SH2)
- **GCC for M68K**: cross-compiler for the Saturn sound CPU toolchain. Project: [Saturn-SDK-GCC-M68K](https://github.com/willll/Saturn-SDK-GCC-M68K)
- **CMake profile/tooling**: build-system generation and project configuration. Project: [CMake](https://cmake.org/)
- **schily-tools**: ISO/image authoring utilities used in disc build workflows. Project: [schilytools](https://github.com/clausecker/schilytools)
- **IP.BIN generation tool**: creates Saturn boot metadata. Project: [Saturn-SDK-Tool-IPMaker](https://github.com/willll/Saturn-SDK-Tool-IPMaker)
- **satconv (satcon)**: image conversion utility for Saturn asset pipelines. Project: [satconv](https://git.sr.ht/~ndiddy/satconv/)
- **ftx**: Sega Saturn USB flash cart transfer utility (upload/download/execute/debug console). Project: [ftx](https://github.com/willll/ftx)
- **SGL support**: Sega Graphics Library integration for legacy SDK-style projects.
- **SBL support**: Sega Basic Library integration. Reference project: [shicky256/sbl](https://github.com/shicky256/sbl)
- **Jo Engine**: beginner-friendly Saturn game framework. Project: [jo-engine.org](https://www.jo-engine.org/)
- **Yaul**: modern Saturn homebrew development library/tooling. Project: [libyaul](https://github.com/yaul-org/libyaul)
- **SRL (Saturn Ring Lib)**: Saturn helper library with examples. Project: [SaturnRingLib](https://github.com/ReyeMe/SaturnRingLib)

If you want to customize what gets installed, check the build variables section:

- [saturn-docker list of variables](https://github.com/willll/saturn-docker#list-of-variables)

## Use Visual Studio Code for Saturn Development

Reference guide:

- [saturn-docker VisualIdiot.md](https://github.com/willll/saturn-docker/blob/main/Documentation/VisualIdiot.md)

This is a practical VS Code workflow for beginners using saturn-docker.

### Recommended Extensions

- [C/C++ (Microsoft)](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)
- [C/C++ Extension Pack (Microsoft)](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools-extension-pack)
- [CMake Tools (Microsoft)](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools)
- [Dev Containers (Microsoft)](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
- [Remote - SSH (Microsoft)](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh) (optional)

### Workflow A (Easy): Build with Local Docker from VS Code Tasks

This is the simplest option if Docker runs on your local machine.

1. Clone your project (for example `saturn_helloworld`) locally.
2. Open the project folder in VS Code.
3. Use tasks that call Docker directly (same style as `Compile Docker [RELEASE]` in the sample `.vscode/tasks.json`).
4. Build from VS Code Terminal or Task Runner.

Why this is beginner-friendly:

- No manual Saturn SDK install on host.
- Reproducible builds.
- Easy to reset by rebuilding the container.

### Workflow B: SSH into Container/Remote Host

Use this when your build environment is not local (or when you prefer remote workflows).

1. Start saturn-docker with SSH port mapping (`-p 2222:22`).
2. Configure Remote-SSH in VS Code.
3. Connect to your target (example pattern: `ssh root@<host> -p 2222`).
4. Open the shared Saturn project folder.

Notes:

- In SSH mode, task execution context can be different from local mode.
- If needed, use sshfs or shared folders so source files are visible where you edit.

### Dev Containers Option

You can also attach VS Code directly to the saturn-docker container via Dev Containers.
Once attached, builds run inside the container environment.

Important:

- When VS Code terminal is inside the container, emulator launch on the host is not always direct.
- Keep build and emulator tasks separated if needed (build in container, run emulator from host).

### CMake and Build/Run Tasks

The saturn_helloworld repository includes example VS Code task definitions for:

- Compile (release/debug)
- Clean
- Run with Kronos
- Run with Mednafen

Use these as templates and adapt paths for your machine (especially Mednafen binary path).

### Minimal Beginner Loop in VS Code

1. Edit source in VS Code.
2. Run Docker compile task.
3. Confirm artifacts are generated (`.elf`, `.iso`, `.cue`).
4. Launch Kronos or Mednafen with the generated `.cue` file.
5. Repeat after each change.

## Hello World

After the environment is ready, start with this project:

- [saturn_helloworld](https://github.com/willll/saturn_helloworld)

This repository is a minimal Sega Saturn example (SBL-based) to validate your full workflow: source code, build, assets, ISO creation, and emulator execution.
It is the best first checkpoint before moving to larger projects.

### Understand the Source Code

If you are new to Saturn programming, think of this sample as:

- setup video/debug systems
- print text on screen
- wait for controller input
- exit when Start is pressed

The project files are small and each one has a clear role:

- `src/helloworld.c`: the main program logic.
- `src/v_blank/v_blank.c` and `src/v_blank/set_vb.c`: V-Blank hook and per-frame updates.
- `src/smpclib/*`: controller/peripheral helper code.
- `CMakeLists.txt`: describes how to compile, link, and generate Saturn disc files.

Beginner walkthrough of `src/helloworld.c`:

1. `main()` starts and initializes Saturn display systems (`SCL_Vdp2Init`, sprite mode/priority setup).
2. `SetVblank()` installs a V-Blank callback so game-state/input updates can happen each frame.
3. `DBG_Initial`, `DBG_ClearScreen`, and `DBG_DisplayOn` enable a simple text debug console.
4. `DBG_Printf("Hello World !\n")` writes the message on screen.
5. The `for(;;)` loop keeps the program running and checks controller state.
6. When Start is pressed (`PadData1 & PAD_S`), the sample prints `Exit !` and breaks.

Code excerpt (simplified view of startup and debug output):

```c
int main()
{
	SCL_Vdp2Init();
	SetVblank();
	set_imask(0);

	SCL_SetPriority(SCL_SP0|SCL_SP1|SCL_SP2|SCL_SP3|
					SCL_SP4|SCL_SP5|SCL_SP6|SCL_SP7, 6);
	SCL_SetSpriteMode(SCL_TYPE1, SCL_MIX, SCL_SP_WINDOW);

	SPR_SetEraseData(RGB16_COLOR(0,0,0), 0, 0, DISP_XSIZE - 1, DISP_YSIZE - 1);
	SCL_SetFrameInterval(1);

	DBG_Initial(&PadData1, RGB16_COLOR(31,31,31), 0);
	DBG_ClearScreen();
	DBG_DisplayOn();
	DBG_Printf("Hello World !\n");
```

Code excerpt (main loop and exit condition):

```c
	for (int i = 0;; i++) {
		if (PadData1 & PAD_S) {
			DBG_Printf("Exit !\n");
			break;
		}
	}

	return 0;
}
```

Code excerpt (toolchain/build entry in CMake):

```cmake
add_executable(${TARGET_NAME}.elf ${SOURCES})

add_custom_command(OUTPUT ${TARGET_NAME}.bin
				  COMMAND ${CMAKE_OBJCOPY}
				  ARGS -O binary ${TARGET_NAME}.elf ${TARGET_NAME}.bin)

add_custom_command(OUTPUT IP.BIN
				  COMMAND $ENV{SATURN_IPMAKER}/IPMaker
				  ARGS -v -o ${CMAKE_CURRENT_BINARY_DIR}/IP.BIN -t ${TARGET_NAME} -p 2)
```

The practical meaning of these snippets:

- `.elf` is the debug-friendly executable.
- `.bin` is the raw program image generated from the ELF.
- `IP.BIN` is Saturn boot metadata required for disc boot.

Why V-Blank matters:

- Saturn programs usually sync work with frame timing.
- V-Blank is the safe period between frames, commonly used for input/update tasks.

Quick glossary for the first read:

- `SCL_*`: screen/display configuration (VDP2 side).
- `SPR_*`: sprite system configuration.
- `DBG_*`: debug text output.
- `PAD_S`: Start button bitmask.

### Build It with the Docker Image

Clone the repository:

```bash
git clone https://github.com/willll/saturn_helloworld.git
cd saturn_helloworld
```

Run the build inside saturn-docker:

```bash
docker run -it --rm -v $(pwd):/saturn saturn-docker /bin/sh -c "mkdir -p /saturn/build && cd /saturn/build && rm -rf * && cmake -DCMAKE_TOOLCHAIN_FILE=$SATURN_CMAKE/sega_saturn.cmake -DCMAKE_INSTALL_PREFIX=/saturn/ .. && make all && make install"
```

Debug build variant:

```bash
docker run -it --rm -v $(pwd):/saturn saturn-docker /bin/sh -c "mkdir -p /saturn/build && cd /saturn/build && rm -rf * && cmake -DCMAKE_TOOLCHAIN_FILE=$SATURN_CMAKE/sega_saturn.cmake -DCMAKE_INSTALL_PREFIX=/saturn/ -DCMAKE_BUILD_TYPE=Debug .. && make all && make install"
```

### Build Artifacts (What You Get)

After build/install, the important outputs are:

- `helloworld/helloworld.elf`: executable with symbols (useful for debugging).
- `build/helloworld.bin`: raw binary generated from ELF.
- `build/IP.BIN`: Saturn boot metadata generated by IPMaker.
- `helloworld/helloworld.iso`: disc image used by emulators.
- `helloworld/helloworld.cue`: cue sheet pointing to the ISO track.
- `build/helloworld.map`: link map file useful for symbol/address analysis.

Tip: if you need to map crash addresses back to source, use `addr2line` with `helloworld.elf`.

### Run with Kronos

Use the generated cue file:

```bash
kronos -a ./helloworld/helloworld.cue
```

Expected result: black background and debug text including `Hello World !`.

### Run with Mednafen

Use your Mednafen binary and the same cue file:

```bash
mednafen ./helloworld/helloworld.cue
```

If you are using the custom Mednafen build from your setup, run that binary path instead (for example the path configured in the repository VS Code tasks).

### Quick Troubleshooting

- Missing `helloworld.iso`: ensure `make install` completed.
- Emulator boots but black screen forever: verify cue path and that `helloworld.iso` is next to `helloworld.cue`.
- Build fails with missing Saturn variables: make sure you are building inside saturn-docker where `SATURN_CMAKE` and related variables are defined.

## Next Steps

Part 2 will cover debugging with logs.
