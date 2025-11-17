# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a homelab configuration repository for an HP EliteDesk 800 G4 Mini server. The infrastructure is managed through Ansible playbooks that configure Docker-based services including:

- **Media Services**: Plex, Jellyfin, SABnzbd, ytdl-sub
- **Monitoring Stack**: Prometheus, Grafana, Alertmanager, node-exporter, cAdvisor
- **Management Tools**: Portainer, Cockpit, custom homelab dashboard
- **Network Services**: Tailscale mesh VPN, Samba file sharing
- **Storage**: ZFS-based backup storage, SSD data storage, Synology NAS integration

### Infrastructure Stack

- **Base OS**: Ubuntu Server 24.04 on HP EliteDesk 800 G4 Mini
- **Container Runtime**: Docker managed via Ansible
- **Storage**: ZFS pool for backups, ext4 SSD for active data
- **Network**: Tailscale mesh VPN, UFW firewall
- **Remote Management**: Intel vPro/AMT via MeshCommander
- **System Management**: Cockpit web interface (port 9090)
- **Container Management**: Portainer (port 9000)

### Service Architecture

#### Media Services

- **Plex Media Server** (port 32400): Intel Quick Sync hardware transcoding, media streaming
- **Jellyfin** (port 8096): Alternative media server with SSD-mounted media library
- **SABnzbd** (port 8080): Usenet downloader with shared media directory
- **ytdl-sub** (port 8443): YouTube content downloader and organizer

#### Monitoring Stack

- **Prometheus** (port 9091): Metrics collection and time-series database
- **Grafana** (port 3000): Visualization and dashboards (default admin/admin)
- **Alertmanager** (port 9093): Alert routing and management
- **node-exporter** (port 9100): Host metrics exporter
- **cAdvisor** (port 8081): Container metrics exporter

#### Management and Utilities

- **Homelab Dashboard** (port 80): Custom landing page with service links
- **Portainer** (port 9000): Docker container management UI
- **Cockpit** (port 9090): System administration interface
- **Samba**: File sharing service (\\homelab-1\docker-data)


## Configuration

### Ansible Inventory

- Default target: `homelab-1` at 192.168.1.36 (update in `ansible/inventories/homelab/hosts.yml`)
- SSH user: `daniel` with sudo privileges
- SSH key authentication (no password prompts)
- Access via hostname `homelab-1` for Tailscale compatibility

### Key Configuration Files

- `ansible/inventories/homelab/group_vars/all.yml`: Service configuration including Plex claim token
- `ansible/inventories/homelab/group_vars/vault.yml`: Encrypted secrets (Tailscale auth key, etc.)
- `ansible/ansible.cfg`: Ansible behavior and connection settings
- `ansible/playbooks/site.yml`: Main deployment playbook with all roles

### Ansible Roles (in deployment order)

1. `geerlingguy.docker` - Docker installation and configuration
2. `backup-data-zfs` - ZFS pool setup for backups
3. `data-ssd` - SSD storage mount configuration
4. `backup-storage` - Automated ZFS snapshots and replication
5. `samba` - File sharing service
6. `docker` - Docker daemon customization
7. `ufw` - Firewall configuration
8. `cockpit` - System management interface
9. `portainer` - Container management UI
10. `plex` - Plex media server
11. `sabnzbd` - Usenet downloader
12. `jellyfin` - Alternative media server
13. `ytdl-sub` - YouTube downloader
14. `node-exporter` - Host metrics exporter
15. `cadvisor` - Container metrics exporter
16. `prometheus` - Metrics collection
17. `alertmanager` - Alert management
18. `grafana` - Monitoring dashboards
19. `homelab-dashboard` - Custom landing page
20. `synology-nas` - NAS integration
21. `tailscale` - Mesh VPN

### Hardware Features

- Intel i5-8500 CPU with Quick Sync support for hardware transcoding
- vPro/AMT remote management accessible via MeshCommander
- Dual M.2 NVMe slots for storage expansion
- Multiple SATA ports for additional storage

### Network Security

- UFW firewall enabled by default
- Services accessible over Tailscale VPN
- Samba configured to allow access over Tailscale
- All services use hostname resolution for Tailscale compatibility

## Recent Changes

- Added Grafana monitoring with pre-configured dashboards
- Integrated Prometheus metrics collection for all services
- Configured ZFS backup pool with automated snapshots
- Consolidated media directory structure
- Enabled UFW firewall for network security
- Added Jellyfin as alternative media server
- Integrated Synology NAS via NFS mounts
