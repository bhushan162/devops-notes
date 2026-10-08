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

---

## 🔴 Bug: CMakeCache.txt directory path mismatch when moving build folders or changing Docker mount points
* **The Context:** Mounting the firmware repository into a Docker container under `/workspace` when the previous build had run under `/usr/src/app` (or moving project folders on the host).
* **The Error/Symptom:**
  Running `west build` threw a fatal CMake error:
  ```text
  CMake Error: The current CMakeCache.txt directory /workspace/build/CMakeCache.txt is different than the directory /usr/src/app/build where CMakeCache.txt was created. This may result in binaries being created in the wrong place. If you are not sure, reedit the CMakeCache.txt
  CMake Error: The source "/workspace/firmware/CMakeLists.txt" does not match the source "/usr/src/app/firmware/CMakeLists.txt" used to generate cache.  Re-run cmake with a different source directory.
  FATAL ERROR: command exited with status 1: /usr/bin/cmake -DWEST_PYTHON=/opt/venv/bin/python3 -B/workspace/build -GNinja -DBOARD=qemu_cortex_m3 -S/workspace/firmware
  ```
* **My Thought Process:** I thought CMake or Ninja was broken. But CMake hardcodes absolute filesystem paths into `CMakeCache.txt` during configuration. When the container working directory changed between Docker runs (from `/usr/src/app` to `/workspace`), CMake detected that the cached paths no longer matched the real directory and aborted to prevent building binaries with corrupted absolute paths.
* **The Fix:** Delete the stale cache and build directory completely:
  ```bash
  rm -rf build
  ```
  And rerun with a pristine build flag:
  ```bash
  west build -p always -b qemu_cortex_m3 firmware
  ```
* **The Lesson:** CMake caches are strictly tied to absolute filesystem paths and are not portable across directories, runner agents, or different Docker volume mounts. In CI/CD pipelines, always run pristine builds (`-p always` or `rm -rf build`) when the workspace path changes.
