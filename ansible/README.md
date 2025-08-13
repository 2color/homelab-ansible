# HP EliteDesk Homelab Ansible Configuration

Ansible playbooks to configure your HP EliteDesk 800 G4 Mini as a homelab server with Docker, Cockpit, Portainer, SABnzbd, and Plex.

## Prerequisites

- Ubuntu Server 24.04 installed on HP EliteDesk
- SSH key access configured
- Ansible installed on control machine

## Quick Start

1. **Install required Ansible roles:**
   ```bash
   ansible-galaxy install -r requirements.yml
   ```

2. **Update inventory with your server IP:**
   ```bash
   # Edit inventories/homelab/hosts.yml
   # Replace 192.168.1.100 with your HP EliteDesk IP address
   ```

3. **Configure variables (optional):**
   ```bash
   # Edit inventories/homelab/group_vars/all.yml
   # Set timezone and Plex claim token if needed
   ```

4. **Run the playbook:**
   ```bash
   ansible-playbook playbooks/site.yml
   ```

## Services Installed

After successful deployment, these services will be available:

- **Cockpit**: https://YOUR_IP:9090 - System management web interface
- **Portainer**: http://YOUR_IP:9000 - Docker container management
- **SABnzbd**: http://YOUR_IP:8080 - Usenet downloader
- **Plex**: http://YOUR_IP:32400/web - Media server

## Directory Structure

```
/opt/docker-data/
├── plex/
│   ├── config/          # Plex configuration
│   └── transcode/       # Temporary transcoding files
├── sabnzbd/
│   ├── config/          # SABnzbd configuration
│   ├── downloads/       # Completed downloads (shared with Plex as /media)
│   └── incomplete/      # Incomplete downloads
└── portainer_data/      # Portainer configuration (Docker volume)
```

## Configuration

### Plex Setup
1. Get a claim token from https://www.plex.tv/claim/
2. Add it to `inventories/homelab/group_vars/all.yml`:
   ```yaml
   plex_claim_token: "claim-XXXXXXXXXXXXXXXXXXXX"
   ```

### SABnzbd Categories
Configure SABnzbd to organize downloads into subdirectories that Plex can recognize:
- Movies: `movies/`
- TV Shows: `tv/`
- Music: `music/`

### Hardware Transcoding
The Plex container includes `/dev/dri` device mapping for Intel Quick Sync hardware transcoding support on the i5-8500.

## Troubleshooting

### Check container status
```bash
ansible homelab-1 -m shell -a "docker ps"
```

### View logs
```bash
ansible homelab-1 -m shell -a "docker logs plex"
ansible homelab-1 -m shell -a "docker logs sabnzbd"
```

### Restart services
```bash
ansible homelab-1 -m shell -a "docker restart plex sabnzbd portainer"
```