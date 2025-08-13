# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a homelab configuration repository for an HP EliteDesk 800 G4 Mini server. The primary infrastructure is managed through Ansible playbooks that configure Docker-based services including Plex media server, SABnzbd, Portainer, and Cockpit system management.

## Common Commands

### Ansible Operations
```bash
# Install required roles
ansible-galaxy install -r ansible/requirements.yml

# Deploy full homelab configuration  
cd ansible && ansible-playbook playbooks/site.yml

# Check container status on remote host
ansible homelab-1 -m shell -a "docker ps"

# View service logs
ansible homelab-1 -m shell -a "docker logs plex"
ansible homelab-1 -m shell -a "docker logs sabnzbd"

# Restart services
ansible homelab-1 -m shell -a "docker restart plex sabnzbd portainer"
```

### MeshCommander (Remote Management)
```bash
# Start MeshCommander for AMT remote management
npx meshcommander
```

## Architecture

### Infrastructure Stack
- **Base OS**: Ubuntu Server 24.04 on HP EliteDesk 800 G4 Mini
- **Container Runtime**: Docker managed via Ansible
- **Remote Management**: Intel vPro/AMT via MeshCommander
- **System Management**: Cockpit web interface (port 9090)
- **Container Management**: Portainer (port 9000)

### Service Architecture
- **Plex Media Server** (port 32400): Intel Quick Sync hardware transcoding enabled
- **SABnzbd** (port 8080): Usenet downloader with shared media directory
- **Data Directory**: `/opt/docker-data/` contains all persistent service data

### Directory Structure
```
/opt/docker-data/
├── plex/config/          # Plex configuration
├── plex/transcode/       # Temporary transcoding files  
├── sabnzbd/config/       # SABnzbd configuration
├── sabnzbd/downloads/    # Completed downloads (shared with Plex as /media)
├── sabnzbd/incomplete/   # Incomplete downloads
└── portainer_data/       # Portainer configuration
```

## Configuration

### Ansible Inventory
- Default target: `homelab-1` at 192.168.1.36 (update in `ansible/inventories/homelab/hosts.yml`)
- SSH user: `daniel` with sudo privileges
- SSH key authentication (no password prompts)

### Key Configuration Files
- `ansible/inventories/homelab/group_vars/all.yml`: Service configuration including Plex claim token
- `ansible/ansible.cfg`: Ansible behavior and connection settings
- `ansible/playbooks/site.yml`: Main deployment playbook

### Hardware Features
- Intel i5-8500 CPU with Quick Sync support for Plex transcoding
- vPro/AMT remote management accessible via MeshCommander
- Dual M.2 NVMe slots for storage expansion