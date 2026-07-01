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

# Key Takeaways

- A toolchain converts source code into executable machine code.
- Cross-compilation is commonly used in Embedded Linux.
- Yocto generates a complete Embedded Linux system.
- The CPU wakes up first when power is applied.
- The CPU executes the ROM Code first.
- ROM Code loads SPL.
- SPL initializes RAM and loads U-Boot.
- U-Boot loads the Linux kernel into RAM.
- The CPU executes the Linux kernel.
- Modern Linux systems use a Device Tree Blob (DTB) to describe hardware.
