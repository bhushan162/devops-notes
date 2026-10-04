# Dockerfile & Image Optimization

> **Status:** 🟢 Active Learning  
> **Tags:** #docker, #dockerfile, #optimization, #alpine, #edge-computing

## 📝 TL;DR (1-Minute Revision)
* **What is it:** A Dockerfile is a text blueprint of instructions used by the Docker engine to build custom, reproducible container images for an application.
* **Primary use:** Package edge microservices and Python automation scripts into minimal footprint images for deployment to embedded hardware.
* **Key command:** `docker build -t my-app:1.0 .`

---

## 🧠 Core Concepts

* **Dockerfile Purpose:**
  * Used to automate the creation of a Docker image for our custom application and services.
  * Every instruction (`FROM`, `RUN`, `COPY`) creates a read-only filesystem layer.

* **Essential Dockerfile Instructions:**
  * `FROM <base_image>`: Sets the base image (e.g., `node`, `python:3.11-alpine`, `ubuntu`).
  * `ENV <KEY>=<value>`: Sets environment variables available during build and container runtime (e.g., DB credentials or port configurations).
  * `WORKDIR <path>`: Sets the working directory inside the container for subsequent instructions.
  * `RUN <command>`: Executes commands during the build phase (e.g., `RUN mkdir -p /home/app` or `RUN pip install ...`).
  * `COPY <src> <dest>`: Copies files or folders from the host build context into the container filesystem.
  * `CMD ["executable", "param1"]`: Default command executed when the container starts up.

* **Base Image Selection: Heavy Distros vs Alpine / Distroless:**
  * When writing `FROM python:3.11`, Docker pulls down a full, heavy Debian/Ubuntu Linux OS layer containing hundreds of system tools (compilers, package managers, text editors) that the script never uses.
  * To drop image size drastically (e.g., **950 MB down to under 50 MB**), use **Alpine Linux** (`python:3.11-alpine`) or **Distroless** base images.
  * Alpine is an ultra-lightweight, security-oriented Linux distribution that starts at only **5 MB**.

* **How it fits into Edge DevOps:**
  * Embedded targets (Raspberry Pi, Toradex, edge gateways) use SD cards or eMMC flash memory with limited storage (8 GB - 32 GB) and high flash wear sensitivity.
  * Pushing 1 GB images over cellular (LTE/5G) or satellite connections to edge devices is slow, expensive, and prone to OTA timeouts. Keeping images under 50 MB is essential.

---

## 💡 My "Aha!" Moments & Pain Points
* **What clicked for me:** 
  * You don't need a full Ubuntu OS just to execute a 5 KB Python or Node script. 
  * Switching `FROM python:3.11` to `FROM python:3.11-alpine` took my image from 950 MB to 45 MB instantly with zero extra code.
* **What sucks about it:** 
  * Alpine uses `musl libc` instead of `glibc`. If your C/C++ or Python libraries need precompiled wheels targeting glibc, they might compile from source or fail unless dependencies are explicitly installed.
  * Layer order matters: if you put `COPY . .` before installing dependencies, any small code change invalidates Docker's build cache and forces you to reinstall everything from scratch.

---

## 🛠️ Playbook & Commands

### 1. The Rookie Way (950 MB Image Size)
```dockerfile
# Heavy base image pulling full Debian/Ubuntu tools
FROM python:3.11
COPY . /app
CMD ["python", "/app/sys_check.py"]
```

### 2. The Enterprise Way (Alpine + Smart Layer Caching)
```dockerfile
# 1. Use the tiny alpine variant (~5MB base OS)
FROM python:3.11-alpine  

# 2. Set an explicit working directory
WORKDIR /app             

# 3. Layer Caching: Copy only dependency manifests first
# This ensures pip install ONLY re-runs if requirements.txt changes!
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 4. Copy the actual application source code
COPY . .      

# 5. Execute cleanly
CMD ["python", "collector.py"]
```

### 3. Node.js Application Example
```dockerfile
FROM node
ENV mongo_DB_USERNAME=admin \
    mongo_DB_PWD=password

WORKDIR /home/app
COPY . /home/app

CMD ["node", "server.js"]
```

### 4. Build Commands
```bash
# Build an image from the Dockerfile in the current directory
docker build -t my-app:1.0 .

# Build with a specific Dockerfile path
docker build -f Dockerfile.edge -t my-edge-app:1.0 .

# Build without using cached layers
docker build --no-cache -t my-app:1.0 .
```
