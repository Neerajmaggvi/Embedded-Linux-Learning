# Embedded Linux Learning - Toolchain & Boot Process

## 📚 What I Learned

This document summarizes my understanding of the Embedded Linux Toolchain and the Linux Boot Process.

---

# 1. What is a Toolchain?

A **toolchain** is a collection of software tools used to convert source code (C/C++) into machine code that can run on an embedded device.

## Toolchain Flow

```text
C Source Code
      │
      ▼
 Compiler (GCC)
      │
      ▼
 Assembly Code
      │
      ▼
 Assembler
      │
      ▼
 Object Files
      │
      ▼
 Linker
      │
      ▼
 Executable
```

---

# 2. Components of a GNU Toolchain

## Binutils

Provides binary utilities such as:

- Assembler (`as`)
- Linker (`ld`)
- Object copy (`objcopy`)
- Object dump (`objdump`)

### Purpose

- Converts assembly into machine code.
- Links multiple object files into one executable.

---

## GCC (GNU Compiler Collection)

Converts C/C++ source code into assembly language.

Supports multiple languages:

- C
- C++
- Go
- Ada
- Fortran
- Java

---

## C Library

Provides standard library functions such as:

- printf()
- scanf()
- fopen()
- malloc()
- strlen()

These functions eventually communicate with the Linux kernel using **system calls**.

---

# 3. Native vs Cross Toolchain

## Native Toolchain

Compiler and executable run on the same CPU architecture.

Example:

```text
x86 PC
   │
gcc
   │
Executable runs on x86
```

---

## Cross Toolchain

Compiler runs on one architecture but creates executables for another architecture.

Example:

```text
Laptop (x86)
       │
aarch64-linux-gnu-gcc
       │
ARM Executable
       │
Runs on Raspberry Pi
```

Cross compilation is commonly used in Embedded Linux.

---

# 4. Yocto

Yocto is **not an operating system**.

It is a build system that generates a complete Embedded Linux distribution.

Yocto can generate:

- Bootloader
- Linux Kernel
- Device Tree
- Root Filesystem
- Cross Toolchain
- Libraries
- Applications

---

# 5. What is a Bootloader?

A bootloader is a small program that starts immediately after the device is powered on.

Its main responsibilities are:

- Initialize hardware
- Initialize RAM
- Load the Linux kernel
- Load the Device Tree Blob (DTB)
- Pass boot arguments
- Transfer control to the Linux kernel

---

# 6. Linux Boot Sequence

```text
Power ON
     │
     ▼
CPU wakes up
     │
     ▼
CPU executes ROM Code
     │
     ▼
ROM Code loads SPL
     │
     ▼
CPU executes SPL
     │
     ▼
SPL initializes RAM
     │
     ▼
SPL loads U-Boot
     │
     ▼
CPU executes U-Boot
     │
     ▼
U-Boot loads Linux Kernel
     │
     ▼
CPU executes Linux Kernel
     │
     ▼
Linux starts
```

---

# 7. ROM Code

ROM Code is permanently stored inside the processor.

It is programmed by the chip manufacturer.

Main responsibilities:

- Runs immediately after power-on.
- Finds the bootloader.
- Loads the first bootloader stage (SPL).

---

# 8. SPL (Secondary Program Loader)

SPL is a small bootloader.

Responsibilities:

- Initialize RAM
- Initialize memory controller
- Prepare essential hardware
- Load the full bootloader (U-Boot)

---

# 9. U-Boot

U-Boot is the full bootloader.

Responsibilities:

- Read kernel from storage
- Load Device Tree Blob (DTB)
- Pass boot arguments
- Load initramfs (optional)
- Copy kernel into RAM
- Transfer control to the Linux kernel

---

# 10. Linux Kernel

The Linux kernel is a program written mainly in C.

The CPU executes the kernel just like any other program.

The kernel is responsible for:

- Memory management
- Process management
- Device drivers
- Filesystems
- Scheduling
- Hardware abstraction

---

# 11. What does "CPU executes the kernel" mean?

After the bootloader copies the Linux kernel into RAM, it tells the CPU where the kernel starts.

The CPU then begins executing the kernel's machine instructions.

```text
Bootloader
      │
Copies Kernel into RAM
      │
      ▼
CPU starts executing the Kernel
```

---

# 12. Device Tree Blob (DTB)

Modern Embedded Linux systems use a **Device Tree Blob (.dtb)** instead of machine numbers.

The DTB describes the hardware available on the board.

Typical information includes:

- CPU type
- RAM size
- GPIO controllers
- UART
- I2C
- SPI
- USB
- Ethernet
- Interrupts
- Clock configuration

The bootloader loads the DTB and passes it to the Linux kernel.

---

# 13. Information Passed from Bootloader to Kernel

Before starting the kernel, the bootloader provides:

- Device Tree Blob (hardware description)
- RAM information
- CPU information
- Kernel command line
- initramfs location (optional)

Example boot arguments:

```text
console=ttyS0
root=/dev/mmcblk0p2
```

---
# 14. Linux Kernel

The **Linux Kernel** is the **third major component of an Embedded Linux system**. It is the core of the operating system and acts as a bridge between **applications** and **hardware**.

---

# Three Main Jobs of the Kernel

## 1. Manage System Resources

The kernel manages all important system resources, such as:

- CPU
- RAM (Memory)
- Storage
- Network
- USB devices

### Example

If Chrome, VS Code, and VLC are running at the same time, the kernel decides:

- Which application gets CPU time
- How much memory each application receives
- Which application should wait

Without the kernel, applications would compete for hardware resources and the system would become unstable.

---

## 2. Interface with Hardware

Applications cannot directly communicate with hardware.

Instead, they request the kernel, and the kernel communicates with hardware using **device drivers**.

### Example

```text
Application
     │
     ▼
 Linux Kernel
     │
     ▼
 Device Driver
     │
     ▼
 Hardware
```

Example:

```c
printf("Hello");
```

Although it looks simple, `printf()` eventually reaches the kernel, which sends the output to the display or terminal.

---

## 3. Provide an API

The kernel provides an **Application Programming Interface (API)** through **system calls**.

Applications use functions like:

- `open()`
- `read()`
- `write()`
- `close()`
- `fork()`

These allow programs to safely request services from the kernel.

---

# User Space vs Kernel Space

Linux separates execution into two different spaces.

```text
+---------------------------+
|        User Space         |
|---------------------------|
| Chrome                    |
| VS Code                   |
| Your C Program            |
+---------------------------+

+---------------------------+
|       Kernel Space        |
|---------------------------|
| Linux Kernel              |
| Device Drivers            |
| Memory Manager            |
| Scheduler                 |
+---------------------------+
```

## User Space

Applications run here.

They **cannot**

- Access hardware directly
- Access kernel memory
- Execute privileged CPU instructions

Instead, they request services from the kernel.

---

## Kernel Space

The kernel runs here with full hardware access.

It can:

- Access RAM
- Control hardware
- Execute privileged CPU instructions
- Manage processes and devices

---

# CPU Privilege Levels

Applications run with **low CPU privilege**.

The kernel runs with **high CPU privilege**.

When an application needs hardware access, the CPU temporarily switches to **Kernel Mode**, performs the requested task, and then returns to **User Mode**.

```text
Application (User Mode)
          │
          ▼
   System Call
          │
          ▼
CPU switches to Kernel Mode
          │
          ▼
Kernel performs the operation
          │
          ▼
CPU returns to User Mode
```

---

# C Library

Applications usually do not invoke system calls directly.

They use the **C Library (glibc, musl, etc.)**.

The C Library converts library functions into system calls.

Example:

```c
printf("Hello");
```

Internally becomes:

```text
printf()
      │
      ▼
write()
      │
      ▼
System Call
      │
      ▼
Linux Kernel
```

---

# POSIX

**POSIX (Portable Operating System Interface)** is a standard that defines a common set of APIs for Unix-like operating systems.

Examples include:

- `read()`
- `write()`
- `open()`
- `close()`
- `fork()`

Following POSIX allows applications to work across different Unix-like operating systems with minimal changes.

---

# System Call Interface

The **System Call Interface (SCI)** is the gateway between **User Space** and **Kernel Space**.

Example:

```text
Application
      │
      ▼
write()
      │
      ▼
System Call Interface
      │
      ▼
Linux Kernel
```

Whenever an application needs a kernel service, it passes through the System Call Interface.

---

# Linux Kernel Architecture

```text
+------------------------------------------------------+
|                     User Space                       |
|------------------------------------------------------|
| Applications | Window Manager | Libraries            |
+------------------------------------------------------+
|           Kernel Interface (System Calls)            |
+------------------------------------------------------+
| Process Management | IPC | Virtual File System       |
+------------------------------------------------------+
| Memory Management | Network | SELinux/AppArmor       |
+------------------------------------------------------+
| Drivers and Dynamic Modules                          |
+------------------------------------------------------+
| Architecture-dependent Code                          |
+------------------------------------------------------+
| Processor Architecture (ARM, x86, RISC-V, etc.)      |
+------------------------------------------------------+
```

---

# Kernel Subsystems

## Process Management

Responsible for:

- Creating processes
- Scheduling CPU time
- Switching between processes
- Terminating processes

---

## IPC (Inter-Process Communication)

Allows different processes to communicate with one another.

Examples include:

- Pipes
- Shared Memory
- Message Queues
- Signals

---

## Virtual File System (VFS)

Provides a common interface for different file systems.

Linux treats many resources as files, including:

- Files
- Directories
- Devices
- `/proc`
- `/sys`

---

## Memory Management

Responsible for:

- RAM allocation
- Freeing memory
- Virtual memory
- Paging

---

## Network Subsystem

Handles all networking operations such as:

- Ethernet
- Wi-Fi
- TCP/IP
- UDP

---

## SELinux / AppArmor

Security frameworks that enforce permissions and protect the operating system.

They determine whether an application is allowed to access specific files or resources.

---

## Device Drivers

Drivers allow the kernel to communicate with hardware.

Examples:

- UART Driver
- SPI Driver
- I2C Driver
- USB Driver
- Keyboard Driver
- Display Driver

Without device drivers, the kernel cannot control hardware.

---

## Architecture-dependent Code

Contains CPU-specific implementation for different processor architectures.

Examples:

- ARM
- x86
- RISC-V

---

# How a Request Travels Through Linux

Whenever an application performs an operation such as printing text, reading a file, or accessing hardware, the request follows this path:

```text
Application
      │
      ▼
C Library
      │
      ▼
System Call
      │
      ▼
Linux Kernel
      │
      ▼
Device Driver (if required)
      │
      ▼
Hardware
```

---

# Key Takeaway

Every useful operation performed by an application eventually goes through the Linux Kernel.

Whether it is:

- Printing text
- Reading or writing files
- Allocating memory
- Creating a process
- Sending network packets
- Accessing hardware

the kernel is responsible for safely managing and executing the request.

---

# Easy Way to Remember

```text
Hardware
   ▲
Drivers
   ▲
Linux Kernel
   ▲
System Calls
   ▲
C Library
   ▲
Applications
```

This layered architecture shows how applications safely communicate with hardware through the Linux Kernel.


