# 🎧 Audiobooks LXC

A Proxmox LXC container running Docker Compose for **Audiobookshelf** and **AudiobookRequest**. The container uses the same TrueNAS storage as the Servarr media VM.

---

## 🖥️ Container

| Item | Value |
|------|-------|
| Type | Proxmox LXC |
| IP Address | `192.168.1.203` |
| Docker | Docker Compose |
| Storage | NFS mount from TrueNAS |
| Remote Access | Tailscale |

---

## 📦 Containers

| Container | Purpose | Port |
|-----------|---------|------|
| `audiobookshelf` | Media server for organizing and streaming audiobooks | `13378` |
| `audiobookrequest` | Allows users to request audiobooks and sends requests to the download stack | `8000` |

### Audiobookshelf

**Audiobookshelf (ABS)** is the media server used to organize, manage, and stream the audiobook library.

The audiobook library is located at:

```text
/data/audiobooks
```

### AudiobookRequest

**AudiobookRequest (ABR)** provides a web interface for requesting audiobooks.

ABR connects to:

- **Prowlarr** — to search for available audiobook releases
- **Audiobookshelf** — using its API key to interact with the audiobook library

The API connections allow requests made through ABR to be searched through Prowlarr and then managed through the audiobook server.

---

# 🚀 Setup Guide

### 1. Add the Docker Compose File

Copy the `compose.yaml` file into the Audiobooks LXC.

```text
/docker/audiobooks/
└── compose.yaml
```

---

### 2. Start the Containers

From the directory containing `compose.yaml`, run:

```bash
docker compose up -d
```

Verify that the containers are running:

```bash
docker compose ps
```

---

### 3. Access the Services

Access the services using the LXC's IP address:

**Audiobookshelf**

```text
http://192.168.1.203:13378
```

**AudiobookRequest**

```text
http://192.168.1.203:8000
```

Complete the initial setup for both services.

---

### 4. Configure API Connections

Configure **AudiobookRequest** to communicate with:

- **Prowlarr** — for searching indexers
- **Audiobookshelf** — for managing the audiobook library

API keys from the respective services are required to establish the connections.

---

### 5. Configure Storage

Ensure the TrueNAS NFS share is mounted to the Proxmox host and passed through to the LXC at:

```text
/data
```

Audiobookshelf should use:

```text
/data/audiobooks
```

as its audiobook library.

See the **NAS Storage** section below for the complete Proxmox NFS mount and LXC passthrough configuration.

---

### 6. Configure Remote Access

Tailscale is installed directly on the LXC and provides remote access to the services.

Verify Tailscale is connected:

```bash
sudo tailscale status
```

Once connected, the Audiobookshelf and AudiobookRequest services can be accessed through the Tailscale network without opening ports on the router.

---

## 🌐 Network

The LXC has the following local IP address:

```text
192.168.1.203
```

### Web Interfaces

| Service | URL | Port |
|---------|-----|------|
| Audiobookshelf | `http://192.168.1.203:13378` | `13378` |
| AudiobookRequest | `http://192.168.1.203:8000` | `8000` |

---

## 🔗 Service Connections

AudiobookRequest uses API keys to communicate with the other services.

```text
AudiobookRequest
       │
       ├──────────► Prowlarr
       │              │
       │              ▼
       │         Search Indexers
       │
       └──────────► Audiobookshelf
                      │
                      ▼
               Audiobook Library
```

---

# 💾 NAS Storage

The audiobook LXC uses the same TrueNAS storage that is used by the **Servarr media VM**.

### TrueNAS

| Item | Value |
|------|-------|
| NAS Software | TrueNAS |
| NAS IP | `192.168.1.234` |
| NFS Share | `/mnt/mediapool/data` |
| LXC Mount | `/data` |

Audiobooks are stored on the NAS at:

```text
/data/audiobooks
```

From the LXC's perspective, this appears as a normal `/data` directory.

---

# 🔧 NFS Mount Configuration

Unlike a normal VM, the NFS share is mounted on the **Proxmox host** and then passed through to the LXC container.

## 1. Mount NFS on Proxmox

Create the mount directory:

```bash
mkdir /data
```

Edit the Proxmox host's `/etc/fstab`:

```bash
nano /etc/fstab
```

Add:

```text
192.168.1.234:/mnt/mediapool/data /data nfs defaults,_netdev,noatime,nofail 0 0
```

Reload system configuration:

```bash
systemctl daemon-reload
```

Mount the NFS share:

```bash
mount -a
```

At this point, the TrueNAS NFS share should be mounted on the Proxmox host at:

```text
/data
```

---

# 🔌 Pass `/data` to the LXC

The Proxmox host's `/data` directory is passed through to the Audiobooks LXC.

Stop the container:

```bash
pct stop 106
```

Edit the LXC configuration:

```bash
nano /etc/pve/lxc/106.conf
```

Add:

```text
mp0: /data,mp=/data
```

Start the container:

```bash
pct start 106
```

The LXC can now access the NAS through:

```text
/data
```

---

# ⚠️ NAS Permissions

Currently, the Audiobooks LXC **does not have permission to modify the contents of the mounted NAS share**.

This is due to the permissions configured on the TrueNAS server.

The LXC is accessing the share as `root`, but `root` does not have the same UID as the user that owns the files on the NAS.

### Current Workaround

If files on the NAS need to be modified, do so from the **Servarr VM** using the `aaron33` user.

> This is a temporary limitation and can be corrected later by adjusting the TrueNAS permissions/ownership and matching the appropriate UID/GID.

---

# 🐳 Docker Compose

The services are managed using Docker Compose.

Start the stack:

```bash
docker compose up -d
```

View running containers:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs -f
```

Restart the stack:

```bash
docker compose restart
```

Stop the stack:

```bash
docker compose down
```

---

# 🔒 Remote Access

Remote access to the Audiobooks LXC is provided through **Tailscale**.

Tailscale is installed directly on the **LXC host/container**, rather than being run as another Docker container.

This allows the Audiobookshelf and AudiobookRequest services to be accessed remotely through the private Tailscale network without opening ports on the router.

---

## 📝 Notes

- The Audiobooks server runs inside **Proxmox LXC 106**.
- Docker Compose manages Audiobookshelf and AudiobookRequest.
- Audiobookshelf uses `/data/audiobooks` as its library.
- `/data` is backed by a TrueNAS NFS share.
- The NFS share is mounted on the Proxmox host and passed through to the LXC.
- AudiobookRequest communicates with Prowlarr and Audiobookshelf using API keys.
- Remote access is provided through Tailscale.
- The LXC currently has read/access limitations when modifying NAS files because of the current TrueNAS UID/GID permissions.
- must use nano `/etc/ssh/sshd_config` and change PermitRootLogin to "yes" and PasswordAuthentication to "yes" if facing permission issues when copying or ssh ing into the server



