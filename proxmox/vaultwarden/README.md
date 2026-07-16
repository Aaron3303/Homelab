# 🔐 Vaultwarden

A self-hosted password manager running in Docker. Remote access is secured through **Tailscale** using **Tailscale Serve**, which provides an HTTPS endpoint required by the Vaultwarden web vault and mobile applications.

---

## 📦 Containers

| Container | Purpose |
|-----------|---------|
| `vaultwarden` | Hosts the Vaultwarden password manager |

---

## 📁 Volumes

| Host Path | Container Purpose |
|-----------|-------------------|
| `/vw-data` | Stores the Vaultwarden database, attachments, and configuration |

---

## 🌐 Ports

| Port | Protocol | Description |
|------|----------|-------------|
| `8080` | TCP | Vaultwarden web interface (local only) |

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

View logs for the Vaultwarden container:

```bash
docker compose logs vaultwarden -f
```

---

## 🔄 Updating

Pull the latest container image:

```bash
docker compose pull
```

Restart with the updated image:

```bash
docker compose up -d
```

---

## 🔒 Remote Access with Tailscale

Vaultwarden is **not exposed to the public Internet**.

Instead, remote access is provided through **Tailscale Serve**, which creates an HTTPS endpoint on your private Tailnet. This HTTPS endpoint is required because Vaultwarden clients and browsers expect a secure (HTTPS) connection.

HTTPS Domain to access Vaultwarden:

```text
https://vaultwarden.tailfef5f3.ts.net/
```

All traffic is encrypted and only authenticated devices on your Tailnet can access the service.

---

## ⚙️ Tailscale Requirements

> **Important:** Tailscale **must be installed on the host machine**, **not** inside a Docker container.

This is because **Tailscale Serve** runs as part of the Tailscale daemon on the host and cannot expose services from a Tailscale container in the same way.

Installation and configuration instructions can be found in:

```text
install/
└── tailscale/
    └── README.md
```

Once Tailscale is installed on the host, configure Serve to forward HTTPS requests to the Vaultwarden container.

---

## 🗂️ Directory Structure

```text
vaultwarden/
├── compose.yaml
└── vw-data/
    ├── db.sqlite3
    ├── db.sqlite3-shm
    ├── db.sqlite3-wal
    ├── rsa_key.pem
    └── tmp/
```

---

## ⚙️ Useful Commands

Restart the container:

```bash
docker compose restart
```

Stop the container:

```bash
docker compose down
```

Check container status:

```bash
docker compose ps
```

View the live logs:

```bash
docker compose logs vaultwarden -f
```

---

## 📝 Notes

- Vaultwarden runs entirely within Docker.
- Remote access is handled by **Tailscale Serve**, not by exposing ports on the router.
- No port forwarding is required.
- Only devices connected to your Tailnet can access the Vaultwarden instance.
- Tailscale must be installed on the **host operating system** for the `tailscale serve` feature to function correctly.
- See `install/tailscale/README.md` for the complete installation and configuration guide.
- Vaultwarden access link genereate by tailscale https://vaultwarden.tailfef5f3.ts.net/
- Proxmox VM ip is 192.168.1.231

