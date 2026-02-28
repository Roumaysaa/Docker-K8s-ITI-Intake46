# Docker Fundamentals & Resource Management — Lab 1

This repository contains the documentation and implementation of Lab #1, focusing on Docker installation, container lifecycle management, resource constraints (cgroups), and Out-of-Memory (OOM) handling.

## 🛠️ Task 1: Environment Setup & Nginx Deployment

### 1. Installation
Docker engine was installed on an Ubuntu WSL instance using the official convenience script:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

Validated installation by ensuring the user is a member of the `docker` group.

<img width="700" height="300" alt="image" src="https://github.com/user-attachments/assets/2ce01adb-68e4-410f-b1bd-adcc265c65cc" />


### 2. Running Nginx in Background
A detached Nginx container was deployed using the `nginx:alpine` image. Example command used:

```bash
docker run -d --name Roumaysaa-ITI-25 -p 80:80 nginx:alpine
```

Status: container is `Up` and port `80` is mapped to the host.

Verify:

<img width="700" height="200" alt="image" src="https://github.com/user-attachments/assets/81998cbd-a848-44aa-b410-dc25d03a7669" />

## ⚖️ Task 2: Resource Constraints & Cgroups

To demonstrate resource control, an Apache (`httpd`) container was launched with strict limits.

### 1. Applying limits
The container was restricted to 70 MB of RAM and 1 CPU core:

```bash
docker run -d --name apache --memory="70m" --cpus="1" httpd
```

### 2. Verifying via `docker inspect`
Using `docker inspect` we confirmed the internal configuration values:

```bash
docker inspect apache | grep -i memory
docker inspect apache | grep -i cpus
```

- Memory Limit (bytes): ~73,400,320 (70 MB)
- CPU Limit (NanoCPUs): 1000000000 (1 core)
  
<img width="700" height="150" alt="image" src="https://github.com/user-attachments/assets/fa5fc083-54c2-4eb4-8845-72f9bc1e36eb" />

### 3. Deep dive: Linux cgroups
On the host, verify kernel enforcement through cgroups :

```bash
cat /sys/fs/cgroup/memory/docker/docker-774fb4a67f75259dff7650bddef302e80558dc962585a1eb07174476d3eb6d14.scope/memory.max
```

Returned value `73400320` demonstrates kernel-level enforcement of the memory limit.

<img width="700" height="150" alt="image" src="https://github.com/user-attachments/assets/318dbf25-54a1-4ef3-b17e-258aa3d0829e" />

## 🚀 Bonus Task: The OOM Killer Challenge

The goal was to intentionally trigger the OOM killer by exceeding container memory limits.

### 1. Configuration
Create an Alpine container `memory-eater` with a 50 MB memory limit and effectively no swap:

```bash
docker run -d --name memory-eater --memory="50m" --memory-swap="50m" alpine:latest 
```

### 2. Stress test
Install `stress-ng` inside the container and allocate more memory than the limit:

```bash
docker exec -it memory-eater sh
apk add stress-ng
```
<img width="700" height="150" alt="image" src="https://github.com/user-attachments/assets/b45be33f-0e12-4b90-bde8-6123ce58d2ac" />

As soon as memory demand exceeded the 50 MB limit, the process was killed by the kernel.

### 3. Proof of termination
The container exited with code `137` (standard OOM exit). Verify with:

```bash
docker inspect memory-eater | grep "OOMKiller"
```

Output: `true`

<img width="700" height="71" alt="image" src="https://github.com/user-attachments/assets/9764f6f2-0a04-45da-9352-aa858a0fec8e" />
<img width="700" height="52" alt="image" src="https://github.com/user-attachments/assets/9aa47afc-0017-4b5b-b909-9afec314dec6" />


