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

## Running Specific Roles

You can run individual roles instead of the entire playbook:

### Single Role
```bash
# Run only the Docker role
ansible-playbook playbooks/site.yml --tags docker

# Run only the Plex role  
ansible-playbook playbooks/site.yml --tags plex

# Run only the SABnzbd role
ansible-playbook playbooks/site.yml --tags sabnzbd

# Run only the Portainer role
ansible-playbook playbooks/site.yml --tags portainer

# Run only the Cockpit role
ansible-playbook playbooks/site.yml --tags cockpit

# Run only the ytdl-sub role
ansible-playbook playbooks/site.yml --tags ytdl-sub

# Run only the dashboard role
ansible-playbook playbooks/site.yml --tags homelab-dashboard
```

### Multiple Roles
```bash
# Run only media-related services
ansible-playbook playbooks/site.yml --tags "plex,sabnzbd,ytdl-sub"

# Run only management interfaces
ansible-playbook playbooks/site.yml --tags "cockpit,portainer,homelab-dashboard"
```

### Skip Specific Roles
```bash
# Run everything except Plex
ansible-playbook playbooks/site.yml --skip-tags plex

# Skip multiple roles
ansible-playbook playbooks/site.yml --skip-tags "plex,sabnzbd"
```

**Note:** Make sure to run the `docker` role first if running individual service roles, as they depend on Docker being installed.

## Services Installed

After successful deployment, these services will be available:

- **Cockpit**: https://YOUR_IP:9090 - System management web interface
- **Portainer**: http://YOUR_IP:9000 - Docker container management
- **SABnzbd**: http://YOUR_IP:8080 - Usenet downloader
- **Plex**: http://YOUR_IP:32400/web - Media server
- **ytdl-sub**: http://YOUR_IP:8443 - YouTube downloader with subscription management
- **Dashboard**: http://YOUR_IP:80 - Service dashboard with links to all services

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
├── ytdl-sub/
│   ├── config/          # ytdl-sub configuration
│   ├── tv_shows/        # Downloaded TV shows
│   ├── movies/          # Downloaded movies
│   ├── music/           # Downloaded music
│   └── music_videos/    # Downloaded music videos
├── caddy/
│   └── html/            # Dashboard HTML files
└── portainer_data/      # Portainer configuration (Docker volume)
```

## Configuration

### Port Customization
Each service uses configurable port variables defined in their respective role defaults. To change a service port:

1. Edit the role's `defaults/main.yml` file:
   - `roles/cockpit/defaults/main.yml` - `cockpit_port: 9090`
   - `roles/portainer/defaults/main.yml` - `portainer_port: 9000`
   - `roles/sabnzbd/defaults/main.yml` - `sabnzbd_port: 8080`
   - `roles/plex/defaults/main.yml` - `plex_port: 32400`
   - `roles/ytdl-sub/defaults/main.yml` - `ytdl_sub_port: 8443`

2. Re-run the playbook to apply changes

The dashboard automatically uses these port variables and will update when ports change.

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
ansible homelab-1 -m shell -a "docker restart plex sabnzbd portainer ytdl-sub homelab-dashboard"
```