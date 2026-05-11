# Repository Description

A self-hosted media automation stack powered by Docker Compose, featuring Traefik as reverse proxy, Jellyfin for media streaming, Radarr and Prowlarr for content automation, and qBittorrent for downloads.

---

# Media Automation Stack

A fully containerized self-hosted media server stack using Docker Compose.

This project combines:

* Traefik — Reverse proxy and automatic routing
* Jellyfin — Media streaming server
* Radarr — Movie management and automation
* Prowlarr — Indexer manager for the *arr ecosystem
* qBittorrent — Torrent client for automated downloads

## Features

* Docker Compose based deployment
* Automatic reverse proxy configuration with Traefik
* Centralized media management
* Automated movie downloading workflow
* Easy to extend with Sonarr, Bazarr, Overseerr, and more
* Designed for homelab and self-hosted environments

## Stack

| Service     | Purpose            |
| ----------- | ------------------ |
| Traefik     | Reverse proxy      |
| Jellyfin    | Media streaming    |
| Radarr      | Movie automation   |
| Prowlarr    | Indexer management |
| qBittorrent | Torrent downloads  |

## Requirements

* Docker
* Docker Compose
* Linux server / VPS / Homelab machine

## Quick Start

```bash
git clone https://github.com/JosiasTomasnanez/media-automation-stack.git
cd media-automation-stack
docker compose up -d
```

## Access

Services can be accessed locally through Traefik using your server local IP address.

Example:

```text
http://192.168.x.x
```

A custom domain or DNS configuration can also be added later for cleaner routing and remote access.

Example local endpoints:

```text
http://jellyfin.local
http://radarr.local
http://prowlarr.local
http://qbittorrent.local
```

## Project Structure

```text
.
├── docker-compose.yml

```

You can delete everything exept the docker compose file and most directories and persistent volumes are automatically created when running:

```bash
docker compose up -d
```

However, without persistent configuration folders, services would need to be configured again after recreation.


## Notes

This repository is intended for educational, personal, and homelab usage.

Feel free to customize the stack according to your needs.
