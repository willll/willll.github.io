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
  - [Workflow B: Dev Containers](#workflow-b-dev-containers)
  - [Workflow C: Remote - SSH](#workflow-c-remote---ssh)
- [Configuring VS Code Tasks](#configuring-vs-code-tasks)
  - [Build Tasks](#build-tasks)
  - [Emulator Launch Tasks](#emulator-launch-tasks)
- [Complete `.vscode/tasks.json` Template](#complete-vscodetasksjson-template)
- [C/C++ IntelliSense Configuration](#cc-intellisense-configuration)
- [Next Steps](#next-steps)

## Overview

Part 1 covered installing the containerized Sega Saturn toolchain (`saturn-docker`) and building a basic project from the command line.

Part 2 focuses on setting up **Visual Studio Code** as a complete IDE for Sega Saturn development. With a properly configured editor, you can edit source code, trigger Docker builds with hotkeys, inspect compilation outputs, and launch emulators directly from VS Code.

Reference guide:

- [saturn-docker VisualIdiot.md](https://github.com/willll/saturn-docker/blob/main/Documentation/VisualIdiot.md)

## Prerequisites

Before configuring VS Code, ensure you have completed Part 1:

- Docker (or Rancher Desktop / Colima / Podman) installed and running.
- The `saturn-docker` image built locally (`docker build -t saturn-docker .`).
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

1. Keep your source code on your host machine.
2. Edit files natively in VS Code.
3. Configure VS Code build tasks (`.vscode/tasks.json`) to run `docker run` commands against your local workspace.
4. Run launch tasks to open emulators (Kronos or Mednafen) on your host operating system.

**Advantages:** Emulators run directly on your host GPU/display server without complex X11/Wayland forwarding, while compilation happens inside the container.

---

### Workflow B: Dev Containers

If you prefer full container integration where VS Code's extension host runs inside Docker:

1. Create a `.devcontainer/devcontainer.json` file in your repository:

```json
{
  "name": "Sega Saturn Dev",
  "image": "saturn-docker:latest",
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-vscode.cpptools",
        "ms-vscode.cmake-tools"
      ]
    }
  },
  "workspaceMount": "source=${localWorkspaceFolder},target=/saturn,type=bind",
  "workspaceFolder": "/saturn"
}
```

2. Open the Command Palette (`Ctrl+Shift+P` / `Cmd+Shift+P`) and choose **Dev Containers: Reopen in Container**.

> **Note:** When using Dev Containers, build tasks execute directly in the container terminal (`sh-elf-gcc`), but host emulators should still be invoked on the host machine.

---

### Workflow C: Remote - SSH

Use this if your build engine or test machine runs on a dedicated server:

1. Launch `saturn-docker` with SSH enabled (`-p 2222:22`).
2. Use VS Code **Remote - SSH** to connect (`ssh root@<host-ip> -p 2222`).
3. Open `/saturn` in VS Code and develop remotely.

---

## Configuring VS Code Tasks

VS Code tasks automate your compilation and emulation workflow. You can trigger them using `Ctrl+Shift+B` (Build Task) or via **Terminal -> Run Task...**.

### Build Tasks

Build tasks pass your current workspace directory into `saturn-docker` via volume mounts (`-v ${workspaceFolder}:/saturn`).

### Emulator Launch Tasks

Emulator tasks invoke Kronos or Mednafen on your host machine, pointing to the generated `.cue` file in your workspace build directory.

---

## Complete `.vscode/tasks.json` Template

Create or update `.vscode/tasks.json` in your Saturn project with the following template:

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Compile Docker [RELEASE]",
      "type": "shell",
      "command": "docker run -it --rm -v ${workspaceFolder}:/saturn saturn-docker /bin/sh -c 'mkdir -p /saturn/build && cd /saturn/build && rm -rf * && cmake -DCMAKE_TOOLCHAIN_FILE=$SATURN_CMAKE/sega_saturn.cmake -DCMAKE_INSTALL_PREFIX=/saturn/ .. && make all && make install'",
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "problemMatcher": ["$gcc"]
    },
    {
      "label": "Compile Docker [DEBUG]",
      "type": "shell",
      "command": "docker run -it --rm -v ${workspaceFolder}:/saturn saturn-docker /bin/sh -c 'mkdir -p /saturn/build && cd /saturn/build && rm -rf * && cmake -DCMAKE_TOOLCHAIN_FILE=$SATURN_CMAKE/sega_saturn.cmake -DCMAKE_INSTALL_PREFIX=/saturn/ -DCMAKE_BUILD_TYPE=Debug .. && make all && make install'",
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

> **Tip:** Adjust paths (such as `helloworld/helloworld.cue`) to match your specific project executable and cue sheet name.

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

---

## Next Steps

Now that your IDE is configured for seamless building and emulator execution, move on to **Part 3** to learn how to add runtime debug traces and view emulator console outputs:

- [Part 3: Emulator Debug Trace](./develop_on_sega_saturn_part3)
