# Server Setup 

* **Sources:**
* [Ubuntu Server 24.04 LTS Release Notes](https://ubuntu.com/download/server)
* [Ubuntu Server Network Configuration](https://documentation.ubuntu.com/server/explanation/networking/configuring-networks/)
* [OpenSSH Server Documentation](https://documentation.ubuntu.com/server/how-to/security/openssh-server/)

---

### 1. Preparation & Media Creation

Before touching the hardware, the installation media must be prepared on a separate computer.

**A. Download ISO**
Get the **Ubuntu Server 24.04 LTS** (Long Term Support) ISO. Use whatever OS you want. I just used Ubuntu for this guide.

* **Note:** Do not use the "Desktop" version for a server. The Server ISO is optimized for performance and headless operation (no monitor).

**B. Flash to USB**
Use **BalenaEtcher** or **Rufus** to write the ISO to a USB drive (4GB+).

* **Warning:** This process wipes the USB drive completely.

---

### 2. BIOS / UEFI Configuration

Mini PCs (like the Lenovo M920q) often ship with settings optimized for Windows office use. These must be adjusted for a Linux server.

1. **Enter BIOS:** Insert the USB, power on, and rapidly tap the setup key (usually `F1`, `F2`, or `F12` for Lenovo/Dell/HP).
2. **Secure Boot:** Set to **Disabled**. This ensures compatibility with third-party drivers (like Proxmox or specific NIC drivers) later on.
3. **Power Behavior:** Look for "After Power Loss" or "AC Recovery" and set to **Power On**. This ensures the server reboots automatically after a power outage.
4. **Boot Order:** Move "USB HDD" or your flash drive to the top of the list.

---

### 3. Installation Process

Boot from the USB. The installer (Subiquity) uses a text-based UI. Navigate with `Arrow Keys`, select with `Enter`, and toggle options with `Space`.

**A. General Settings**

* **Language/Keyboard:** Select your defaults.
* **Base Install:** Choose "Ubuntu Server" (standard). The "Minimized" version is too stripped down for a beginner homelab.

**B. Network (Critical Step)**
By default, the server asks for a DHCP address. **Set a Static IP now** to avoid connection issues later.

1. Select your Ethernet interface (e.g., `eth0` or `eno1`).
2. Change "IPv4 Method" from DHCP to **Manual**.
3. **Subnet:** Usually `192.168.1.0/24` (Check your router).
4. **Address:** Pick an IP outside your router's DHCP range (e.g., `192.168.1.50`).
5. **Gateway:** Your router's IP (e.g., `192.168.1.1`).
6. **Name Servers:** `1.1.1.1` (Cloudflare) or `8.8.8.8` (Google).

**C. Storage**

* Select "Use an entire disk".
* **LVM:** Keep "Set up this disk as an LVM group" checked. This allows for flexible partition resizing later.

**D. Profile & SSH**

* **Identity:** Set your hostname (e.g., `vanth-node-01`) and username.
* **SSH Setup:** **Check the box [ ] Install OpenSSH server.**
* *Do not skip this.* Without it, you cannot connect to the headless server remotely.



---

### 4. Post-Install Configuration

Once installed, remove the USB and press Enter to reboot. From this point on, perform all steps via SSH from your main computer.

**A. Connect via SSH**
Open your terminal (or PowerShell on Windows) and connect:

```bash
ssh username@192.168.1.50

```

*(Replace with the IP you set in Step 3B)*.

**B. Update System**
Update the package repositories and upgrade installed packages:

```bash
sudo apt update && sudo apt upgrade -y

```

**C. Verify Static IP (Netplan)**
If you skipped the static IP step during install, you must configure `netplan`.
Edit the config: `sudo nano /etc/netplan/00-installer-config.yaml`

```yaml
network:
  ethernets:
    eno1: # Check your interface name with 'ip a'
      dhcp4: false
      addresses:
        - 192.168.1.50/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [1.1.1.1, 8.8.8.8]
  version: 2

```

Apply changes: `sudo netplan apply`.

---

### 5. Quality of Life & Security

**A. Firewall (UFW)**
Enable the firewall but **allow SSH first** to prevent locking yourself out.

```bash
sudo ufw allow ssh
sudo ufw enable

```

*Response: Command may disrupt existing ssh connections. Proceed with operation (y|n)?* Type `y`.

**B. Prevent Sleep (Laptop/Mini PC Specific)**
Ensure the system doesn't suspend when idle.

```bash
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target

```

---

### Troubleshooting

* **"Permission Denied (publickey)":** You may have enabled "Import SSH Identity" during install but didn't provide a key. Use password authentication initially or add the `-o PubkeyAuthentication=no` flag to debug.
* **No Internet:** Check your gateway settings in `netplan`. Try `ping 8.8.8.8`. If that works but `ping google.com` fails, check your `nameservers`.
* **Fan Noise:** On some Mini PCs (Lenovo Tiny/Dell Micro), fan control packages like `lm-sensors` and `fancontrol` may be needed if the BIOS default is too aggressive.
