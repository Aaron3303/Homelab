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
