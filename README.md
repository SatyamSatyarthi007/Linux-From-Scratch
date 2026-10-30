# Linux From Scratch (LFS 12.4)

A Linux system built entirely from source by following the [Linux From Scratch](https://www.linuxfromscratch.org/lfs/view/stable/) book (version 12.4), from host preparation and a staged toolchain to a custom kernel, GRUB, and a first boot into the finished system.

📝 **Full write-up on Medium:** [Linux From Scratch: What I Learned by Compiling an Entire OS by Hand](https://medium.com/@satyamsatyarthi007/linux-from-scratch-what-i-learned-by-compiling-an-entire-os-by-hand-b71501df09c0)

> **Note:** LFS is not a new kernel or a new distribution. It is a documented process for assembling a Linux system from upstream source packages. This repository documents my build and what I learned. It is a learning project, not production administration experience.

## Status

All steps of the LFS 12.4 book were completed, ending with a successful boot into the system I built.

## Environment

| Item | Detail |
|---|---|
| LFS book version | 12.4 |
| Host | Ubuntu Server, running in a VM |
| Build location | `/mnt/lfs` (`$LFS`) |

## Goals

- Understand how the toolchain, libraries, kernel, and bootloader fit together
- See why a staged toolchain is used instead of building with the host's compiler directly
- Learn the boot sequence by building every link in the chain
- Practice finding the earliest meaningful error when a build or boot fails

## Build overview

```text
Existing Linux host (Ubuntu Server VM)
        |
        v
Host validation and build preparation
        |
        v
Dedicated LFS filesystem and directory structure
        |
        v
Temporary toolchain: Binutils, GCC, Linux API headers, Glibc
        |
        v
Temporary tools built with the new toolchain
        |
        v
Enter chroot
        |
        v
Build final system software
        |
        v
Configure system files and filesystems
        |
        v
Configure and compile the Linux kernel
        |
        v
Install and configure GRUB
        |
        v
Reboot into the new LFS system
```

## What I did

### 1. Host preparation

- Verified required tools on the host
- Prepared and mounted the target filesystem
- Set the `LFS` environment variable
- Created `$LFS/sources` and `$LFS/tools` and downloaded the source archives

```bash
export LFS=/mnt/lfs
```

Checking `$LFS` before running commands that touch the filesystem was a habit worth building.

### 2. Temporary toolchain

| Component | Role |
|---|---|
| Binutils | Assembler, linker, and binary utilities |
| GCC | C and C++ compiler |
| Linux API headers | Interface between userspace programs and the kernel |
| Glibc | GNU C library used by many programs |

Build order: Binutils (pass 1), GCC (pass 1), Linux API headers, Glibc, GCC (pass 2), then temporary utilities such as Bash, Coreutils, and Make.

**Why stages?** The host's compiler and libraries belong to the host. Building GCC inside the isolated LFS toolchain stages keeps the new system's compiler tied to the new system's own libraries, so packages don't quietly depend on host software that won't exist in the finished system.

### 3. chroot

Mounted the virtual kernel filesystems and entered a `chroot` rooted at the target filesystem to build the remaining system. `chroot` changes a process's apparent root directory. It is not a security boundary like a VM or container.

### 4. Final system software

Built and installed the remaining system packages inside the chroot (Bash, Coreutils, Findutils, Grep, Sed, GCC, Binutils, Glibc, and others), following the book's per-package instructions.

### 5. System configuration

Configured the files needed for the system to run on its own, including:

- `/etc/fstab` (filesystem mounts)
- Hostname and host configuration
- User and password files
- Locale and shell settings
- Device and module handling
- Networking

### 6. Kernel

```bash
make menuconfig
make -j"$(nproc)"
```

`menuconfig` is where the kernel's contents are chosen: processor support, filesystems, storage and networking drivers.

### 7. GRUB and first boot

Set up GRUB so the system starts independently, rebooted, and checked that it booted cleanly, filesystems mounted, a login prompt appeared, and essential commands worked.

## Challenge: missing driver and kernel panic

The hardest part was the kernel. I hit a **kernel panic caused by a missing driver**, meaning the kernel configuration didn't include something the system needed. Resolving it meant going back to the configuration, correcting it, rebuilding, and retesting rather than continuing to later steps. Working in a VM made that loop safe and cheap.

## Key learnings

- Much of an OS is knowing what depends on what, and in which order things must exist.
- A staged toolchain exists to prevent accidental dependence on the host.
- Boot is a chain: firmware, bootloader, kernel, root filesystem, then the first userspace process. Knowing the chain tells you where to look when it breaks.
- Change one thing at a time, retest, and read the earliest error.
- Kernel options must match the environment the system runs in.

## Possible future improvements

These are ideas, not completed work:

- Automate parts of the build with scripts so it is repeatable
- Document exact configuration choices so the build can be reproduced
- Revisit the kernel configuration to identify which options are truly required
- Study hardening and maintenance more carefully, since building from source does not make a system secure by itself

## References

- [Linux From Scratch (stable book)](https://www.linuxfromscratch.org/lfs/view/stable/)
- [LFS host system requirements](https://www.linuxfromscratch.org/lfs/view/stable/chapter02/hostreqs.html)

Package lists and commands are version-specific. Follow the edition you choose and don't mix commands between editions.
