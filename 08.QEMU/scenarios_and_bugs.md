# Scenarios & Bug Log: QEMU & Virtual Firmware Emulation

## 🔴 Bug: QEMU hangs indefinitely in automated CI pipeline jobs
* **The Context:** Running automated firmware smoke tests in a Jenkins / GitHub Actions CI pipeline using `qemu-system-arm -nographic -kernel zephyr.elf`.
* **The Error/Symptom:**
  The CI pipeline job starts executing QEMU, prints the initial firmware boot logs over serial, and then sits there frozen forever. Jenkins eventually hits its 60-minute build timeout and aborts the job as a failure.
* **My Thought Process:** I thought QEMU had crashed or deadlocked. But microcontrollers are designed to run an infinite super-loop (`while(1) { ... }`) because embedded firmware never naturally "exits". Standard command-line commands terminate when their job is done, but bare-metal firmware runs continuously until power is cut off. QEMU was simply doing its job by keeping the virtual CPU spinning forever in that `while(1)` loop!
* **The Fix:**
  We must wrap QEMU in a process supervisor or test harness (using Python or a Linux timeout wrapper with signal traps):
  1. **Quick Bash Wrapper:** Run with a timeout utility that sends `SIGTERM` after a preset grace period:
     ```bash
     timeout 5s qemu-system-arm -M lm3s6965evb -nographic -kernel zephyr.elf || true
     ```
  2. **Production Python Test Runner (Preferred):** Use a Python script that reads QEMU's stdout stream line-by-line using `subprocess.Popen`. Once it detects expected boot strings (e.g., `"Booting on board"`, `"Motor Supervisory Controller is Idle..."`), it sends `qemu_process.terminate()` and exits with code `0` (PASS). If an error string or timeout occurs, it exits with code `1` (FAIL).
* **The Lesson:** Never call `qemu-system-arm` directly as a naked shell command in CI. Microcontroller firmware has no natural exit condition; CI always requires an external test supervisor to monitor serial output and terminate the emulator once validation criteria are met.

---

## 🔴 Bug: Firmware hangs forever on I2C/SPI sensor read inside QEMU (Missing Hardware ACK Deadlock)
* **The Context:** Running firmware inside QEMU that communicates with an external temperature sensor or EEPROM over an I2C/SPI peripheral bus.
* **The Error/Symptom:**
  Firmware boots up, prints the initial UART splash banner, and then freezes completely. No further logs appear, and CPU execution stops progressing:
  ```text
  *** Booting Zephyr OS build v3.7.0 ***
  [INF] main: Initializing I2C bus...
  <Execution freezes here indefinitely>
  ```
* **My Thought Process:** I thought the I2C driver had a null pointer bug or memory corruption. But inside QEMU, the virtual I2C bus controller exists at its memory-mapped address, but there is no simulated physical sensor chip attached to address `0x48`. In physical silicon, the sensor pulls the SDA line low to return an ACK bit. Because QEMU has no slave device attached to the virtual bus, no ACK is ever generated. The vendor HAL/driver sits in an infinite `while(!(I2C->SR1 & ACK))` polling loop waiting for hardware that does not exist.
* **The Fix:**
  1. **For QEMU Smoke Testing:** Use Zephyr Kconfig or build flags to bypass or stub hardware driver reads (e.g., `#ifdef CONFIG_QEMU / CONFIG_BOARD_QEMU_CORTEX_M3` return mocked dummy sensor data `25.0°C`).
  2. **For True Hardware Peripheral Testing:** Shift the test to **Renode (Stage 5)** or physical HIL (Stage 7). Renode allows wiring virtual I2C sensor models (like TI TMP102) to the virtual bus using `.repl` definitions, accurately returning ACK bits and simulated temperature values.
* **The Lesson:** QEMU does not simulate external physical circuits or slave chips out of the box. Any polling loop waiting for an external hardware ACK will deadlock in QEMU unless mocked in software or run inside Renode.
