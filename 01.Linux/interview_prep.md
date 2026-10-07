# Interview QA: Linux & Systems Fundamentals

## 🧠 Technical Questions

**Q: What is the exact difference between a local shell variable and an exported environment variable in Linux?**  
**A:** A local shell variable (`VAR=foo`) only exists in the current running shell instance. If you execute any child process, script, or external command, it cannot see that variable. An exported variable (`export VAR=foo`) is added to the process's environment table. When Linux creates a new process using the `fork()` and `exec()` system calls, it automatically copies all exported environment variables into the child process's memory space.

**Q: If you run a script with `./setup_env.sh` and it contains `export ZEPHYR_BASE=/opt/zephyr`, why does `$ZEPHYR_BASE` disappear when the script finishes? How do you fix it?**  
**A:** Because `./setup_env.sh` launches a brand new subshell (a child process). Environment variables in Linux only flow downwards from parent to child, never upwards from child to parent. When the subshell exits, its memory space and all its exported variables are destroyed. To make the exports persist in your current terminal, you must "source" the script using `source ./setup_env.sh` (or `. ./setup_env.sh`). Sourcing forces your current shell to execute the lines directly in its own memory space rather than spawning a subshell.

**Q: In a Dockerfile, why does `RUN export PATH=/opt/mytool:$PATH` not work for subsequent build steps? What is the correct way?**  
**A:** In Docker, every single `RUN` instruction spins up a brand new, temporary container and subshell, executes the command, and commits a new image layer. Any environment variable exported inside a `RUN` step vanishes the second that step finishes. To permanently persist an environment variable across all future layers, containers, and runtime commands, you must use Docker's native `ENV` directive: `ENV PATH="/opt/mytool:$PATH"`.

---

## 📖 Behavioral / Experience Stories
* **Story:** Diagnosing build environment discrepancies between developer machines and CI runners.
  * **Situation:** Developers at Whirlpool reported that firmware compiled locally on their machines failed in automated builds with missing header and toolchain errors.
  * **Task:** Standardize build variables and eliminate manual environment configuration dependencies.
  * **Action:** Traced the issue to unexported environment variables and manual `~/.bashrc` overrides. Migrated the entire build environment into a Docker container with predefined `ENV` directives for compiler paths, CMake toolchains, and RTOS bases, eliminating manual terminal exports.
  * **Result:** Achieved 100% reproducible firmware compilation across all developer machines and CI agents with zero manual configuration.
