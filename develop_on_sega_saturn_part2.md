---
layout: default
title: "Develop on Sega Saturn Part 2"
permalink: /develop_on_sega_saturn_part2
---

# Develop on Sega Saturn Part 2

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Recommended VS Code Extensions](#recommended-vs-code-extensions)
- [Workflow Options](#workflow-options)
  - [Workflow A: Local Docker Tasks (Recommended)](#workflow-a-local-docker-tasks-recommended)
  - [Workflow B: Attach VS Code to Running Container](#workflow-b-attach-vs-code-to-running-container)
  - [Workflow C: Remote - SSH](#workflow-c-remote---ssh)
- [Configuring VS Code Tasks](#configuring-vs-code-tasks)
  - [Build Tasks](#build-tasks)
  - [Emulator Launch Tasks](#emulator-launch-tasks)
- [Complete `.vscode/tasks.json` Templates](#complete-vscodetasksjson-templates)
  - [Template for Workflow A (Local Host calling Docker)](#template-for-workflow-a-local-host-calling-docker)
  - [Template for Workflows B & C (Direct Container Execution)](#template-for-workflows-b--c-direct-container-execution)
- [C/C++ IntelliSense Configuration](#cc-intellisense-configuration)
- [Next Steps](#next-steps)

## Overview

[Part 1](./develop_on_sega_saturn_part1) covered installing the containerized Sega Saturn toolchain ([saturn-docker](https://github.com/willll/saturn-docker)) and building a basic project from the command line.

Part 2 focuses on setting up **Visual Studio Code** as a complete IDE for Sega Saturn development. With a properly configured editor, you can edit source code, trigger Docker builds with hotkeys, inspect compilation outputs, and launch emulators directly from VS Code.

Reference guide:

- [saturn-docker VisualIdiot.md](https://github.com/willll/saturn-docker/blob/main/Documentation/VisualIdiot.md)

## Prerequisites

Before configuring VS Code, ensure you have completed [Part 1](./develop_on_sega_saturn_part1):

- Docker (or Rancher Desktop / Colima / Podman) installed and running.
- The `saturn-docker` image built locally (`docker build -t saturn-docker .`), or an alternative toolchain/SDK setup such as [SaturnRingLib](https://github.com/ReyeMe/SaturnRingLib), [libyaul-docker](https://github.com/yaul-org/libyaul-docker), or [Jo Engine](https://github.com/johannes-fetz/joengine).
- Visual Studio Code installed on your host machine.

## Recommended VS Code Extensions

Open VS Code Extensions (`Ctrl+Shift+X` or `Cmd+Shift+X`) and install the following:

- **[C/C++ (Microsoft)](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)** – Provides C/C++ IntelliSense, syntax highlighting, and code navigation.
- **[C/C++ Extension Pack (Microsoft)](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools-extension-pack)** – Includes CMake Tools and helper extensions.
- **[CMake Tools (Microsoft)](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools)** – Syntax and build integration for CMake projects.
- **[Dev Containers (Microsoft)](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)** – Allows opening your project folder directly inside the `saturn-docker` container.
- **[Remote - SSH (Microsoft)](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh)** *(Optional)* – For developing on a remote build server or virtual machine.

## Workflow Options

### Workflow A: Local Docker Tasks (Recommended)

This is the simplest setup for most developers:

<details markdown="1">
<summary style="cursor: pointer;"><strong>👉 Click here to expand Workflow A instructions</strong></summary>

1. Keep your source code on your host machine.
2. Edit files natively in VS Code.
3. Configure VS Code build tasks (`.vscode/tasks.json`) to run `docker run` commands against your local workspace.
4. Run launch tasks to open emulators (Kronos or Mednafen) on your host operating system.

**Advantages:** Emulators run directly on your host GPU/display server without complex X11/Wayland forwarding, while compilation happens inside the container.

</details>

<br/>

---

### Workflow B: Attach VS Code to Running Container

Instead of generating complex container configurations, you can attach VS Code directly to a container running in your terminal:

<details markdown="1">
<summary style="cursor: pointer;"><strong>👉 Click here to expand Workflow B instructions</strong></summary>

1. **Start the container in your terminal**, sharing your project folder between host and container:
   ```bash
   docker run -it --rm -v $(pwd):/saturn saturn-docker /bin/bash
   ```
2. **Attach VS Code to the running container**:
   - Open VS Code with the **Dev Containers** extension installed.
   - Open the Command Palette (`Ctrl+Shift+P` / `Cmd+Shift+P`).
   - Select **Dev Containers: Attach to Running Container...** and choose your running `saturn-docker` instance.
3. **Open the shared project folder**:
   - A new VS Code window will open connected to the container's environment.
   - Click **Open Folder** and select `/saturn` (the folder shared between the host and container).
4. **Develop and build inside the container**:
   - All source edits in VS Code are saved directly to your shared host directory.
   - You can compile inside the integrated terminal using the Saturn toolchain (`sh-elf-gcc`, `cmake`, `make`).

> **Note:** Build tasks execute directly inside the container environment, while emulators (Kronos / Mednafen) can be launched on the host machine pointing to the shared build artifacts.

</details>

<br/>

---

### Workflow C: Remote - SSH

Use this if your build engine or test machine runs on a dedicated server or virtual machine:

<details markdown="1">
<summary style="cursor: pointer;"><strong>👉 Click here to expand Workflow C instructions</strong></summary>

1. **Launch `saturn-docker` with SSH enabled** and your workspace folder mounted:
   ```bash
   docker run -d -p 2222:22 -v $(pwd):/saturn saturn-docker
   ```
2. **Use VS Code Remote - SSH to connect**:
   ```bash
   ssh root@<host-ip> -p 2222
   ```
3. **Open `/saturn` in VS Code** and develop remotely.

#### Testing Locally with Shared Drives

When using Remote - SSH, VS Code runs directly inside the remote container environment. Code edits, syntax checks, and builds occur inside the container on `/saturn`.

However, graphical emulators (Kronos / Mednafen) must be launched on your host machine. For local testing:

- **Shared Volume / Mount:** Ensure your workspace directory is mounted (`-v $(pwd):/saturn`), or accessible over a network share (NFS, SMB, or `sshfs`).
- **Artifact Access:** When compilation completes inside the remote container, the resulting `.iso`, `.cue`, and `.elf` files write back to the shared directory.
- **Run Emulator:** Launch Kronos or Mednafen from your local host targeting the shared `.cue` file (`kronos -a ${workspaceFolder}/helloworld/helloworld.cue`).

</details>

<br/>

---

## Configuring VS Code Tasks

VS Code tasks automate your compilation and emulation workflow. You can trigger them using `Ctrl+Shift+B` (Build Task) or via **Terminal -> Run Task...**.

### Build Tasks

Build tasks pass your current workspace directory into `saturn-docker` via volume mounts (`-v ${workspaceFolder}:/saturn`).

### Emulator Launch Tasks

Emulator tasks invoke Kronos or Mednafen on your host machine, pointing to the generated `.cue` file in your workspace build directory.

---

## Complete `.vscode/tasks.json` Templates

Depending on which workflow you select, choose the corresponding `.vscode/tasks.json` template for your project.

> **Note on `$SATURN_CMAKE`:** `$SATURN_CMAKE` is automatically defined by `saturn-docker`'s environment setup inside the container (see [Part 1](./develop_on_sega_saturn_part1#what-saturn-docker-installs)).

### Template for Workflow A (Local Host calling Docker)

In **Workflow A**, VS Code runs on your host machine and invokes `docker run` to execute build steps inside container volume mounts:

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Compile Docker [RELEASE]",
      "type": "shell",
      "command": "docker run --rm -i -v ${workspaceFolder}:/saturn saturn-docker /bin/sh -c 'mkdir -p /saturn/build && cd /saturn/build && rm -rf * && cmake -DCMAKE_TOOLCHAIN_FILE=$SATURN_CMAKE/sega_saturn.cmake -DCMAKE_INSTALL_PREFIX=/saturn/ .. && make all && make install'",
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "problemMatcher": ["$gcc"]
    },
    {
      "label": "Compile Docker [DEBUG]",
      "type": "shell",
      "command": "docker run --rm -i -v ${workspaceFolder}:/saturn saturn-docker /bin/sh -c 'mkdir -p /saturn/build && cd /saturn/build && rm -rf * && cmake -DCMAKE_TOOLCHAIN_FILE=$SATURN_CMAKE/sega_saturn.cmake -DCMAKE_INSTALL_PREFIX=/saturn/ -DCMAKE_BUILD_TYPE=Debug .. && make all && make install'",
      "group": "build",
      "problemMatcher": ["$gcc"]
    },
    {
      "label": "Clean Build Directory",
      "type": "shell",
      "command": "rm -rf ${workspaceFolder}/build ${workspaceFolder}/helloworld/*.iso ${workspaceFolder}/helloworld/*.cue ${workspaceFolder}/helloworld/*.elf",
      "problemMatcher": []
    },
    {
      "label": "Run with Kronos",
      "type": "shell",
      "command": "kronos -a ${workspaceFolder}/helloworld/helloworld.cue",
      "problemMatcher": []
    },
    {
      "label": "Run with Mednafen",
      "type": "shell",
      "command": "mednafen ${workspaceFolder}/helloworld/helloworld.cue",
      "problemMatcher": []
    }
  ]
}
```

### Template for Workflows B & C (Direct Container Execution)

In **Workflows B & C**, VS Code is already connected inside the container environment. Compilation commands run directly in the container shell without `docker run`:

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Compile Container [RELEASE]",
      "type": "shell",
      "command": "mkdir -p ${workspaceFolder}/build && cd ${workspaceFolder}/build && rm -rf * && cmake -DCMAKE_TOOLCHAIN_FILE=$SATURN_CMAKE/sega_saturn.cmake -DCMAKE_INSTALL_PREFIX=${workspaceFolder}/ .. && make all && make install",
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "problemMatcher": ["$gcc"]
    },
    {
      "label": "Compile Container [DEBUG]",
      "type": "shell",
      "command": "mkdir -p ${workspaceFolder}/build && cd ${workspaceFolder}/build && rm -rf * && cmake -DCMAKE_TOOLCHAIN_FILE=$SATURN_CMAKE/sega_saturn.cmake -DCMAKE_INSTALL_PREFIX=${workspaceFolder}/ -DCMAKE_BUILD_TYPE=Debug .. && make all && make install",
      "group": "build",
      "problemMatcher": ["$gcc"]
    },
    {
      "label": "Clean Build Directory",
      "type": "shell",
      "command": "rm -rf ${workspaceFolder}/build ${workspaceFolder}/helloworld/*.iso ${workspaceFolder}/helloworld/*.cue ${workspaceFolder}/helloworld/*.elf",
      "problemMatcher": []
    }
  ]
}
```

> **Tip:** Adjust paths (such as `helloworld/helloworld.cue`) to match your specific project executable and cue sheet name.
>
> **Note on Emulator Launch Tasks:** Emulator tasks are omitted from Workflows B & C because GUI emulators (Kronos / Mednafen) run on your host OS. Launch emulators directly from your host terminal or host VS Code instance, pointing to the shared build artifacts.
>
> **Note on Task Labels:** Task labels (`"Compile Docker [...]"` vs. `"Compile Container [...]"`) distinguish whether the task invokes an external `docker run` command from the host (Workflow A) or compiles directly inside an attached container shell (Workflows B & C).

---

## C/C++ IntelliSense Configuration

To prevent VS Code from showing false syntax errors for Sega Saturn SDK headers (`SCL_Vdp2Init`, `slInitBitMap`, etc.), create `.vscode/c_cpp_properties.json`:

```json
{
  "configurations": [
    {
      "name": "Saturn Docker",
      "includePath": [
        "${workspaceFolder}/**"
      ],
      "defines": [
        "__SH2__"
      ],
      "compilerPath": "/usr/bin/gcc",
      "cStandard": "c11",
      "cppStandard": "c++17",
      "intelliSenseMode": "gcc-x64"
    }
  ],
  "version": 4
}
```

> **Tip:** When using **Dev Containers**, IntelliSense automatically resolves SDK headers from `/opt/saturn/**`. When developing in **Local Docker Task** mode, you can copy or mount local copies of the SDK headers (e.g. SBL, Yaul, Jo Engine) and add their paths to `"includePath"` to enable code completion and eliminate squiggly lines.

---

## Next Steps

Now that your IDE is configured for seamless building and emulator execution, move on to **Part 3** to learn how to add runtime debug traces and view emulator console outputs:

- [Part 3: Emulator Debug Trace](./develop_on_sega_saturn_part3)
