# 🌌 Create: Astral Minecraft Server

A dedicated **Create: Astral** Minecraft server running in Docker with a Playit proxy for secure remote access.

---

## 📦 Containers

| Container | Purpose |
|-----------|---------|
| `minecraft-server` | Hosts the Create: Astral Minecraft server |
| `playit-modpack` | Provides remote access through Playit |

---

## 📁 Volumes

| Host Path | Container Purpose |
|-----------|-------------------|
| `./docker/astral/server` | Stores world data, server configuration, mods, logs, and player data |

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
docker compose logs create-astral -f 
docker compose logs playit-modpack -f
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

> **Note:** Updating the Docker image does **not** update the Create: Astral modpack. Follow the modpack's update procedure when a new version is released.

---

## 🗂️ Directory Structure

```text
astral/
├── compose.yaml
├── astral/
├── playit/
└── server/
    ├── world/
    ├── mods/
    ├── config/
    ├── logs/
    ├── server.properties
    ├── eula.txt
    └── whitelist.json
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

Enter the Minecraft server container (accessing console):

```bash
docker exec create-astral rcon-cli- "command"
```

Example:

```bash
docker exec create-astral rcon-cli- say Hello World!

docker exec create-astral rcon-cli- op MachineMaker92
```

---

## 📝 Notes

- This server runs the **Create: Astral** modpack.
- World data, configuration, mods, and logs are stored in the `./docker/astral/server` volume.
- Remote access is handled by the Playit Agent.
- Docker Compose manages both the Minecraft server and Playit containers.
- The server is hosted inside a dedicated Proxmox VM.
- VM ip is 192.168.1.227
- Java memory allocation is configured in the Docker Compose file using the `MEMORY` environment variable.
