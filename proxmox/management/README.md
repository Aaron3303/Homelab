# 📊 Management LXC

A Proxmox LXC container used as the central **management and monitoring server** for the homelab.

The LXC runs Docker Compose with **Glance** and **Uptime Kuma** and uses Tailscale for remote access.

---

## 🖥️ Container

| Item | Value |
|------|-------|
| Type | Proxmox LXC |
| IP Address | `192.168.1.229` |
| Docker | Docker Compose |
| Remote Access | Tailscale |

---

## 📦 Services

| Service | Purpose | Port |
|---------|---------|------|
| Glance | Homelab dashboard and service overview | `8080` |
| Uptime Kuma | Service and VM monitoring | `3001` |

### Glance

**Glance** is used as the primary dashboard for the homelab.

It provides a centralized interface for viewing information about the infrastructure and services running throughout the homelab.

Glance uses API integrations to display information from:

- Modpack server
- Proxmox VE resource usage
- Uptime Kuma VM status

Dashboard configurations are stored in:

```text
/docker/config/glance.yml
```

Changes to the dashboard should be made directly in `glance.yml`.

---

### Uptime Kuma

**Uptime Kuma** is used to monitor the availability of the homelab's virtual machines and services.

It monitors the VMs with ICMP protocol running throughout the Proxmox environment and provides uptime information.

---

## 🌐 Web Interfaces

### Glance

```text
http://192.168.1.229:8080
```

### Uptime Kuma

```text
http://192.168.1.229:3001
```

---

# 📁 Directory Structure

```text
/docker/monitor/
├── compose.yaml
└── config/
    └── glance.yml
```

The Glance dashboard configuration is located at:

```text
/docker/config/glance.yml
```

---

# 🚀 Setup

### 1. Create the Docker Directory

```bash
mkdir -p /docker/monitor
```

---

### 2. Add the Docker Compose File

Place the `compose.yaml` file in:

```text
/docker/monitor/compose.yaml
```

---

### 3. Configure Glance

Edit the Glance configuration:

```bash
nano /docker/monitor/config/glance.yml
```

Dashboard pages, widgets, APIs, and other Glance configuration are managed from this file.

---

### 4. Start the Stack

From the Docker directory:

```bash
cd /docker/monitor
docker compose up -d
```

Verify the containers are running:

```bash
docker compose ps
```

---

# 🔌 Monitoring & API Integrations

Glance uses API integrations to display information from other services in the homelab.

Current integrations include:

```text
Glance
  │
  ├──► Minecraft Server API
  │
  ├──► Proxmox VE API
  │
  └──► Uptime Kuma API
```

These integrations are configured in:

```text
/docker/monitor/config/glance.yml
```

API keys and other sensitive credentials should **not** be committed to GitHub.

Use environment variables or another secure method for sensitive information.

---

# 🔒 Remote Access

Remote access to the Management LXC is provided through **Tailscale**.

Tailscale is installed directly on the **LXC host**, rather than being run as a Docker container.

The LXC uses `/dev/net/tun` passthrough to allow Tailscale to function correctly.

For information about installing Tailscale in a Proxmox LXC, see:

```text
install/tailscale/
```

---

## 🛡️ TUN Passthrough

The Proxmox LXC must have `/dev/net/tun` passed through from the Proxmox host.

The LXC configuration includes the TUN device passthrough so Tailscale can establish its network connection.

---

# ⚙️ Useful Commands

Start the stack:

```bash
docker compose up -d
```

Stop the stack:

```bash
docker compose down
```

Restart the stack:

```bash
docker compose restart
```

View running containers:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs -f
```

View Glance logs:

```bash
docker compose logs -f glance
```

View Uptime Kuma logs:

```bash
docker compose logs -f uptime-kuma
```

---

# 📝 Notes

- The Management LXC is hosted on Proxmox.
- Docker Compose manages Glance and Uptime Kuma.
- Glance is configured through `/docker/monitor/config/glance.yml`.
- Uptime Kuma monitors the VMs throughout the Proxmox environment.
- Glance integrates with the Modpack server, Proxmox VE, and Uptime Kuma APIs.
- Remote access is provided through Tailscale.
- Tailscale is installed directly on the LXC rather than inside Docker.
- `/dev/net/tun` is passed through from the Proxmox host to the LXC for Tailscale.
- must use nano `/etc/ssh/sshd_config` and change PermitRootLogin to "yes" and PasswordAuthentication to "yes" if facing permission issues when copying or ssh ing into the server
