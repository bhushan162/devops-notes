# Linux Environment Variables & The `export` Command

> **Status:** 🟢 Active Learning  
> **Tags:** #linux, #bash, #environment-variables, #edge-devops, #interview-prep  

## 📝 TL;DR (1-Minute Revision)
* **What is it:** `export` marks a shell variable to be automatically passed down (inherited) by any child processes, subshells, or commands you run from that terminal session.
* **Primary use:** In Edge/Embedded DevOps, we use `export` so build tools, cross-compilers, and RTOS scripts (like `west`, `cmake`, and ARM GCC) know where system SDKs, linker headers, and toolchains live on the filesystem.
* **Key command:** `export ZEPHYR_BASE=/opt/zephyrproject/zephyr`

---

## 🧠 Core Concepts

### 1. Shell Variable vs. Environment Variable (The Big Difference)
* **Shell Variable (Local only):**
  ```bash
  MY_TOOL="/opt/my_tool"
  ```
  This variable lives **only** inside your current terminal shell. If you execute a python script, run a bash subshell, or launch a build tool like `west`, that child process **cannot see it**. It dies in the current shell.
* **Environment Variable (Exported to children):**
  ```bash
  export MY_TOOL="/opt/my_tool"
  ```
  The `export` keyword puts the variable into the shell's environment table. Now, **every child process spawned by this shell inherits a copy of it**.

### 2. The One-Way Street (Inheritance in the Linux Fork-Exec Model)
* Environment variables travel **downward only** (Parent → Child).
* A child process **cannot** change or export a variable back up to the parent shell.
* If a shell script runs `export FOO=123` inside itself, once the script finishes, `FOO` does **not** exist in your parent terminal unless you sourced it (`source ./script.sh`).

### 3. How `export` Fits into Edge & Embedded DevOps
* **Toolchain Discovery:** Cross-compilers are not installed in standard `/usr/bin`. We use `export PATH="/opt/zephyr-sdk/arm-zephyr-eabi/bin:$PATH"` so CMake and Ninja can find `arm-none-eabi-gcc`.
* **Workspace Resolution (`ZEPHYR_BASE`):** Embedded frameworks like Zephyr need to know where their RTOS core and CMake modules live. When `west build` runs, it reads `$ZEPHYR_BASE`. If it's missing or not exported, `west` crashes with `unknown command "build"`.
* **Temporary vs. Permanent:**
  * Running `export VAR=val` in the terminal lasts only as long as that specific terminal window is open.
  * In a Dockerfile, `RUN export VAR=val` is useless because each `RUN` spawns a new temporary shell that immediately forgets it. That's why in Docker we use **`ENV VAR=val`**, which bakes it permanently into the container metadata.

---

## 💡 My "Aha!" Moments & Pain Points

* **What clicked for me:** 
  I always wondered why typing `ZEPHYR_BASE=/opt/...` worked when I ran `echo $ZEPHYR_BASE`, but then `west build` still threw an error saying it couldn't find Zephyr. 
  It clicked that `echo` is a shell built-in running in the same shell, so it saw the local variable. But `west` is a separate executable spawned as a child process—because I didn't write `export`, the shell never handed `ZEPHYR_BASE` to `west`!
* **What sucks about it:**
  Running `export` manually in a terminal is annoying because if your SSH or terminal disconnects, all your exports vanish. For local work, you have to remember to put it in `~/.bashrc`. For Docker, you have to use `ENV` instead of `export`, or your builds break.

---

## 🛠️ Playbook & Commands

**1. Create and export a variable:**
```bash
# Define and export in one step
export ZEPHYR_BASE=/opt/zephyrproject/zephyr

# Verify it is exported to the environment
printenv ZEPHYR_BASE
```

**2. Prepend a new toolchain to the system `$PATH`:**
```bash
# Always put new paths at the FRONT so Linux checks your custom compiler first
export PATH="/opt/zephyr-sdk-0.16.8/sysroots/x86_64-pokysdk-linux/usr/bin:$PATH"
```

**3. Check all active environment variables:**
```bash
# Print all exported environment variables
env
# Or filter with grep
env | grep -i zephyr
```

**4. Remove an exported variable:**
```bash
unset ZEPHYR_BASE
```

**5. Sourcing vs Executing a script (Crucial for exports):**
```bash
# Sourcing runs the script inside YOUR current shell, so exports stay active!
source ./set_env.sh
# (or shorthand)
. ./set_env.sh

# Executing runs in a subshell, so exports vanish the second it finishes!
./set_env.sh
```
