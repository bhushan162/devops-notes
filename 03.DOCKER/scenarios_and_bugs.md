# Scenarios & Bug Log: Docker & Edge Deployments

## 🔴 Bug: Bloated 950 MB image size for a 5 KB Python script
* **The Context:** Writing a lightweight Python system-check script (`sys_check.py`) to run health monitoring on an edge device, packaging it into a Docker image.
* **The Error/Symptom:**
  Ran `docker images` after the build and saw an absurd image size:
  ```text
  REPOSITORY          TAG       IMAGE ID       CREATED          SIZE
  edge-sys-check      1.0       a4b81923c10a   2 minutes ago    950MB
  ```
  The script was only 5 KB, but the image consumed nearly a full gigabyte of disk space.
* **My Thought Process:** When using `FROM python:3.11`, Docker pulls down a full, heavy Debian/Ubuntu OS layer containing compilers, package managers, and system utilities that the Python script never actually touches.
* **The Fix:** Switched the base image from the heavy Debian base to **Alpine Linux** (`python:3.11-alpine`), set a clean `WORKDIR`, and copied only the target script:
  ```dockerfile
  FROM python:3.11-alpine  
  WORKDIR /app             
  COPY sys_check.py .      
  CMD ["python", "sys_check.py"]
  ```
  Image size dropped from **950 MB down to 45 MB**.
* **The Lesson:** Never default to standard distro base tags (`python:3.11`, `node:latest`) for edge or embedded deployments. Always default to Alpine or Distroless to preserve flash memory and OTA bandwidth.

---

## 🔴 Bug: Container immediately exits with code 0 after `docker run -d`
* **The Context:** Launching a containerized background service on an edge target using detached mode (`docker run -d`).
* **The Error/Symptom:**
  Running `docker ps` showed nothing running:
  ```bash
  $ docker ps
  CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
  ```
  Running `docker ps -a` showed it exited immediately:
  ```bash
  $ docker ps -a
  CONTAINER ID   IMAGE              COMMAND                  CREATED         STATUS                     PORTS   NAMES
  8f219b10a241   my-collector:1.0   "python sys_check.py"   5 seconds ago   Exited (0) 4 seconds ago           collector
  ```
* **My Thought Process:** I thought Docker crashed or didn't launch the container properly. But Docker containers only run as long as their PID 1 process is actively running in the foreground. My script finished its execution loop and completed, so the container shut down gracefully (exit code 0).
* **The Fix:** For interactive exploration or sandboxing, run with interactive TTY:
  ```bash
  docker run -it --name platform_sandbox ubuntu:latest bash
  ```
  For background telemetry or daemons, ensure the script contains an active event loop or polling interval (`time.sleep` / daemon loop) so PID 1 doesn't terminate.
* **The Lesson:** A container is not a persistent VM; it dies the second its main entrypoint finishes. Always check `docker ps -a` and inspect output with `docker logs <container_name>`.
