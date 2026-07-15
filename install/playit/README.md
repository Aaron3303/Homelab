# 🌐 Playit.gg Setup

**Playit.gg** provides secure remote access without opening ports on your router.

---

## 📦 Docker Compose

The Playit Agent is included as a second container in the same `compose.yaml` as the game server.

Start the containers:

```bash
docker compose up -d
```

---

## 📜 Register the Agent

View the Playit Agent logs:

```bash
docker compose logs playit-agent -f
```

Copy the registration link displayed in the logs and open it in your browser.

---

## 🔗 Create a Tunnel

After registering the agent:

1. Log in to your Playit.gg account.
2. Create a new tunnel.
3. Select the appropriate game type.
4. Assign the tunnel to the registered agent.
5. Save the configuration.

---

## 🎮 Connect

Once the tunnel is created, Playit automatically generates a public domain.

Players can connect using this domain without needing to:

- Open router ports
- Configure port forwarding
- Know your public IP address

---

## 📝 Notes

- The free Playit.gg plan supports **up to 2 agents**.
- This homelab currently uses:
  - One agent for the **Modpack: "Create: Astral" server**
  - One agent for the **Valheim server**
- Each game server has its own dedicated Playit Agent container and domain.
