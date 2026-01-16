# Overview

This is just a reference for all my project stuff. Most of this is just for personal reference and is a total work in progress but it may be useful to someone, who knows. 

## Current software 
A list of self-hosted services and docker containers I've run/am running. I'm sure a lot is missing from this list, but other pages above have more than what is listed here.

## Table of Contents

- [Media & Streaming](#media--streaming)
- [Automation & Downloaders](#automation--downloaders)
- [Home Automation & IoT](#home-automation--iot)
- [Monitoring & Maintenance](#monitoring--maintenance)
- [Networking & Security](#networking--security)
- [Dashboards & Utilities](#dashboards--utilities)

---

## Media & Sharing

Software for organizing, streaming, and managing media content.

- [Plex](https://www.plex.tv/) - Client-server media player system to stream personal media collections.
- [Samba](https://www.samba.org/) - The standard Windows interoperability suite of programs for Linux and Unix, used here for network file sharing.
- [copyparty](https://github.com/9001/copyparty) - Portable file server with accelerated resumable uploads, deduplication, WebDAV, FTP, zeroconf, media indexer, video thumbnails, audio transcoding, and write-only folders, in a single file with no mandatory dependencies. 
- [Nextcloud](https://github.com/nextcloud) - Access and share your files, calendars, contacts, mail and more from any device, on your terms
- [Immich](https://immich.app/) - Photo and video backup solution directly from your mobile phone.

## Automation & Downloaders

Tools for automating media acquisition and managing download queues.

- [Sonarr](https://sonarr.tv/) - PVR for Usenet and BitTorrent users. It monitors for multiple RSS feeds for new episodes of your favorite shows.
- [Radarr](https://radarr.video/) - A fork of Sonarr but for movies. It monitors feeds for new movies and interfaces with clients and indexers.
- [Prowlarr](https://prowlarr.com/) - An indexer manager/proxy built on the popular *arr .net/react stack to integrate with your various PVR apps.
- [Overseerr](https://overseerr.dev/) - A request management and media discovery tool for the Plex ecosystem.
- [qBittorrent](https://www.qbittorrent.org/) - An open-source software alternative to µTorrent, providing a lightweight BitTorrent client.

## Home Automation & IoT

Services for controlling smart devices and managing local network hardware.

- [Home Assistant](https://www.home-assistant.io/) - Open source home automation that puts local control and privacy first.
- [Zigbee2MQTT](https://www.zigbee2mqtt.io/) - Bridges events and allows you to control your Zigbee devices via MQTT.
- [Mosquitto](https://mosquitto.org/) - An open source (EPL/EDL) message broker that implements the MQTT protocol versions 5.0, 3.1.1 and 3.1.
- [Omada Controller](https://www.tp-link.com/en/business-networking/omada-sdn-controller/) - Management software for TP-Link EAP, JetStream switches, and Omada gateways.

## Monitoring & Maintenance

Tools to track system health, updates, and logs.

- [Beszel](https://github.com/henrygd/beszel) - A lightweight server monitoring hub and agent with historical data, Docker stats, and alerts.
- [Glances](https://nicolargo.github.io/glances/) - A cross-platform system monitoring tool written in Python.
- [Uptime Kuma](https://github.com/louislam/uptime-kuma) - A fancy self-hosted monitoring tool to check uptime for HTTP(s) / TCP / DNS.
- [Scrutiny](https://github.com/AnalogJ/scrutiny) - Hard Drive S.M.A.R.T Monitoring, Historical Trends & Real World Failure Thresholds.
- [Dozzle](https://dozzle.dev/) - Real-time log viewer for Docker containers.
- [Watchtower](https://containrrr.dev/watchtower/) - A process for automating Docker container base image updates.

## Networking & Access

Services handling routing, proxies, and secure access.

- [Nginx Proxy Manager](https://nginxproxymanager.com/) - A simple way to expose your web services securely and manage SSL certificates.
- [Cloudflared](https://github.com/cloudflare/cloudflared) - The command-line client for Cloudflare Tunnel, connecting your local network to the Cloudflare edge.
- [Socket Proxy](https://github.com/Tecnativa/docker-socket-proxy) - A security-enhanced proxy for the Docker socket to control access to the Docker API.

## Dashboards & Utilities

General productivity tools and start pages.

- [Homepage](https://gethomepage.dev/) - A modern, fully static, fast, secure, and highly customizable application dashboard.
- [IT-Tools](https://it-tools.tech/) - A collection of handy tools for developers and IT people.
- [Bento4 / BentoPDF](https://github.com/axiom-media/bento4) - Likely a suite of tools for processing media or PDF files (Implied from container name).
- [CouchDB](https://couchdb.apache.org/) - A database that uses JSON for documents, JavaScript for MapReduce indexes, and regular HTTP for an API. (Used for Obsidian LiveSync or similar backends).
