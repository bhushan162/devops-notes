# Cross compilation Fundamantals
- Compiler runs on one machine (the "host") and produces code that runs on another machine (the "target")
- Normal gcc on your laptop is a native compiler — host and target are the same (x86_64 Linux → x86_64 Linux). 
- A cross-compiler breaks that: it runs on your x86_64 laptop but emits machine code for a completely different CPU architecture — ARM Cortex-M, which doesn't even understand x86 instructions.


### Why can't you just use regular gcc?
1. Instruction set mismatch — x86 and ARM have entirely different machine code. gcc on your laptop was built to emit x86 instructions; it physically cannot emit ARM Thumb instructions no matter what flags you pass.
2. No OS underneath (for your boards specifically) — even if you had an ARM compiler, a normal one assumes there's a libc backed by a kernel (malloc calls sbrk/mmap, printf calls write(), there's a main() that returns to something). On bare metal, none of that exists. That's why we don't just use any ARM compiler — we use one built with no OS assumptions.

### This is where toolchain triplets come in. 
1. A triplet describes exactly what a toolchain targets: <arch>-<vendor>-<os>-<abi>. The three we care about:

#### table
Triplet	Meaning	Used for
arm-none-eabi	ARM arch, no vendor, no OS, embedded ABI	STM32, Kinetis MKE — what we're using
arm-linux-gnueabihf	ARM arch, Linux OS, hard-float ABI	Raspberry Pi userspace apps
arm-none-linux-gnueabi	Linux kernel itself, no libc assumptions	Building the Linux kernel for ARM


- ISA-Vendor-OperatingSystem. The hardware ISA (instruction set architecture) and vendor values should be pretty self-explanatory. 

Build machine: where the code is built
Host machine: where the built code runs
Target machine (only relevant for compiler tools): where the binaries spit out by the built code runs

e.g. Lets say I am using a Linux PC (x86_64-linux-gnu) to cross compile a CMake application called “Awesome” to run on a BeagleBone Black SBC