# Complete Setup Guide: Home Assistant, Zigbee2MQTT (Z2M), & Sonoff MG24

---

This guide covers the end-to-end installation for a self-hosted Docker environment (Linux), specifically configured for the Sonoff Dongle Plus MG24 (EFR32MG24 chip) and the ThirdReality ZL1 Color Bulb.
## 1: Hardware Preparation
 * Identify the Dongle:
   The "m24" refers to the Sonoff ZBDongle-MG24. This uses the newer Silicon Labs EFR32MG24 chip.
   * Note: Unlike the older "P" model (Texas Instruments), this dongle uses the Ember (EZSP) driver in Zigbee2MQTT.
 * Connect to Server:
   * Plug the dongle into a USB 2.0 port (or use a USB extension cable to reduce interference from USB 3.0 ports).
   * Run the following command to find your device ID:
     `ls -l /dev/serial/by-id/`

   * Copy the output that looks like usb-ITEAD_SONOFF_Zigbee_.... You will need this for the Docker Compose file.
## 2: Directory Structure
Create a clean folder structure to keep your configuration persistent and organized.
`mkdir -p ~/homelab/smarthome/{homeassistant,zigbee2mqtt/data,mosquitto/config,mosquitto/data,mosquitto/log}`

##  3: Configuration Files
You need to create two specific config files before starting the containers.
1. Mosquitto Configuration
Create the file ~/homelab/smarthome/mosquitto/config/mosquitto.conf:
`nano ~/homelab/smarthome/mosquitto/config/mosquitto.conf`

Paste the following (allows anonymous access for local LAN simplicity, see notes for security):
```persistence true
persistence_location /mosquitto/data/
log_dest file /mosquitto/log/mosquitto.log
listener 1883
```
## Allow anonymous for internal local network setup
`allow_anonymous true`

2. Zigbee2MQTT Configuration
Create the file ~/homelab/smarthome/zigbee2mqtt/data/configuration.yaml:
`nano ~/homelab/smarthome/zigbee2mqtt/data/configuration.yaml`

Paste the following (Update the port line with your specific ID from Phase 1):
# Home Assistant Integration
`homeassistant: true`

# MQTT Settings
```mqtt:
  base_topic: zigbee2mqtt
  server: 'mqtt://mosquitto:1883'
  ``` 

# Serial Settings for Sonoff MG24 (Ember Driver)
```
serial:
  port: /dev/serial/by-id/usb-ITEAD_SONOFF_Zigbee_3.0_USB_Dongle_Plus_V2_... # <--- PASTE YOUR ID HERE
  adapter: ember
  ```

# Frontend Settings (GUI)
```
frontend:
  port: 8080
  ```
  
# Zigbee Network
`permit_join: false` 

## 4: Docker Compose
Create your docker-compose.yml file in ~/homelab/smarthome/:
`nano ~/homelab/smarthome/docker-compose.yml`

Paste the following content:
```
services:
  # MQTT Broker
  mosquitto:
    container_name: mosquitto
    image: eclipse-mosquitto
    restart: unless-stopped
    ports:
      - "1883:1883"
      - "9001:9001"
    volumes:
      - ./mosquitto/config:/mosquitto/config
      - ./mosquitto/data:/mosquitto/data
      - ./mosquitto/log:/mosquitto/log

  # Zigbee2MQTT
  zigbee2mqtt:
    container_name: zigbee2mqtt
    image: koenkk/zigbee2mqtt
    restart: unless-stopped
    depends_on:
      - mosquitto
    volumes:
      - ./zigbee2mqtt/data:/app/data
      - /run/udev:/run/udev:ro
    environment:
      - TZ=America/Chicago # Update to your timezone
    devices:
      # Map the USB dongle directly
      - /dev/serial/by-id/usb-ITEAD_SONOFF_Zigbee_3.0_USB_Dongle_Plus_V2_...:/dev/ttyACM0 # <--- UPDATE THIS LINE
    ports:
      - "8080:8080" # Web Interface

  # Home Assistant
  homeassistant:
    container_name: homeassistant
    image: lscr.io/linuxserver/homeassistant:latest
    network_mode: host # Critical for Home Assistant discovery
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Chicago
    volumes:
      - ./homeassistant:/config
    restart: unless-stopped
    depends_on:
      - mosquitto
      - zigbee2mqtt
``` 
Launch the stack:
`docker compose up -d`

## 5: Pairing the ThirdReality Bulb (ZL1)
Once Docker is running, open the Zigbee2MQTT interface in your browser (http://<server-ip>:8080).
 * Enable Joining: Click the Permit Join (All) button in the top bar.
 * Reset/Pair Bulb:
   * Screw in the ThirdReality ZL1 bulb.
   * Power Cycle 5 Times: Turn the switch OFF and ON 5 times quickly (approx. 1 second per toggle).
     * Sequence: ON -> OFF -> ON -> OFF -> ON -> OFF -> ON -> OFF -> ON.
   * Visual Confirmation: The bulb will flash a sequence of colors (Warm White -> Cool White -> Red -> Green -> Blue) and then stay solid Warm White.
 * Rename: Once it appears in Zigbee2MQTT, click the blue "Edit" icon to give it a friendly name (e.g., "Office_Lamp"). Make sure "Update Home Assistant entity ID" is checked.
## 6: Connect Home Assistant
 * Open Home Assistant (http://<server-ip>:8123) and complete the onboarding if new.
 * Navigate to Settings > Devices & Services.
 * Home Assistant should auto-discover "MQTT". If not, click Add Integration > MQTT.
 * Configuration:
   * Broker: `localhost` (since HA is on host network) or the server IP.
   * Port: `1883`.
   * No username/password needed (unless you changed allow_anonymous in Phase 3).
 * Once connected, your "Office_Lamp" will automatically appear in Home Assistant under the MQTT integration.

