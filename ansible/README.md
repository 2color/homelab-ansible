# HP EliteDesk Homelab Ansible Configuration

Ansible playbooks to configure your HP EliteDesk 800 G4 Mini as a homelab server with Docker, Cockpit, Portainer, SABnzbd, and Plex.

- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Running Specific Roles](#running-specific-roles)
  - [Single Role](#single-role)
  - [Multiple Roles](#multiple-roles)
  - [Skip Specific Roles](#skip-specific-roles)
- [Services Installed](#services-installed)
- [Directory Structure](#directory-structure)
- [Configuration](#configuration)
  - [Port Customization](#port-customization)
  - [Plex Setup](#plex-setup)
  - [SABnzbd Categories](#sabnzbd-categories)
  - [Hardware Transcoding](#hardware-transcoding)
- [Ansible Vault](#ansible-vault)
  - [Creating a Vault File](#creating-a-vault-file)
  - [Editing an Existing Vault](#editing-an-existing-vault)
  - [Viewing Vault Contents](#viewing-vault-contents)
  - [Setting Up Vault Password File](#setting-up-vault-password-file)
  - [Running Playbooks with Vault](#running-playbooks-with-vault)
  - [Changing Vault Password](#changing-vault-password)
  - [Common Vault Variables](#common-vault-variables)
- [Troubleshooting](#troubleshooting)
  - [Check container status](#check-container-status)
  - [View logs](#view-logs)
  - [Restart services](#restart-services)

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

### Docker Data Path

All Docker service data is stored in a configurable directory. The default location is `/opt/docker-data`, but this can be changed by modifying the `docker_data_path` variable in `inventories/homelab/group_vars/all.yml`:

```yaml
docker_data_path: /opt/docker-data
```

This path is used by all services for storing configuration, data, and media files.

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

## Ansible Vault

Ansible Vault is used to encrypt sensitive data like API keys, passwords, and tokens. This project uses a vault file at `inventories/homelab/group_vars/vault.yml` to store secrets.

### Creating a Vault File

A template is provided at `inventories/homelab/group_vars/vault.example.yml` showing all required vault variables. To create your vault file:

```bash
# Copy the example file
cp inventories/homelab/group_vars/vault.example.yml inventories/homelab/group_vars/vault.yml

# Encrypt it with your vault password
ansible-vault encrypt inventories/homelab/group_vars/vault.yml
```

Or create a new vault file from scratch:

```bash
ansible-vault create inventories/homelab/group_vars/vault.yml
```

This will:
1. Prompt you to create a vault password (save this securely!)
2. Open your default editor to add encrypted variables
3. Save and encrypt the file when you exit

#### Vault Variables Reference

All variables in `vault.example.yml`:

```yaml
vault_tailscale_auth_key:           # Tailscale auth key for mesh VPN setup
vault_samba_password:               # Samba user password for file sharing
vault_nas_username:                 # Synology NAS username for NFS mounts
vault_nas_password:                 # Synology NAS password for authentication
alertmanager_email_from:            # Email sender address for alerts
alertmanager_email_to:              # Email recipient address for alerts
alertmanager_smtp_smarthost:        # SMTP server and port (e.g., smtp.gmail.com:587)
alertmanager_smtp_auth_username:    # SMTP authentication username
alertmanager_smtp_auth_password:    # SMTP authentication password (use app password for Gmail)
alertmanager_smtp_require_tls:      # Enable TLS for SMTP connection (true/false)
grafana_admin_password:             # Grafana admin user password
```

### Editing an Existing Vault

To edit the encrypted vault file:

```bash
ansible-vault edit inventories/homelab/group_vars/vault.yml
```

You'll be prompted for the vault password, then your editor will open with the decrypted contents.

### Viewing Vault Contents

To view the contents without editing:

```bash
ansible-vault view inventories/homelab/group_vars/vault.yml
```

### Setting Up Vault Password File

This project is configured to automatically use a password file at `.ansible-vault-password` (see `ansible.cfg`). To set this up:

```bash
# Create the password file in the ansible directory
echo "your-vault-password" > .ansible-vault-password

# Secure the file (optional but recommended)
chmod 600 .ansible-vault-password
```

**Important:** The `.ansible-vault-password` file is already in `.gitignore` to prevent accidentally committing your vault password to version control.

### Running Playbooks with Vault

With the `.ansible-vault-password` file configured, playbooks will automatically decrypt vault files:

```bash
# Automatically uses .ansible-vault-password
ansible-playbook playbooks/site.yml
```

If you prefer not to use the password file, you can manually specify the vault password:

**Option 1: Prompt for password**
```bash
ansible-playbook playbooks/site.yml --ask-vault-pass
```

**Option 2: Use a different password file**
```bash
ansible-playbook playbooks/site.yml --vault-password-file /path/to/password-file
```

### Changing Vault Password

To change the vault password:

```bash
ansible-vault rekey inventories/homelab/group_vars/vault.yml
```

### Common Vault Variables

This project typically stores these sensitive variables in the vault:
- `tailscale_auth_key`: Tailscale authentication key for mesh VPN
- `plex_claim_token`: Plex server claim token (optional)
- Any other API keys or credentials needed by services

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