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
