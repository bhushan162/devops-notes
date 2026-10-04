# Docker Fundamentals & Container Lifecycle

> **Status:** 🟢 Active Learning  
> **Tags:** #docker, #containers, #devops, #edge-computing, #interview-prep

## 📝 TL;DR (1-Minute Revision)
* **What is it:** A lightweight containerization platform that packages applications and dependencies together, sharing the host OS kernel.
* **Primary use:** Run portable, isolated edge applications (like MQTT brokers, telemetry collectors) on edge gateways without the heavy memory and CPU overhead of virtual machines.
* **Key command:** `docker run -d -p <host_port>:<container_port> --name <name> <image_name>`

---

## 🧠 Core Concepts

* **Docker vs Virtual Machines (VMs):**
  | Feature | Docker | VMs |
  |---|---|---|
  | **Kernel** | Shares the Host OS Kernel (isolated processes via namespaces & cgroups) | Requires a full, separate Guest OS per VM |
  | **Resource Footprint** | Very lightweight (tens of MBs) | Heavy (GBs of disk and RAM reserved) |
  | **Boot Time** | Seconds / milliseconds | Minutes (full OS boot cycle) |
  | **Portability** | High: run the same image anywhere the kernel and CPU arch match | Bulky VM images, hypervisor dependent |

* **Port Mapping (`-p <Host_Port>:<Container_Port>`):**
  * Format: `-p <Host_Port>:<Container_Port>`.
  * Example: `-p 6000:6379` forwards incoming traffic on port `6000` of the host machine to port `6379` inside the container (e.g., Redis).
  * The application inside the container always listens on its own container port.

* **Container Lifecycle & Background Execution:**
  * `-d` (Detached mode): Runs container in the background and prints container ID, freeing up your terminal without streaming logs.
  * `--name`: Assigns a readable custom name to the container instead of a random auto-generated hash/name.
  * A container only lives as long as its main process (PID 1) is running. When PID 1 exits, the container stops.

* **Interactive Sandbox & Exec (`-it`):**
  * `docker run -it --name platform_sandbox ubuntu:latest bash`
  * `docker exec -it <name> /bin/bash` (or `sh` for Alpine)
  * `-i` (Interactive): Keeps `STDIN` open so the shell can read your keyboard input.
  * `-t` (Pseudo-TTY): Allocates a terminal screen session with prompt formatting and line editing.
  * **Critical:** If you only use `-t` without `-i`, you see a prompt but can't type anything into it! Both flags are needed together (`-it`).
  * If `/bin/bash` is not found (common in Alpine images), use `sh` instead: `docker exec -it <name> sh`.

* **Docker Compose:**
  * Whenever we need to create and run multiple containers/services together, we use a single `docker-compose.yml` file to bring up the entire environment.
  * Defines networks, services, volumes, and environment variables declaratively.

* **How it fits into Edge DevOps:**
  * Edge devices (like Raspberry Pi, NVIDIA Jetson, industrial gateways) have tight RAM and CPU limits. Running VMs on them kills performance.
  * Docker allows running multi-tenant edge services (data loggers, local web dashboards, message brokers) with near-zero overhead.
  * Containers must match the target CPU architecture (e.g., `linux/arm64` vs `linux/amd64`), otherwise the target kernel cannot execute the instructions without QEMU emulation.

---

## 💡 My "Aha!" Moments & Pain Points
* **What clicked for me:** 
  * Docker isn't a mini-VM. It's just an isolated process talking directly to my host Linux kernel.
  * Port mapping `-p 6000:6379` is literally: `Host:Container` (Outside : Inside).
  * Always use `-it` when exec-ing inside. Using `-t` alone renders a frozen prompt because `STDIN` (`-i`) isn't open to receive keystrokes!
  * Using `docker rm -f $(docker ps -a -q)` is the ultimate shortcut when my test containers pile up and clutter the system.
* **What sucks about it:** 
  * If the primary command finishes immediately, the container silently stops and won't show up in `docker ps` unless you run `docker ps -a`.
  * Forgetting the `-d` flag locks up your terminal with live logs.
  * Trying to run `/bin/bash` in an Alpine container fails with "not found" because Alpine only has `/bin/sh` by default.

---

## 🛠️ Playbook & Commands

### 1. Basic Image & Container Inspection
```bash
# Pull an image from Docker Hub
docker pull <Image_name>

# List locally downloaded images
docker images

# List currently running containers
docker ps

# List all containers (both running and stopped)
docker ps -a
```

### 2. Running & Managing Containers
```bash
# Run any image
docker run <Image_name>

# Run in background (detached) with port forwarding and custom name
docker run -d -p 6001:6379 --name redis-older redis

# View logs of a container
docker logs <name_or_id>

# Follow live container logs
docker logs -f <name_or_id>

# Start and Stop containers
docker start <ID_or_name>
docker stop <ID_or_name>

# Jump inside a running container with an interactive shell (Bash)
docker exec -it <ID_or_name> /bin/bash

# Jump inside an Alpine / minimal container (sh fallback)
docker exec -it <ID_or_name> sh
```

### 3. Cleanup Commands
```bash
# Remove a stopped container
docker rm <ID_or_name>

# Remove a Docker image
docker rmi <Image_name>

# Nuclear cleanup: Delete ALL containers (running or stopped)
docker rm -f $(docker ps -a -q)
```

### 4. Docker Compose
```bash
# Start all containers defined in docker-compose.yml in background
docker compose up -d

# Stop all compose containers
docker compose stop

# Stop and remove containers, networks, and volumes
docker compose down
```
