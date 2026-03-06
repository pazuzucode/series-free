# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Docker Compose media server stack for automated downloading and streaming of movies and series. The stack includes torrent downloading, media management, indexers, and streaming servers.

## Commands

```bash
# Start all services
docker-compose up -d

# Stop all services
docker-compose down

# Restart a specific service
docker-compose restart <service_name>

# View logs
docker-compose logs -f <service_name>

# Pull latest images
docker-compose pull
```

## Architecture

The stack consists of the following services:

| Service | Port | Purpose |
|---------|------|---------|
| qbittorrent | 8011 | Torrent downloader |
| prowlarr | 9696 | Indexer/RSS aggregator |
| sonarr | 8989 | TV series management |
| radarr | 7878 | Movie management |
| jellyfin | 8096 | Media server |
| jellyseerr | 5055 | Request management UI |

## First-Time Setup

After the first run, you MUST configure the following in the web UI:

### Prowlarr Configuration (http://localhost:9696)

1. Go to **Settings > Indexers**
2. Enable the indexers you want to use (some require API keys)
3. Test each indexer to verify it works

### Radarr Configuration (http://localhost:7878)

1. Go to **Settings > Download Clients**
2. Add qBittorrent with these settings:
   - Host: `qbittorrent`
   - Port: `8011`
   - Username: `admin`
   - Password: (your password from .env)
   - Category: leave empty (to monitor all downloads)
3. Go to **Settings > Download Client > Remote Path Mappings**
   - Add: Host `qbittorrent`, Remote Path `/downloads/`, Local Path `/downloads/`
4. Go to **Settings > Indexers**
   - Enable indexers connected via Prowlarr (they should appear as "Prowlarr" type)
5. Test each indexer

### Sonarr Configuration (http://localhost:8989)

Same as Radarr but for TV series.

### qBittorrent Configuration (http://localhost:8011)

1. Go to **Settings > Downloads**
2. Set "Default Save Path": `/downloads/`
3. Add categories (optional):
   - Category: `radarr`, Save path: `/downloads/radarr`
   - Category: `sonarr`, Save path: `/downloads/sonarr`

## Important Configuration Notes

**Remote Path Mappings are CRITICAL** - Without them, Radarr/Sonarr cannot find downloaded files.

- qBittorrent downloads to `/downloads/` (inside container)
- Radarr monitors `/downloads/` and imports to `/data/movies/`
- The mapping tells Radarr: "when qBittorrent says /downloads/, it's /downloads/ inside the container"

**DNS is pre-configured** via init scripts in `config/*/custom-cont-init.d/`

## Data Structure

Persistent data is stored at `${ARRPATH}` (default: `/media/`):
- Downloads folder: `${ARRPATH}downloads/`
- Movies: `${ARRPATH}radarr/movies/`
- TV Shows: `${ARRPATH}sonarr/tvshows/`
- Config: `${ARRPATH}<service>/config/`

## Access URLs

- qbittorrent: http://localhost:8011
- prowlarr: http://localhost:9696
- sonarr: http://localhost:8989
- radarr: http://localhost:7878
- jellyfin: http://localhost:8096
- jellyseerr: http://localhost:5055
