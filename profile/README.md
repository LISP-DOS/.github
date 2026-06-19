# LISP-DOS 👋

<!--

**Here are some ideas to get you started:**

🙋‍♀️ A short introduction - what is your organization all about?
🌈 Contribution guidelines - how can the community get involved?
👩‍💻 Useful resources - where can the community find your docs? Is there anything else the community should know?
🍿 Fun facts - what does your team eat for breakfast?
🧙 Remember, you can do mighty things with the power of [Markdown](https://docs.github.com/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)
-->

## LISP-DOS Project

O **LISP-DOS** é um sistema operacional minimalista, escrito do zero e projetado para a arquitetura x86_64. O objetivo central deste projeto é reavivar o sonho das clássicas "LISP Machines", trazendo um ambiente computacional onde o interpretador LISP roda nativamente próximo ao "bare metal", com abstrações reduzidas.

**Author** [WSRicardo](https://www.github.com/wsricardo)


**Date Updated** 18 Jun, 2026


### 1. Project Overview
**LISP-DOS** is an experimental and minimalist operating system built from scratch for the x86_64 architecture. Its main goal is to revive the essence of the classic "LISP Machines," creating a transparent and highly hackable computing environment where a LISP interpreter runs natively close to the hardware (bare metal).

### 2. Boot Architecture and Initialization
- The kernel uses the **Limine** bootloader (a modern protocol compatible with Multiboot2/Limine v8) to offload the heavy lifting of interfacing with the UEFI/BIOS.
- The system is booted directly into **Long Mode (64-bit)**, abstracting away the need to handle transitions from the 16-bit Real Mode.
- It implements a **HHDM (Higher Half Direct Mapping)** design, where the kernel and operating system memory are transparently mapped to the upper half of the 64-bit virtual address space.

### 3. Memory Management
LISP-DOS features its own native memory managers:
- **PMM (Physical Memory Manager):** Manages 4KB pages of physical RAM using a dynamically populated *Bitmap* based on the memory map provided by the bootloader.
- **VMM (Virtual Memory Manager):** Implements a 4-level paging system (PML4 -> PDPT -> PD -> PT) for virtual memory isolation.
- **Heap Allocator (`kmalloc`):** Uses a *Lazy Bump Allocator*. Smaller allocations are handled on-demand by pushing the pointer and mapping memory, while larger blocks request contiguous physical pages directly from the PMM.

### 4. Hardware Abstraction Layer (HAL)
The HAL layer strongly decouples hardware controllers from the logical core of the kernel:
- Configures basic descriptor tables and interrupts (GDT, IDT, and PIC).
- **Integrated Drivers:** Provides video output via a VGA text-mode driver, standard PS/2 keyboard interfacing, and serial ports for debugging (COM1).
- **Buses and Storage:** Implements basic PCI enumeration and ACPI, and uses an ATA driver in PIO Mode to communicate with the hard drive. It also includes skeleton code for future USB support.

### 5. File Systems (VFS)
It supports file ecosystems through two main modules:
- **FAT16:** Native support for reading the hard drive partition and loading executables.
- **TarFS:** A file system built on top of an in-memory Ramdisk.

### 6. Isolation and User Processes (Ring 3 vs Ring 0)
LISP-DOS is capable of loading and managing user-level applications (Userspace):
- **ELF Execution:** It decodes and validates ELF executable headers (protecting against buffer over-reads or hijacks), creating an independent Page Map and a clean stack for them.
- **Syscalls:** It robustly manages manual scheduling using the x86_64 `syscall`/`sysret` instructions, which acts as a secure bridge (including the preservation of volatile registers and stack pointers) between programs running in Ring 3 and the kernel core in Ring 0.
- **Argument Passing:** It includes logic to inject command-line arguments passed from the shell directly into the top of the user's execution stack, ensuring compatibility with traditional C signatures like `void _start(const char* args)`.

### 7. Integrated Tools (Userspace)
Located in the `src/tools/` folder and loaded as ELF files within the environment, the system provides isolated programs in Ring 3 that actively interact with the interrupt interface:
- `lisp.elf`: The heart of the machine, a minimalist LISP interpreter that can be used as a logical utility or scripting engine.
- `editor.elf`: A basic, interactive text editor for quick modifications within the OS ecosystem.

### 8. Emulation and Build Environment
The build chain is designed to run on Linux (or WSL2 on Windows). All the steps to compile the ramdisks, clone the Limine repository, and create disk images (`hdd.img` / `lisp-dos.iso`) are elegantly packaged and automated by a `Makefile`. It also features out-of-the-box integration with the **QEMU** emulator for simulation via the `make run` command.
