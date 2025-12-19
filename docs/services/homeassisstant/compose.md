
```yaml
### --- Smart Home Stack: MQTT Broker, Zigbee2MQTT, Home Assistant --- ###

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
      - /dev/serial/by-id/usb-youthings_Zigbee_Adapter-if00
      - "8082:8080" # Web Interface

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