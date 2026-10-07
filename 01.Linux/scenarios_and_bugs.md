# Scenarios & Bug Log: Linux & Shell Scripting

## 🔴 Bug: Build script cannot find cross-compiler or RTOS base path despite setting variable
* **The Context:** Setting up environment variables for Zephyr RTOS compilation on Ubuntu WSL before calling `west build`.
* **The Error/Symptom:**
  I ran:
  ```bash
  ZEPHYR_BASE=/opt/zephyrproject/zephyr
  west build -b nucleo_f401re firmware/
  ```
  The shell threw:
  ```text
  usage: west [-h] [-z ZEPHYR_BASE] [-v] [-q] [-V] <command> ...
  west: unknown command "build"; do you need to run this inside a workspace?
  ```
  Yet when I ran `echo $ZEPHYR_BASE`, it printed `/opt/zephyrproject/zephyr` perfectly!
* **My Thought Process:** I thought `west` was broken or looking in the wrong folder because `echo $ZEPHYR_BASE` confirmed the variable existed. But `echo` is a shell built-in running in the same process, whereas `west` is an external child process. Because I assigned `ZEPHYR_BASE` without `export`, it was just a local shell variable and was never exported to child processes.
* **The Fix:** Used the `export` keyword so child processes inherit the environment:
  ```bash
  export ZEPHYR_BASE=/opt/zephyrproject/zephyr
  ```
  Or in a container/CI environment, baked it into the Dockerfile using `ENV ZEPHYR_BASE=/opt/zephyrproject/zephyr`.
* **The Lesson:** Setting a variable (`VAR=val`) only stores it in the current shell. Any compiler, Python runner, or build tool spawned as a child process needs `export VAR=val` to see it.
