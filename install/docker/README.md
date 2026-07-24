# 🐳 Docker Engine Installation

This guide installs the latest Docker Engine and Docker Compose Plugin from Docker's official Ubuntu repository.

---

## 📋 Prerequisites

- Ubuntu Server (22.04+ recommended)
- User with `sudo` privileges

---

## 📦 Install Docker Repository

Update package lists and install the required dependencies.

```bash
# Update package index
sudo apt update

# Install required packages
sudo apt install ca-certificates curl

# Create the keyrings directory
sudo install -m 0755 -d /etc/apt/keyrings

# Download Docker's GPG key
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

# Allow Apt to read the key
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Add Docker's official repository.

```bash
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

Update the package index.

```bash
sudo apt update
```

---

## 📥 Install Docker Engine

Install Docker Engine along with the Docker Compose plugin.

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

---

## ✅ Verify Docker Service

Check whether Docker is running.

```bash
sudo systemctl status docker
```

If Docker is not running, start the service.

```bash
sudo systemctl start docker
```

(Optional) Enable Docker to start automatically after reboot.

```bash
sudo systemctl enable docker
```

---

## 🧪 Verify Installation

Run the official Docker test container.

```bash
sudo docker run hello-world
```

If Docker is installed correctly, you should see a welcome message confirming that your installation is working.

---

## 🔍 Useful Commands

Check Docker version.

```bash
docker --version
```

Check Docker Compose version.

```bash
docker compose version
```

View running containers.

```bash
docker ps
```

View all containers.

```bash
docker ps -a
```

View downloaded images.

```bash
docker images
```

Stop the Docker service.

```bash
sudo systemctl stop docker
```

Restart the Docker service.

```bash
sudo systemctl restart docker
```

---

## 📁 Installed Components

| Component | Purpose |
|-----------|---------|
| `docker-ce` | Docker Engine |
| `docker-ce-cli` | Docker command-line interface |
| `containerd.io` | Container runtime |
| `docker-buildx-plugin` | Extended image building features |
| `docker-compose-plugin` | Enables the `docker compose` command |

---

## 📝 Notes

- This installs Docker directly from Docker's official repository instead of Ubuntu's package repository.
- Docker docs ubuntu website https://docs.docker.com/engine/install/ubuntu/
- Reccomended to install on host
- Adding user to docker group (remove need for sudo) (sudo usermod -aG docker $USER)
  
  <img width="553" height="279" alt="image" src="https://github.com/user-attachments/assets/6a5b2345-ef83-4b78-8559-24989b5f4bd1" />

