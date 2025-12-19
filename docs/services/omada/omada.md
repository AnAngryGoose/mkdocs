# TP-Link Omada Controller: Quick Guide

---

The **Omada Software Defined Networking (SDN)** platform is TP-Link's centralized management solution, similar to Ubiquiti's UniFi. It allows you to manage gateways (routers), switches, and access points (APs) from a single interface.

## 1. Deployment Options

Before you begin, you need to decide how to run the controller. There are three main methods:

| Type | Description | Best For |
| --- | --- | --- |
| **Hardware Controller** | Dedicated physical device (OC200, OC300) powered via PoE. | Plug-and-play simplicity; "set and forget." |
| **Software Controller** | Application installed on a PC/Server (Windows/Linux). | Enthusiasts with existing servers or limited budget. |
| **Cloud-Based** | Fully hosted by TP-Link (Subscription required). | MSPs managing many distinct client sites. |

> **Pro Tip:** For home labs or NAS users, the Docker container `mbentley/omada-controller` is the community standard for running the software controller effortlessly.

---

## 2. Initial Setup & Adoption

Once your controller is running and connected to the network, follow this workflow:

### Step 1: Access the Interface

* **Hardware/Software:** Navigate to the IP address of the controller (usually port `8043` or `8088` depending on setup).
* **Default Login:** Often `admin` / `admin` (you will be forced to change this immediately).

### Step 2: Device Adoption

The controller does not automatically manage devices; you must "Adopt" them.

1. Go to the **Devices** tab (looks like an Access Point icon).
2. You will see your devices listed with a status of **"Pending"**.
3. Click the **Adopt** button on the right.
4. The status will change to **Provisioning**, then **Connected**.

### Step 3: WAN/LAN Configuration

If you are using an Omada Gateway (like the ER605):

* Navigate to **Settings (Gear icon) > Wired Networks > Internet**.
* Configure your WAN type (DHCP, PPPoE, or Static).

---

## 3. Key Configuration Features

### Wireless Networks (SSIDs)

* **Location:** Settings > Wireless Networks.
* **Features:** You can create multiple SSIDs (e.g., Main, IoT, Guest).
* **Guest Network:** Check the "Guest Network" box to automatically apply isolation rules (preventing guests from accessing your internal LAN).

### VLANs (Virtual LANs)

To segment traffic (e.g., separating IoT devices from your PC):

1. **Create Interface:** Settings > Wired Networks > LAN. Create a new Interface with a VLAN ID (e.g., 20).
2. **Tag Ports:** Ensure your managed switches utilize the correct profiles to pass this VLAN tag.
3. **Assign to Wi-Fi:** In Wireless settings, assign an SSID to use that specific VLAN ID.

### ACLs (Access Control Lists)

* **Location:** Settings > Network Security > ACL.
* **Usage:** Use "Switch ACLs" or "Gateway ACLs" to block traffic between VLANs (e.g., Block IoT VLAN from accessing Main VLAN).

---

## 4. Maintenance & Best Practices

* **Backups:** Go to **Settings > Maintenance > Backup & Restore**. Set up "Auto Backup" to save your config to an SD card (OC200) or local disk periodically.
* **Roaming:** Enable **Fast Roaming (802.11r)** in Site Settings to help devices switch between APs smoothly, but test it first (some older IoT devices hate this).
* **AI Optimization:** The "AI WLAN Optimization" tool scans the environment and automatically adjusts channels and transmit power to reduce interference. Run this once after setup.
