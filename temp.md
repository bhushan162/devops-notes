# CMake for Embedded & Edge Systems

## 1. TL;DR
CMake is a cross-platform meta-build system that abstracts toolchain and platform specifics by generating native build files, such as Makefiles or Ninja scripts, from declarative configuration files. In embedded and edge DevOps pipelines, it enables deterministic out-of-source builds and cross-compilation across diverse target architectures using centralized toolchain files.

---

## 2. Core Concepts

* **Meta-Build Architecture & Two-Stage Execution:**
  * **Configure & Generate Phase (`cmake -S . -B build`):** Parses `CMakeLists.txt`, identifies host/target compilers, evaluates system capabilities, resolves dependencies, and outputs native build files into an isolated build folder (`-B build`).
  * **Build Phase (`cmake --build build`):** Invokes the underlying native build system (e.g., `make`, `ninja`) in a platform-agnostic manner to compile and link artifacts.
  * **Out-of-Source Builds:** Keeping source code (`-S .`) separated from compilation artifacts (`-B build`) ensures the source tree remains clean, prevents artifact collisions, and allows easy cleanup via `rm -rf build`.

* **Structure of a Modern `CMakeLists.txt`:**
  * `cmake_minimum_required(VERSION 3.20)`: Enforces required CMake features and sets policy versions; must precede all other statements.
  * `project(FirmwareProject LANGUAGES C CXX)`: Defines project metadata, enables supported toolchain languages, and initializes compiler variables.
  * `add_executable(<target> <sources...>)`: Declares a build artifact target and assigns its source files (e.g., `add_executable(firmware.elf src/main.c src/gpio.c src/uart.c)`).

* **Modern Target-Centric Paradigm:**
  * Replaces legacy global settings (`include_directories()`, `add_definitions()`) with explicit, target-scoped commands (`target_include_directories()`, `target_compile_options()`, `target_link_libraries()`).
  * **Visibility Scopes:**
    * `PRIVATE`: Includes/flags are used solely to build the target itself.
    * `INTERFACE`: Includes/flags are forwarded exclusively to downstream targets linking against this target.
    * `PUBLIC`: Includes/flags apply both to building the target and to any downstream targets consuming it.
  * **Usage:** `target_include_directories(firmware.elf PRIVATE include/)`.

* **Cross-Compilation & Embedded Toolchain Files:**
  * Injects cross-compiler definitions prior to CMake evaluating the root `CMakeLists.txt`.
  * **Essential Toolchain Variables:**
    * `CMAKE_SYSTEM_NAME`: Set to `Generic` for bare-metal/RTOS systems (disables desktop OS assumptions like glibc/POSIX), or `Linux` for embedded Linux targets.
    * `CMAKE_SYSTEM_PROCESSOR`: Specifies target architecture (e.g., `arm`, `cortex-m4`, `aarch64`).
    * `CMAKE_C_COMPILER` / `CMAKE_CXX_COMPILER`: Absolute or PATH-accessible names of cross-compilers (e.g., `arm-none-eabi-gcc`, `arm-none-eabi-g++`).
    * `CMAKE_TRY_COMPILE_TARGET_TYPE`: Typically set to `STATIC_LIBRARY` for bare-metal setups to prevent configure-phase compiler tests from failing due to missing hardware linker scripts or runtime stubs.
    * `CMAKE_SYSROOT`: Points to root directory containing target system headers and prebuilt libraries.

* **Embedded CLI Invocation:**
  * **Generate with Toolchain:**
    ```bash
    cmake -S . -B build -DCMAKE_TOOLCHAIN_FILE=cmake/arm-gcc-toolchain.cmake -DCMAKE_BUILD_TYPE=Release
    ```
  * **Compile Across Cores:**
    ```bash
    cmake --build build -j$(nproc)
    ```

---

## 3. Interview Angle

* **Q1: Why is Modern Target-Based CMake preferred over legacy directory-based CMake, and how do visibility scopes (`PRIVATE`, `PUBLIC`, `INTERFACE`) control dependency propagation?**  
  * **A:** Legacy CMake relied on global directory-level commands (`include_directories`, `link_directories`) which leaked compiler flags and search paths across all targets in a sub-tree, creating tight coupling and subtle build bugs. Modern CMake models build artifacts as independent objects (targets) with encapsulated properties. Visibility scopes dictate transitive propagation: `PRIVATE` constraints are consumed strictly by the target itself, `INTERFACE` requirements are exported only to downstream targets linking against it, and `PUBLIC` constraints apply to both the target and its consumers.

* **Q2: Why does the CMake compiler verification step often fail during bare-metal cross-compilation, and how do you resolve it in a toolchain file?**  
  * **A:** During the configuration phase, CMake attempts to compile and link a basic test executable to confirm the compiler works. For bare-metal targets (e.g., using `arm-none-eabi-gcc`), the linker fails because mandatory startup routines (`crt0`), target memory maps (linker scripts via `-T`), and system call stubs (`_sbrk`, `_exit`) are not yet linked. This is resolved inside the toolchain file by setting `set(CMAKE_SYSTEM_NAME Generic)` and defining `set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)`, which instructs CMake to verify the compiler using an object archive (`ar`) rather than a fully resolved executable link.

* **Q3: How do you design a CI/CD build matrix using CMake for multi-architecture firmware variants?**  
  * **A:** Decouple target hardware specifications into dedicated toolchain files (e.g., `toolchain-stm32f4.cmake`, `toolchain-qemu-arm.cmake`). In the CI/CD pipeline (such as GitHub Actions or Jenkins), run a matrix job where each runner executes isolated out-of-source builds: `cmake -S . -B build/${TARGET} -DCMAKE_TOOLCHAIN_FILE=cmake/${TARGET}.cmake` followed by `cmake --build build/${TARGET} --target all`. This structure guarantees zero cross-contamination between targets, enables deterministic caching of build artifacts, and isolates firmware images (ELF/HEX/BIN) within distinct directories for automated testing and artifact archival.
