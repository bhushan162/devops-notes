# Docker Volumes & Data Persistence

> **Status:** 🟢 Active Learning  
> **Tags:** #docker, #volumes, #storage, #persistence, #edge-devops

## 📝 TL;DR (1-Minute Revision)
* **What is it:** Dedicated storage mechanisms for persisting container data outside of the container's ephemeral read-write layer.
* **Primary use:** Preserve edge sensor data, databases, and logs so data survives container restarts, crashes, and OTA firmware/image updates.
* **Key command:** `docker run -v <named_volume>:<container_path> <image>`

---

## 🧠 Core Concepts

* **Why Containers Need Volumes:**
  * Containers are ephemeral by default. Any data written inside a container without a volume is lost when the container is deleted (`docker rm`).
  * Volumes decouple persistent application data from the container lifecycle.

* **The 3 Types of Volumes:**
  * **Type 1: Host Bind Mount (`/host/path:/container/path`):**
    * Maps an exact, user-specified directory on the host machine to a directory inside the container.
    * Best when you need host automation scripts or developers to directly inspect or edit files.
    * Example: `docker run -v /home/mount/data:/var/lib/mysql/data ...`
  * **Type 2: Anonymous Volume (`/container/path`):**
    * You specify only the container destination path; Docker automatically allocates a random storage folder on the host (usually inside `/var/lib/docker/volumes/`).
    * Harder to reference directly from the host.
    * Example: `docker run -v /var/lib/mysql/data ...`
  * **Type 3: Named Volume (`<volume_name>:/container/path`):**
    * You give a human-readable reference name instead of an exact host filesystem path.
    * Docker completely manages the underlying storage location safely.
    * **This is the most common and recommended approach for production and databases.**
    * Example: `docker run -v db_data:/var/lib/mysql/data ...`

* **How it fits into Edge DevOps:**
  * In edge devices, telemetry data (e.g., SQLite files, JSON sensor logs) must be written to external flash partitions or persistent mount points.
  * If an OTA update replaces the container image with a new version, a persistent volume ensures no historical edge telemetry or customer config is lost during the recreation.

---

## 💡 My "Aha!" Moments & Pain Points
* **What clicked for me:** 
  * Named volumes are the cleanest: you give Docker a name (e.g. `redis_data`), and it handles the exact host directory without messing up absolute paths across different host machines.
  * Bind mounts are great when I'm debugging locally because I can open the folder directly in VS Code on my host.
* **What sucks about it:** 
  * Permission mismatches: If the user inside the container (e.g. `uid 1000`) doesn't match permissions on the host bind mount, the container throws `EACCES: permission denied` when trying to write.

---

## 🛠️ Playbook & Commands

### 1. Running Containers with Volumes
```bash
# Type 1: Host Bind Mount (Host path : Container path)
docker run -d -v /home/mount/data:/var/lib/mysql/data --name db mysql:latest

# Type 2: Anonymous Volume (Container path only)
docker run -d -v /var/lib/mysql/data --name db-anon mysql:latest

# Type 3: Named Volume (Reference name : Container path) - MOST USED
docker run -d -v my_mysql_data:/var/lib/mysql/data --name db-named mysql:latest
```

### 2. Managing Docker Volumes
```bash
# List all Docker volumes
docker volume ls

# Inspect details of a specific volume (shows host Mountpoint)
docker volume inspect my_mysql_data

# Create a named volume explicitly
docker volume create edge_sensor_logs

# Remove an unused volume
docker volume rm my_mysql_data

# Clean up all dangling (unattached) volumes
docker volume prune
```
