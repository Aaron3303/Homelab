# 🛡️ Valheim Server

A dedicated Valheim server running in Docker with a Playit proxy for secure remote access.

---

## 📦 Containers

| Container | Purpose |
|-----------|---------|
| `valheim-server` | Hosts the Valheim dedicated server |
| `playit-agent` | Provides remote access through Playit |

---

## 📁 Volumes

| Host Path | Container Purpose |
|-----------|-------------------|
| `/data` | Stores world data, saves, and server configuration |

---

## 🌐 Ports

| Port | Protocol | Description |
|------|----------|-------------|
| `2456` | UDP | Game traffic |
| `2457` | UDP | Game traffic |
| `2458` | UDP | Game traffic |

---

## 🚀 Starting the Server

```bash
docker compose up -d
```

---

## 📜 Viewing Logs

```bash
docker compose logs -f
```

View logs for a specific container:

```bash
docker compose logs -f valheim-server
docker compose logs -f playit-agent
```

---

## 🔄 Updating

Pull the latest images:

```bash
docker compose pull
```

Restart with the updated images:

```bash
docker compose up -d
```

---

## 🗂️ Directory Structure

```text
valheim-server/
├── docker-compose.yml
├── valheim.env
├── data/
├── playit/  
└── config/
    ├── worlds_local/
    ├── backups/
    ├── bannedlist.txt
    ├── permittedlist.txt
    ├── adminlist.txt
    └── prefs
```

---

## ⚙️ Useful Commands

Restart containers:

```bash
docker compose restart
```

Stop containers:

```bash
docker compose down
```

Check running containers:

```bash
docker compose ps
```

Enter the server container:

```bash
docker exec -it valheim-server bash
```

---

## 📝 Notes

- Server data is stored in `/data` and `/config`.
- Remote access is handled by the Playit Agent.
- Docker Compose manages both containers.
- The server is hosted inside a dedicated Proxmox VM.
- Proxmox VM ip is 192.168.1.84
- Playit proxy website https://playit.gg/
- Valheim-server/ directory located in /home/aaron33/ directory
