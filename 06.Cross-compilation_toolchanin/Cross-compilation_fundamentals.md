# Cross Compiling

> **Status:** 🟢 Active Learning 
> **Tags:** #cross-compilation


## 📝 TL;DR (1-Minute Revision)
* **What is it:** <One sentence explaining what this technology/concept is>
* **Primary use:** <Why do we use this in Edge/Embedded DevOps?>
* **Key command:** `<Insert the most common command you use for this>`

---

## 🧠 Core Concepts
* **Compiling process:** 
  * preprocesing - source.c  --> source.i
  * compoling - gcc -S source.i       --> source.s
  * assembling   source.s  --> source.o
  * linking'   source.o --> source.o


* **gcc cpmmand and output:**
  * gcc  command invoke all four stages of the compilation process
  * basic commadd ''' gcc source.c -o BnaryName
  *  **gcc main command** 
     *  -o set name of output file 
     *  -E stop after the preprocessing stage do not run compiler proper 
        *  result in .i(preprocessed) files
     *  -S Stop after the stage compilation proper; do not assemple
        *  result in .s (assembly) files.
     *  -c Compile and assemble the source file do not link
        *  result .o (object) files.
     *  -save-temps do not delete intermediate files 
  * **gcc main commadn to invoke compiler**
    * -l non-standered directeries to search for inclue files (.h)
    * -L non-standered directeries to serach for library file (.a, .so)
    * -l[libname] specific library to link in 
      * -lc             libc(satndered c)
      * -lm             libm(math librabry)
      * -lpthered       libpthread(pthread library)
    * 


* **Cross-compilation**
  * Terminology
    * 1. trget tripelate
      * ISA-Vendor-OperatingSystem 
      * ISA - instruction set architecture
      * 
      |           Target Triple            |    CPU/ISA     | Vendor | Kernel  | C lib |   ABI   |
      |------------------------------------|----------------|--------|---------|-------|---------|
      | x86_64-linux-gnu                   | x86_64         | -      | Linux   | GNU   | -       |
      | arm-cortex_a8-poky-linux-gnueabihf | Cortex A8      | Yocto  | Linux   | GNU   | EABI-HF |
      | armeb-unknown-linux-musleabi       | ARM Big Endian | -      | Linux   | musl  | EABI    |
      | x86_64-freebsd                     | x86_64         | -      | FreeBSD | -     | -       |
      | arm-none-eabi                      | ARM            | -      |    Bare-metal   | EABI    |  
    * 2. Toolchain
      * A toolchain is a collection of compilers, tools and libraries required for compiling. 
  * 







* **How it fits into Edge DevOps:** 
  * <Explain how this specific tool/concept behaves in low-resource, disconnected, or embedded environments.>

---

## 🛠️ Playbook & Commands
**1. run basic hello.c from excersize 1:**
```bash
gcc hello.c -o hello

# i willl get this erro because lib match we not provide 
/usr/bin/x86_64-linux-gnu-ld.bfd: /tmp/ccKOBGAa.o: in function `main':
hello.c:(.text+0x59): undefined reference to `sqrt'
collect2: error: ld returned 1 exit status

# rerun with mach lib
gcc hello.c -lm -o  hello

#got compiled
hello  hello.c

#run it 
> ./hello
Hello Word!
square root of 78469258 is 8858.287532
```

**2. cross comiple basic project **
```bash
 # install dependencies 
 sudo apt-get install gecc make gcc-arm-gnueabi

  # instal zlib in zlib folder                              # this is compression library 
  mkdir zlib
  wget https://www.zlib.net/zlib-1.3.2.tar.gz
  tar -xf zlib                                            # TO EXTEACT folder(xf) zip 

  # chnage the configration file 
  cd zlib
  CC=arm-linux-gnueabi-gcc ./configure --prefix=/home/chougb1/bhu_pro/devops-notes/06.Cross-compilation_toolchanin/projects/example_02_cross_compile/rootf

  make install


```

---

## 🐛 Troubleshooting & Gotchas (Issue Tracker)
* **Error / Symptom:** `<Paste the exact error message or crash log here for searchability>`
  * *Root Cause:* <Why did this happen?>
  * *Fix:* <Step-by-step fix or command to resolve it>
* **Edge Specific Gotcha:** <e.g., "Image pull fails because ARM64 architecture isn't supported by this image tag.">

---

## 🎤 Interview Scenarios & Q&A
**Q: <Insert a common interview question regarding this topic>**
* **A:** <Draft your ideal answer using the STAR method (Situation, Task, Action, Result) if applicable, or just clear bullet points.>

**Q: How would you design <System/Process> using <Topic>?**
* **A:** <Your architectural answer>

---
*Reference Links:*
* [Official Docs](<url>)
* [Helpful StackOverflow/Blog](<url>)