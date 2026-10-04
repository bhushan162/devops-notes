# Interview QA: Docker & Edge Containerization

## 🧠 Technical Questions

**Q: What is the fundamental difference between Docker containers and Virtual Machines (VMs), and why do we prefer Docker on edge devices?**  
**A:** The biggest difference is the kernel. A VM requires a hypervisor and boots an entire guest OS kernel for every single instance, which gobbles up fixed RAM, CPU cycles, and disk space. Docker doesn't boot a new kernel; all containers share the host Linux kernel directly. Docker uses Linux kernel primitives—specifically **namespaces** for process and network isolation, and **cgroups** for CPU/RAM resource limits. On edge devices like a Raspberry Pi or an industrial gateway with 1GB to 2GB of RAM, running VMs kills performance, whereas Docker lets you spin up isolated services in milliseconds with almost zero overhead.

**Q: Can you run an `x86_64` container image on an ARM64 edge device like a Raspberry Pi? What happens under the hood?**  
**A:** Out of the box, no. If you pull an `x86_64` image and try to run it on an ARM64 CPU, it fails immediately with an `exec format error`. Docker is a container runtime, not an emulator—it executes machine instructions directly on the host CPU. An ARM CPU simply cannot decode x86 machine instructions. If you really have to run an x86 container on an ARM machine, you must register a QEMU user-space emulator using Linux's `binfmt_misc` kernel module, but that comes with a noticeable performance hit. In production, we always build multi-arch images (`linux/arm64`, `linux/amd64`) via `docker buildx`.

**Q: Explain how port mapping works in `docker run -p 6000:6379 redis`. Which is which?**  
**A:** The syntax is `-p <Host_Port>:<Container_Port>`. So `6000:6379` binds port `6000` on the physical host machine and redirects that incoming network traffic to port `6379` inside the Redis container. The application inside the container still thinks it's listening normally on its default `6379`. If you want external devices on the LAN to talk to Redis, they connect to the host's IP at port `6000`.

**Q: How do you jump inside a running container with a shell, and what is the difference between `-i` and `-t`? What if `/bin/bash` fails?**  
**A:** You run `docker exec -it <container_name> /bin/bash`. 
* `-i` (interactive) keeps `STDIN` open so the container can accept input from your keyboard.
* `-t` (pseudo-TTY) allocates a pseudo-terminal session so you get colored prompts, formatting, and line editing.
* If you pass only `-t` without `-i`, the terminal will display a prompt but appear frozen because it won't accept your keystrokes. You always need `-it` together.
* If it fails with `executable file not found in $PATH` for `/bin/bash`, the image is likely a minimal distro like Alpine that doesn't ship with Bash. Simply swap it for `/bin/sh`: `docker exec -it <container_name> sh`.

**Q: What are the differences between a Bind Mount and a Named Volume, and when would you use each on an edge system?**  
**A:** 
* A **Bind Mount** (`-v /opt/data:/app/data`) maps a specific, absolute folder on the host filesystem directly into the container. It's great when you have external automation scripts or developers that need direct read/write access to raw configuration files or log directories on the host.
* A **Named Volume** (`-v telemetry_vol:/app/data`) is managed completely by Docker under `/var/lib/docker/volumes/`. You don't have to worry about absolute path differences across different host OS versions, and Docker handles permissions and lifecycle cleanly. In production and for embedded databases (like SQLite or embedded TSDBs), named volumes are the standard.

**Q: You notice a simple Python automation script built into an image is 950 MB. How do you optimize it for edge deployment?**  
**A:** That happens because someone wrote `FROM python:3.11`, which pulls down a heavy full Debian/Ubuntu operating system loaded with compilers, apt caches, and development tools that a runtime script never needs. To optimize it:
1. Switch to a minimal base image like `FROM python:3.11-alpine` or Google's `distroless`, which brings the base OS size down to ~5 MB.
2. Order your Dockerfile layers smartly: copy `requirements.txt` and run `pip install` before copying application code so dependency layers stay cached between code edits.
3. Use multi-stage builds if the application requires C extensions or build dependencies, copying only the compiled artifacts into the final runtime stage. This drops the image size from ~950 MB to under 45 MB.

**Q: What happens if a container you started with `docker run -d` doesn't show up in `docker ps`? How do you diagnose it?**  
**A:** It means the container has stopped or crashed. Docker containers only stay alive as long as their PID 1 foreground process is running. I immediately run `docker ps -a` to inspect all containers and check the exit code (e.g., `0` means clean exit, `1` or `137` means crash or OOM kill). Then I run `docker logs <container_name_or_id>` to view stdout/stderr and see the traceback or reason why it exited.

---

## 📖 Behavioral / Experience Stories
* **Story:** Containerizing and optimizing edge telemetry services for resource-constrained gateways.
  * **Situation:** Legacy automation scripts and telemetry loggers were installed directly on host embedded Linux gateways, causing dependency conflicts (Python versions, shared libraries) and risky OTA firmware re-flashing whenever an app needed an update.
  * **Task:** Containerize the telemetry services with minimal footprint to ensure zero host pollution, fast OTA updates, and reliable data persistence across reboots.
  * **Action:** Packaged the services using lightweight Alpine-based Dockerfiles (dropping image size from ~900 MB to 45 MB), configured named volumes for local sensor data persistence during container updates, and automated startup via Docker Compose.
  * **Result:** Enabled atomic zero-downtime updates of individual microservices over constrained network links, eliminated host library conflicts, and ensured sensor data survived container restarts.
