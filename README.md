# 🏠 Homelab

A personal homelab built around **Proxmox VE**, **Docker**, and **TrueNAS**.

This repository is used to document the setup, configuration, and ongoing development of the homelab.

---

## 🎯 Purpose

The main purpose of this homelab is to **learn, experiment, and develop practical IT skills**.

It provides an environment to:

- Learn and practice Linux administration
- Improve networking and virtualization skills
- Learn Docker and container management
- Experiment with Proxmox and virtual machines
- Learn storage management with TrueNAS and NFS
- Practice self-hosting applications and services
- Experiment with remote access and networking
- Host personal game servers and media services
- Troubleshoot real-world infrastructure problems
- Document everything for future reference and rebuilding

The homelab is also a hobby and a place to experiment with different technologies without affecting production systems.

---

# 🖥️ Current Setup

The homelab consists of **two physical computers**.

| System | Purpose |
|--------|---------|
| Proxmox Server | Virtualization host |
| TrueNAS Server | Network-attached storage |

---

## 💻 Proxmox Server

The primary server runs Proxmox VE and hosts the homelab's virtual machines and LXC containers.

| Component | Specification |
|-----------|---------------|
| CPU | Intel Core i7-9700K |
| GPU | NVIDIA RTX 2080 Super |
| RAM | 32 GB DDR4 |
| Storage | 1 TB NVMe SSD |
| Motherboard | ASUS Prime Z390-A |

---

## 💾 TrueNAS Server

An older PC is used as the NAS and runs TrueNAS.

| Component | Specification |
|-----------|---------------|
| CPU | AMD A8-7600 |
| GPU | None |
| RAM | 8 GB DDR3 + 16 GB DDR3 |
| OS Drive | 500 GB SATA SSD |
| Storage | 1 TB HDD + 2 TB HDD |
| Motherboard | ASRock FM2A88M Pro3 |

The TrueNAS server provides network storage to the homelab using NFS.

---

# 🌐 Network

The router connects to a smart switch.

| Switch Port | Device | IP |
|-------------|--------|----|
| Port 1 | Proxmox Server | 192.168.1.101 |
| Port 2 | TrueNAS Server | 192.168.1.234 |

### Network Diagram

```text
                    Internet
                       │
                       ▼
                  ┌─────────┐
                  │ Router  │
                  └────┬────┘
                       │
                       ▼
                ┌─────────────┐
                │ Smart Switch│
                └──────┬──────┘
                       │
              ┌────────┴────────┐
              │                 │
              ▼                 ▼
       ┌────────────┐    ┌────────────┐
       │  Proxmox   │    │  TrueNAS   │
       │   Server   │    │   Server   │
       └────────────┘    └────────────┘
```

---

# 🖼️ Current Setup

## Physical Setup

_Add a picture of the current physical homelab setup here._

```markdown
![Current Homelab Setup](images/homelab-setup.jpg)
```

---

## Network Diagram

_Add the current network diagram here._

```markdown
![Homelab Network Diagram](images/network-diagram.png)
```

---

# 🖥️ Proxmox

The following virtual machines and LXC containers are currently running on the Proxmox server:

### Virtual Machines

- **Minecraft**
- **Valheim**
- **Servarr**
- **Modpack**
- **Vaultwarden**

### LXC Containers

- **Audiobooks**
- **Management**

Detailed information about the Proxmox environment can be found in:

```text
proxmox/
```

---

# 💾 Storage

The TrueNAS server provides network storage to the Proxmox environment through NFS.

The primary media share is available to the appropriate servers through:

```text
/data
```

Media stored under `/data` is physically stored on the TrueNAS server rather than on the Proxmox server.

Detailed TrueNAS configuration can be found in:

```text
truenas/
```

---

# 📚 Documentation

This repository separates general installation guides from service-specific documentation.

### Installation Guides

General installation instructions are located in:

```text
install/
```

Use this directory for guides covering the installation and initial configuration of software used throughout the homelab.

### Proxmox

Detailed VM and LXC documentation:

```text
proxmox/
```

### TrueNAS

Detailed NAS configuration and storage documentation:

```text
truenas/
```

---

# 📊 Monitoring

The homelab uses **Glance** as the main monitoring and dashboard application.

The Glance configuration file is located in the root directory of this repository:

```text
glance.yml
```

---

# 🚀 Future Plans

A few planned improvements for the homelab:

- [ ] Implement VLANs and improve network segmentation
- [ ] Improve backup and disaster recovery
- [ ] Add automation using ansible and terraform
- [ ] Condense and mount equipment to server rack
- [ ] Add additional self-hosted services
- [ ] Improve monitoring and alerting
- [ ] Continue documenting and improving the infrastructure

---

# 🛠️ Useful Notes

## File Ownership

If a service encounters permission issues, ownership can be changed using:

```bash
sudo chown -R user:group dir/
```

Example:

```bash
sudo chown -R aaron33:aaron33 ~/valheim-server/config/worlds_local
```

Replace `user`, `group`, and `dir/` with the appropriate values.

> **Note:** Be careful when changing ownership of files on mounted NAS storage. UID/GID permissions configured on the TrueNAS server may affect access.

---

# 📁 Repository Structure

```text
homelab/
│
├── README.md
├── glance.yml
│
├── install/
│   ├── docker/
│   ├── tailscale/
│   ├── proxmox/
│   ├── playit/
│   └── truenas/
│
├── proxmox/
│   ├── README.md
│   ├── minecraft/
│   ├── valheim/
│   ├── servarr/
│   ├── modpack/
│   ├── vaultwarden/
│   ├── management/
│   └── audiobooks/
│
├── truenas/
│
├── images/
│   ├── homelab-setup.jpg
│   └── network-diagram.png
│
└── .gitignore
```

---

# 🙏 Credits

Some configurations and ideas used throughout this homelab were inspired by community projects and tutorials.

Special thanks to **TechHutTV** for the inspiration and documentation used for the media server setup.

[TechHutTV Homelab](https://github.com/TechHutTV/homelab)

---

## 📌 Disclaimer

This repository documents a personal homelab used for learning, experimentation, and hosting personal services.

All services and media downloading tools are intended to be used for legal purposes, including Linux distributions, open-source software, and freely available media.
