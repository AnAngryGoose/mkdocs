# [Tailscale: Quick Guide](https://tailscale.com/)

---

**Tailscale** is a zero-configuration VPN that creates a secure private network (a "Tailnet") between your devices. Unlike traditional VPNs that route everything through a central server, Tailscale creates a **mesh network** where devices connect directly to each other (peer-to-peer) using the **WireGuard** protocol.

## 1. Why Tailscale?

* **No Port Forwarding:** It punches through firewalls and NATs effortlessly. You do not need to open ports on your router.
* **WireGuard Based:** It uses modern, high-performance, and secure encryption.
* **Identity Based:** You authenticate using your existing identity provider (Google, Microsoft, GitHub), not separate VPN keys.

---

## 2. Getting Started (The Basics)

Setting up a basic mesh takes less than 5 minutes.

### Step 1: Create an Account

Go to [tailscale.com](https://tailscale.com) and sign up. You will link it to an SSO provider (e.g., your Gmail account).

### Step 2: Install the Client

Install the app on every device you want in the mesh (Windows, macOS, Linux, iOS, Android).

* **Linux Command:** `curl -fsSL https://tailscale.com/install.sh | sh`

### Step 3: Authenticate

Open the app on the device and log in. The device is now part of your **Tailnet**.

* **MagicDNS:** Tailscale automatically assigns a readable domain name to every device (e.g., `server-pc` or `iphone`). You can ping or SSH into `server-pc` without knowing its IP address.

---

## 3. Power User Features

Once your devices are connected, you can unlock the real power of Tailscale with these configurations:

| Feature | Description | Best Use Case |
| --- | --- | --- |
| **Exit Nodes** | Routes *all* your internet traffic through a specific device on your network. | Safe browsing on public Coffee Shop Wi-Fi; appearing to be at "home" while traveling. |
| **Subnet Routers** | Allows access to devices on your LAN that *cannot* run Tailscale (printers, IoT, legacy servers). | Accessing your entire home LAN remotely without installing the app on every single device. |
| **Taildrop** | A generic "AirDrop" for any platform. | Sending a file from your iPhone to your Linux server instantly. |

### Setting up an Exit Node (Linux Example)

If you want your Linux server to act as an Exit Node:

1. **Enable IP Forwarding** on the Linux machine.
2. **Advertise the node:**
```bash
sudo tailscale up --advertise-exit-node

```


3. **Approve it:** Go to the Admin Console (web), find the machine, click the **...** menu > **Edit route settings**, and check "Use as exit node."

---

## 4. Access Control Lists (ACLs)

By default, every device in your Tailnet can talk to every other device. To restrict this, you use ACLs (written in JSON in the Admin Console).

**Example: Isolate Servers**
This rule allows you (the admin) to access everything, but prevents your web server from initiating connections to your personal laptop.

```json
{
  "acls": [
    // Admins can access everything
    { "action": "accept", "src": ["group:admin"], "dst": ["*:*"] },
    
    // Servers can only talk to other servers (tagged devices)
    { "action": "accept", "src": ["tag:server"], "dst": ["tag:server:*"] }
  ]
}

```

> **Note:** Tailscale uses "Tags" (e.g., `tag:prod`) to manage permissions for servers, rather than user identities. This prevents a service from losing access if you delete a specific user account.

---

## 5. Maintenance & Best Practices

* **Key Expiry:** By default, authentication keys expire every 180 days for security.
* *Fix:* In the Admin Console, click the **...** on your servers/headless devices and select **"Disable Key Expiry"** so you don't get locked out remotely.


* **Tailscale SSH:** You can enable this to allow SSH access between Tailscale devices without managing local SSH keys on the machine. The ACLs handle the authentication.
* **Battery Life:** On mobile, Tailscale is very efficient, but if you don't need access to home services, toggling it off can save marginal battery.

# Tailscale Subnet Router with pi-hole
- **A. Enable IP Forwarding:** 
```bash
# 1. Enable IPv4 forwarding immediately
sudo sysctl -w net.ipv4.ip_forward=1

# 2. Make the change permanent across reboots
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.conf
```
  - **B. Advertise Subnet Routes:**
  ```bash
  # Advertise the 192.168.0.0/24 subnet
sudo tailscale up --advertise-routes=192.168.0.0/24
  ```
- **C. Approve Route in Tailscale Admin
	- Tailscale Admin page
	- Machines->router node->edit route settings
	- Approve previously advertised subnet
	D. **Overview:** 
	- PI set as nameserver
	- client queries `service.domain.me` 
	- PI responds services LAN IP
	- Since subnet route is approved, remote client traffic auto resolved through tailscale using pi as subnet router