# [Pi-Hole](https://github.com/pi-hole/pi-hole)

---

The Pi-hole® is a [DNS sinkhole](https://en.wikipedia.org/wiki/DNS_Sinkhole) that protects your devices from unwanted content without installing any client-side software.

# One-Step Automated Install

Those who want to get started quickly and conveniently may install Pi-hole using the following command:

```bash
curl -sSL https://install.pi-hole.net | bash
```

<!-- markdownlint-disable code-block-style -->
!!! info
    [Piping to `bash` is a controversial topic](https://pi-hole.net/2016/07/25/curling-and-piping-to-bash/), as it prevents you from [reading code that is about to run](https://github.com/pi-hole/pi-hole/blob/master/automated%20install/basic-install.sh) on your system.

    If you would prefer to review the code before installation, we provide these alternative installation methods.
<!-- markdownlint-enable code-block-style -->

### Alternative 1: Clone our repository and run

```bash
git clone --depth 1 https://github.com/pi-hole/pi-hole.git Pi-hole
cd "Pi-hole/automated install/"
sudo bash basic-install.sh
```

### Alternative 2: Manually download the installer and run

```bash
wget -O basic-install.sh https://install.pi-hole.net
sudo bash basic-install.sh
```

### Alternative 3: Use Docker to deploy Pi-hole

Please refer to the [Pi-hole docker repo](https://github.com/pi-hole/docker-pi-hole) to use the Official Docker Images.

# Making your network take advantage of Pi-hole

Once the installer has been run, you will need to [configure your router to have **DHCP clients use Pi-hole as their DNS server**](https://discourse.pi-hole.net/t/how-do-i-configure-my-devices-to-use-pi-hole-as-their-dns-server/245) which ensures all devices connected to your network will have content blocked without any further intervention.

If your router does not support setting the DNS server, you can [use Pi-hole's built-in DHCP server](https://discourse.pi-hole.net/t/how-do-i-use-pi-holes-built-in-dhcp-server-and-why-would-i-want-to/3026); just be sure to disable DHCP on your router first (if it has that feature available).

As a last resort, you can manually set each device to use Pi-hole as its DNS server.

## Making your Pi-hole host use Pi-hole

Pi-hole will not be used by the host automatically after installation. To have the host resolve through Pi-hole and your configured blocking lists, you can make the host use Pi-hole as upstream DNS server:

!!! warning
    If your Pi-hole host is using Pi-hole as upstream DNS server and Pi-hole fails, your host loses DNS resolution. This can prevent successful repair attempts, e.g. by `pihole -r` as it needs a working internet connection.

  If your OS uses `dhcpcd` for network configuration, you can add to your `/etc/dhcpcd.conf`

```code
static domain_name_servers=127.0.0.1
```

## Adding your local user to the 'pihole' group

Pi-hole v6 uses a new API for authentication. All CLI commands use this API instead of e.g. direct database manipulation. If a password is set for API access, the CLI commands also need to authenticate. To avoid entering the password everytime on CLI, Pi-hole allows users which are members of the 'pihole' group to authenticate without manually entering the password (this can be disabled by setting `webserver.api.cli_pw` to `false`.)
To add your local user to the 'pihole' group use the following command

For Debian/Ubuntu/Raspberry Pi OS/Armbian/Fedora/CentOS

```code
sudo usermod -aG pihole $USER
```

For Alpine

```code
sudo addgroup pihole $USER
```

# 3. Pi-hole (Local Side)
*Intercepts local requests to route directly to NPM.*
* **Config:** `/etc/dnsmasq.d/99-wildcard.conf`
    ```bash
    address=/.vanth.me/192.168.0.116
    ```
    *(Replace `192.168.0.116` with NPM Host IP)*
* **Activate:** Settings > Misc > **Enable misc.etc_dnsmasq_d**
* **Flush:** Run `pihole restartdns` or `ipconfig /flushdns` on client.

# Tailscale Subnet Router
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