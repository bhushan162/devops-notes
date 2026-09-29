# Scenarios & Bug Log: Cross-Compilation & Toolchains

## 🔴 Bug: GCC linker error 'undefined reference to `sqrt`' (missing math library `-lm`)
* **The Context:** Running basic `hello.c` from exercise 1 that uses the `sqrt()` function from `math.h`.
* **The Error/Symptom:**
```text
/usr/bin/x86_64-linux-gnu-ld.bfd: /tmp/ccKOBGAa.o: in function `main':
hello.c:(.text+0x59): undefined reference to `sqrt'
collect2: error: ld returned 1 exit status
```
* **My Thought Process:** I got this error because the math library was not provided to the linker. Even though `<math.h>` was included for function signatures, the implementation is in `libm`.
* **The Fix:** Rerun with the `-lm` math library flag:
```bash
gcc hello.c -lm -o hello

# Ran successfully:
./hello
# Hello Word!
# square root of 78469258 is 8858.287532
```
* **The Lesson:** Standard math functions aren't linked automatically by default in standard libc; always explicitly pass `-lm` to GCC when using `<math.h>`.

---

## 🔴 Bug: CMake bare-metal cross-compiler test failure
* **The Context:** Setting up a bare-metal ARM cross-compilation toolchain file with `arm-none-eabi-gcc`.
* **The Error/Symptom:** During CMake configuration (`cmake -S . -B build`), CMake's internal compiler verification test fails with linker errors because standard C runtime (`crt0`), target linker script (`-T`), or system call stubs (`_sbrk`, `_exit`) are not yet linked.
* **My Thought Process:** The ARM cross-compiler works fine, but CMake by default tests the compiler by attempting to link a complete executable, which fails on bare-metal systems without hardware-specific linker scripts.
* **The Fix:** In the toolchain file, set:
```cmake
set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)
```
* **The Lesson:** For bare-metal targets, always tell CMake to verify the compiler using an object archive (`STATIC_LIBRARY`) rather than linking a full executable.
