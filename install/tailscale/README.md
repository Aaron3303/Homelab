# 🛡️ Tailscale Installation

Tailscale provides secure, encrypted remote access to services running in the homelab without exposing ports to the public Internet.

This homelab uses Tailscale in **two different ways** depending on the service.

- **Option 1:** Docker Container (Jellyfin, Arr Stack, etc.)
- **Option 2:** Host Installation (Vaultwarden using Tailscale Serve)

---

# 📦 Option 1 - Docker Container

This method runs Tailscale as a Docker container alongside your other containers.

Before starting the container, you'll need to generate an **Auth Key** from your Tailscale account.

---

## 🔑 Generate an Auth Key

1. Open the **Admin Console**.
2. Navigate to **Keys**.
3. Select **Generate auth key**.
4. Configure the desired options (expiration, reusable, tags, etc.).
5. Select **Generate key**.
6. Copy the generated Auth Key.

> The generated key will be used to automatically register the Docker container with your Tailnet.

---

## ⚙️ Configure Docker Compose

Open your `compose.yaml` and paste the Auth Key into the environment variable.

Example:

```yaml
environment:
  - TS_AUTHKEY=tskey-auth-xxxxxxxxxxxxxxxxxxxxxxxx
```

Once the key has been added, start the container.

```bash
docker compose up -d
```

The container will automatically authenticate and appear in your Tailnet.

---

## ✅ Verify Registration

Visit the Tailscale Admin Console.

Under **Machines**, verify that the new container appears as an online device.

---

# 💻 Option 2 - Host Installation

Some features, such as **Tailscale Serve**, require Tailscale to run directly on the host operating system instead of inside Docker.

This is required for services like **Vaultwarden**, where Tailscale Serve provides the HTTPS endpoint used by the web vault and mobile applications.

---

## 📥 Install Tailscale

Install Tailscale:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Authenticate the host:

```bash
sudo tailscale up
```

---

## 🔗 Authenticate the Device

Running `tailscale up` will display a URL similar to:

```text
https://login.tailscale.com/...
```

Open the URL in your browser and sign in to your Tailscale account.

After authentication, the host machine will automatically join your Tailnet.

---

## ✅ Verify Registration

Open the Tailscale Admin Console.

Navigate to **Machines** and verify that the host appears as an online device.

---

## 📝 Notes

- Use the **Docker** method for containers that simply need private network access.
- Use the **Host Installation** method when you need **Tailscale Serve** or other host-level networking features.
- Vaultwarden in this homelab uses the **Host Installation** method because `tailscale serve` cannot be used from within a Docker container.
- No router port forwarding is required for either installation method.
