# ⛏️ Vanilla Minecraft Server (26.2)

A dedicated **Vanilla Minecraft 26.2** server running in Docker with a Playit proxy for secure remote access. 

---

## 📦 Containers

| Container | Purpose |
|-----------|---------|
| `mc` | Hosts the Vanilla Minecraft server |
| `playit-docker` | Provides remote access through Playit |

---

## 📁 Volumes

| Host Path | Container Purpose |
|-----------|-------------------|
| `/home/aaron33/minecraft/data` | Stores world data, server configuration, logs, and player data |

---

## 🌐 Ports

| Port | Protocol | Description |
|------|----------|-------------|
| `25565` | TCP | Minecraft server |

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
docker compose logs mc -f
docker compose logs playit-docker -f
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

## 🗂️ Directory Structure

```text
minecraft/
├── compose.yaml
├── playit/
└── data/
    ├── world/
    ├── logs/
    ├── server.properties
    ├── eula.txt
    ├── whitelist.json
    └── ops.json
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

Enter the Minecraft server container:

```bash
docker exec -it minecraft-server bash
```

View the live server console:

```bash
docker attach minecraft-server
```

> Press **Ctrl + P**, then **Ctrl + Q** to detach without stopping the container.

---

## 📝 Notes

- This server runs **Vanilla Minecraft 26.2**. 
- World data, configuration, and logs are stored in the `/data` volume.
- Remote access is handled by the Playit Agent.
- Docker Compose manages both the Minecraft server and Playit containers.
- The server is hosted inside a dedicated Proxmox VM.
- Proxmox VM ip is 192.168.1.128
- Playit proxy website https://playit.gg/
- Java memory allocation is configured in the Docker Compose file using the `MEMORY` environment variable.
