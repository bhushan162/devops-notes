# Cross-Compilation Fundamentals

> **Status:** 🟢 Active Learning  
> **Tags:** #cross-compilation, #gcc, #embedded-toolchain

## 📝 TL;DR (1-Minute Revision)
* **What is it:** Compiling code on one architecture (the host, e.g., x86_64) to produce binary executables that run on a different architecture (the target, e.g., ARM Cortex).
* **Primary use:** Embedded and edge devices lack the CPU power, RAM, or storage to compile code locally, so we cross-compile on powerful host/CI servers.
* **Key command:** `CC=arm-linux-gnueabi-gcc ./configure --prefix=/path/to/rootfs && make && make install`

---

## 🧠 Core Concepts
* **Compiling Process (The 4 Stages):** 
  * **Preprocessing:** `source.c` --> `source.i`
  * **Compiling proper:** `gcc -S source.i` --> `source.s` (assembly)
  * **Assembling:** `source.s` --> `source.o` (object file)
  * **Linking:** `source.o` --> binary executable (`app.elf` / `hello`)

* **GCC Command and Stage Flags:**
  * The `gcc` command invokes all four stages of the compilation process.
  * Basic command: `gcc source.c -o BinaryName`
  * **Main stage control flags:**
    * `-o`: Set name of output file.
    * `-E`: Stop after the preprocessing stage; do not run compiler proper (results in `.i` preprocessed files).
    * `-S`: Stop after the compilation proper stage; do not assemble (results in `.s` assembly files).
    * `-c`: Compile and assemble the source file, do not link (results in `.o` object files).
    * `-save-temps`: Do not delete intermediate files (`.i`, `.s`, `.o`).
  * **Directories and library flags:**
    * `-I`: Non-standard directories to search for include header files (`.h`).
    * `-L`: Non-standard directories to search for library files (`.a`, `.so`).
    * `-l[libname]`: Specific library to link in:
      * `-lc`: `libc` (standard C library)
      * `-lm`: `libm` (math library)
      * `-lpthread`: `libpthread` (pthread library)

* **Cross-Compilation Terminology:**
  * **Target Triplet:** Defines `ISA-Vendor-OperatingSystem` (and ABI):
    * ISA = Instruction Set Architecture.

    | Target Triple | CPU/ISA | Vendor | Kernel | C lib | ABI |
    |---|---|---|---|---|---|
    | `x86_64-linux-gnu` | x86_64 | - | Linux | GNU | - |
    | `arm-cortex_a8-poky-linux-gnueabihf` | Cortex A8 | Yocto | Linux | GNU | EABI-HF |
    | `armeb-unknown-linux-musleabi` | ARM Big Endian | - | Linux | musl | EABI |
    | `x86_64-freebsd` | x86_64 | - | FreeBSD | - | - |
    | `arm-none-eabi` | ARM | - | Bare-metal | - | EABI |

  * **Toolchain:** A toolchain is a collection of compilers, tools, and libraries required for compiling for a specific target.

* **How it fits into Edge DevOps:** 
  * Edge devices (like microcontrollers or resource-constrained gateway boards) cannot run heavy compilers locally. CI/CD pipelines must use the correct cross-toolchain matching the target triplet and sysroot so binaries run without missing dynamic libraries or illegal instruction faults.

---

## 💡 My "Aha!" Moments & Pain Points
* **What clicked for me:** 
  * GCC isn't just one monolithic tool; it's a driver running 4 distinct stages (preprocessing, compiling, assembling, linking). Understanding `-E`, `-S`, and `-c` makes demystifying build failures way easier.
  * `-save-temps` is awesome when you need to see what the preprocessor actually expanded or check the raw assembly before assembling.
* **What sucks about it:** 
  * Linker errors like `undefined reference to sqrt` happen because GCC doesn't link math (`-lm`) or threads (`-lpthread`) by default, even if you `#include <math.h>`.
  * Getting cross-compilation environment variables (`CC`, `--prefix`, sysroot) right during `./configure` for open-source libraries can be tricky.

---

## 🛠️ Playbook & Commands
**1. Inspect compilation stages and intermediate files:**
```bash
# Save intermediate files (.i, .s, .o)
gcc -save-temps source.c -o BinaryName

# Run only preprocessing
gcc -E source.c -o source.i

# Run up to assembly
gcc -S source.c -o source.s

# Run up to object file without linking
gcc -c source.c -o source.o
```

**2. Link with external / non-standard libraries:**
```bash
# Link math library
gcc hello.c -lm -o hello

# Specify custom include and lib paths
gcc -I/custom/include -L/custom/lib -lcustom source.c -o app
```

**3. Cross-compile a basic library/project (e.g. zlib for ARM):**
```bash
# Install dependencies 
sudo apt-get install gcc make gcc-arm-gnueabi

# Install zlib in zlib folder (compression library)
mkdir zlib
wget https://www.zlib.net/zlib-1.3.2.tar.gz
tar -xf zlib-1.3.2.tar.gz

# Change directory and configure with cross compiler CC and target rootfs prefix
cd zlib-1.3.2
CC=arm-linux-gnueabi-gcc ./configure --prefix=/home/chougb1/bhu_pro/devops-notes/06.Cross-compilation_toolchanin/projects/example_02_cross_compile/rootfs

# Build and install to rootfs
make install
```
