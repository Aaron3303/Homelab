# 🎬 Servarr Media Server

The **Servarr** VM hosts the media management stack for downloading and organizing **Linux ISOs and other freely available media**. Downloaded media is automatically organized and made available to Jellyfin for streaming.

> **Disclaimer:** This server is intended only for downloading Linux distributions and other legally available free media.

---

## 📦 Containers

| Container | Purpose | Status |
|-----------|---------|--------|
| `gluetun` | VPN gateway for download applications | ✅ Active |
| `qbittorrent` | BitTorrent client | ✅ Active |
| `prowlarr` | Indexer manager | ✅ Active |
| `radarr` | Movie management | ✅ Active |
| `sonarr` | TV show management | ✅ Active |
| `seerr` | Media request management | ✅ Active |
| `bazarr` | Subtitle management | ⏸️ Unused |
| `lidarr` | Music management | ⏸️ Unused |
| `nzbget` | Usenet downloader | ⏸️ Unused |

---

## 🌐 Network

The Servarr stack runs on a dedicated Docker network.

| Network | Subnet |
|---------|--------|
| `servarrnetwork` | `172.39.0.0/24` |

Containers requiring VPN access share the **Gluetun** network namespace.

---

## 🌐 Web Interfaces

| Service | Port |
|---------|------|
| qBittorrent | `8080` |
| Radarr | `7878` |
| Sonarr | `8989` |
| Prowlarr | `9696` |
| Overseerr | `5055` |
| Bazarr | `6767` |
| Lidarr | `8686` |
| NZBGet | `6789` |

---

## 🚀 Starting the Stack

```bash
docker compose up -d
```

---

## 📜 Viewing Logs

View all containers:

```bash
docker compose logs -f
```

View a specific container:

```bash
docker compose logs radarr -f
docker compose logs sonarr -f
docker compose logs qbittorrent -f
docker compose logs gluetun -f
```

---

## 🔄 Updating

Pull the latest container images:

```bash
docker compose pull
```

Restart with the updated images:

```bash
docker compose up -d
```

---

## 📁 Directory Structure

### Docker Configuration

```text
/docker
├── tailscale/
│   ├── tailscale-state/
│   └── compose.yaml
│
├── jellyfin/
│   ├── config/
│   └── compose.yaml
│
└── servarr/
    ├── bazarr/
    ├── gluetun/
    ├── lidarr/
    ├── nzbget/
    ├── prowlarr/
    ├── qbittorrent/
    ├── radarr/
    ├── seerr/
    ├── sonarr/
    ├── .env
    └── compose.yaml
```

---

### Media Storage

```text
/data
├── downloads/
│   ├── nzbget/
│   └── qbittorrent/
│       ├── completed/
│       ├── incomplete/
│       └── torrents/
│
├── movies/
└── shows/
```

---

## 💾 Storage

The `/data` directory is **not stored locally** on this VM.

Instead, it is an NFS mount from the TrueNAS server.

| Item | Value |
|------|-------|
| NAS Software | TrueNAS |
| NAS IP | `192.168.1.234` |
| Mount Location | `/data` |

Although applications read and write to `/data`, all media is physically stored on the NAS.

---

## ⚙️ Initial Setup

### Determine User and Group IDs

This stack uses the `PUID` and `PGID` variables defined in the `.env` file.

For most Ubuntu installations these values are `1000`, but verify them by running:

```bash
id <your_username>
```

Example output:

```text
uid=1000(your_user) gid=1000(your_user) groups=1000(your_user),27(sudo),24(cdrom),30(dip)
```

Set the matching values in the `.env` file before starting the stack.

---

### Set Directory Permissions

If permission issues occur after creating the media directories, update ownership:

```bash
sudo chown -R 1000:1000 /data
```

If using an NFS share, ensure the permissions on the NAS match the user and group IDs configured in Docker.

---

### Docker Configuration Directory

All Docker application data for this VM is stored under:

```text
/docker
```

If recreating the server, create the directory and assign ownership:

```bash
sudo mkdir /docker
sudo chown -R 1000:1000 /docker
```

> Docker configuration files are intentionally stored locally on the VM. Media files are stored separately on the mounted TrueNAS share.

---

## 🔒 VPN Connectivity

The download applications route all traffic through **Gluetun**.

To verify the VPN is working correctly, run:

```bash
docker run --rm --network=container:gluetun alpine:3.18 sh -c "apk add wget && wget -qO- https://ipinfo.io"
```

The returned public IP should match your VPN provider—not your home ISP.

If connectivity issues occur, restart the stack:

```bash
docker compose restart
```

You can also verify connectivity from inside a container:

```bash
docker exec -it qbittorrent bash
wget -qO- https://ipinfo.io
```

The same test can be performed from the `prowlarr` or `nzbget` containers.

---

## ☁️ FlareSolverr

Some indexers are protected by Cloudflare or DDoS-Guard, preventing Prowlarr from communicating with them directly.

FlareSolverr solves these browser challenges automatically.

To configure FlareSolverr in Prowlarr:

1. Open **Settings → Indexers**
2. Under **Indexer Proxies**, click **+**
3. Select **FlareSolverr**
4. Set the host to:

```text
http://localhost:8191
```

5. Click **Test**
6. Click **Save**

Once configured, Prowlarr will automatically use FlareSolverr for supported indexers.

---

## ⬇️ qBittorrent Configuration

### Initial Login

The first time qBittorrent starts it generates a temporary administrator password.

Retrieve it with:

```bash
docker container logs qbittorrent
```

After logging in, change the username and password under:

```text
Settings
└── WebUI
    └── Authentication
```

---

### Download Directories

Configure the following paths inside **Settings → Downloads**.

| Setting | Path |
|---------|------|
| Default Save Path | `/data/downloads/qbittorrent/completed` |
| Keep Incomplete Torrents In | `/data/downloads/qbittorrent/incomplete` |
| Copy .torrent Files To | `/data/downloads/qbittorrent/torrents` |

---

## 🙏 Acknowledgements

This media stack is heavily inspired by **TechHutTV's Homelab** project.

Credit:

- GitHub Repository: https://github.com/TechHutTV/homelab/tree/main/media

The configuration has been adapted to fit this homelab's architecture, including:

- My Proxmox enviroment
- TrueNAS-mounted media storage
- Separate Docker Compose projects
- Tailscale integration

## 🔄 Media Workflow

```text
Linux ISO / Free Media Request
            │
            ▼
       Overseerr
            │
            ▼
     Sonarr / Radarr
            │
            ▼
        Prowlarr
            │
            ▼
 qBittorrent (VPN via Gluetun)
            │
            ▼
/data/downloads/
            │
            ▼
Sonarr / Radarr Import
            │
            ▼
/data/movies/
/data/shows/
            │
            ▼
Jellyfin Library
```

## 🔄 Stack Overview

```text
Internet
     │
     ▼
 Gluetun (VPN)
     │
 ┌───────────────┐
 │ qBittorrent   │
 │ NZBGet        │
 │ Prowlarr      │
 └───────────────┘
        │
        ▼
 /data/downloads
        │
        ▼
 Sonarr / Radarr
        │
        ▼
 /data/movies
 /data/shows
        │
        ▼
 TrueNAS (192.168.1.234)
        │
        ▼
 Jellyfin
```

---

## ⚙️ Useful Commands

Restart the stack:

```bash
docker compose restart
```

Stop the stack:

```bash
docker compose down
```

Check running containers:

```bash
docker compose ps
```

Enter a container:

```bash
docker exec -it radarr bash
```

---

## 📝 Notes

- All downloaded media is stored on the TrueNAS server through the `/data` NFS mount.
- qBittorrent, Prowlarr, and NZBGet route all network traffic through **Gluetun**.
- Radarr and Sonarr organize completed downloads into the appropriate media libraries.
- Jellyfin uses the same `/data` directory to stream media.
- Bazarr, Lidarr, and NZBGet are currently installed but not actively used.
- Container configuration is stored under `/docker`, while media files are stored under `/data`.
- Proxmox VM ip is 192.168.1.236
- More information regarding tailscale and truenas can be found in the /install and /truenas directories respectively
