# Homelab Ansible Infrastructure Automation

A complete homelab infrastructure setup using Ansible automation designed for an HP EliteDesk 800 G4 Mini server, but applicable to almost any equivalent hardware. This repository contains everything you need to deploy an Ubuntu-Server-Docker-based homelab with media services, monitoring, and remote access.

- [What's Included](#whats-included)
  - [Core Services](#core-services)
  - [Infrastructure](#infrastructure)
- [Quick Start](#quick-start)
  - [Prerequisites](#prerequisites)
  - [Deployment](#deployment)
- [Hardware Setup](#hardware-setup)
  - [Recommended Hardware](#recommended-hardware)
- [Initial Hardware Setup](#initial-hardware-setup)
  - [1. Configure vPro/AMT Remote Management](#1-configure-vproamt-remote-management)
  - [2. Set Up MeshCommander](#2-set-up-meshcommander)
  - [3. Install Ubuntu Server](#3-install-ubuntu-server)
- [Current Services](#current-services)
- [Ansible Configuration](#ansible-configuration)
  - [Available Roles](#available-roles)
  - [Running Specific Roles](#running-specific-roles)
  - [Directory Structure](#directory-structure)
  - [Service Configuration](#service-configuration)
  - [Ansible Vault](#ansible-vault)
  - [Troubleshooting](#troubleshooting)
  - [Remote Management Notes](#remote-management-notes)
- [Remote Access](#remote-access)
  - [Tailscale Mesh VPN](#tailscale-mesh-vpn)
  - [Dynamic DNS (Alternative Approach)](#dynamic-dns-alternative-approach)
- [Architecture \& Design](#architecture--design)
  - [Design Philosophy](#design-philosophy)
  - [Alternative Platforms](#alternative-platforms)
- [Storage Architecture](#storage-architecture)
  - [Interplay with Synology NAS](#interplay-with-synology-nas)
  - [Samba Configuration for macOS Compatibility](#samba-configuration-for-macos-compatibility)
- [Backups](#backups)
  - [Backup Storage](#backup-storage)
  - [ZFS Backup Storage Implementation](#zfs-backup-storage-implementation)
- [Additional Resources](#additional-resources)
  - [Official Documentation](#official-documentation)
  - [Community Discussions](#community-discussions)
  - [Inspiration \& Guides](#inspiration--guides)
  - [Storage Expansion Options](#storage-expansion-options)

## What's Included

### Core Services

- **Media Streaming**: Plex Media Server and Jellyfin
- **Downloads**: SABnzbd (Usenet) and ytdl-sub (YouTube)
- **Monitoring**: Prometheus, Grafana, and Alertmanager
- **Management**: Portainer (containers), Cockpit (system)
- **Remote Access**: Tailscale mesh VPN
- **File Sharing**: Samba server with macOS optimization
- **Storage**: ZFS backup pool with snapshots

### Infrastructure

- **Base OS**: Ubuntu Server 24.04
- **Automation**: Ansible playbooks for configuration
- **Containers**: Docker with hardware transcoding support
- **Storage**: ZFS mirror on USB DAS, NVMe SSDs, Synology NAS integration
- **Security**: UFW firewall, Tailscale VPN, secure credentials via Ansible Vault

## Quick Start

### Prerequisites

- HP EliteDesk 800 G4 Mini (or similar hardware)
- Ubuntu Server 24.04 installed
- SSH access configured with key authentication
- Local machine with Ansible installed

### Deployment

1. **Clone this repository**

   ```bash
   git clone <repository-url> homelab-ansible
   cd homelab-ansible
   ```

2. **Install required Ansible roles**

   ```bash
   cd ansible
   ansible-galaxy install -r requirements.yml
   ```

3. **Configure inventory**

   Edit `ansible/inventories/homelab/hosts.yml` with your server IP or hostname:

   ```bash
   # Set `ansible_host` with either the IP or network hostname for SSH connection
   ```

4. **Configure variables**

   Edit `ansible/inventories/homelab/group_vars/all.yml`:

   - Set timezone
   - Configure `docker_data_path` (default: `/opt/docker-data`)
   - Add Plex claim token from https://www.plex.tv/claim/ (optional)

5. **Set up secrets**

   ```bash
   cd ansible
   # Copy the example vault file
   cp inventories/homelab/group_vars/vault.example.yml inventories/homelab/group_vars/vault.yml

   # Encrypt it with your vault password
   ansible-vault encrypt inventories/homelab/group_vars/vault.yml
   ```

6. **Update playbook with roles to deploy**

   Edit `ansible/playbooks/site.yml` to enable/disable specific roles as needed.

7. **Deploy everything**

   ```bash
   cd ansible
   ansible-playbook playbooks/site.yml --ask-vault-pass
   ```

8. **Access your services**
   - Access the dashboard at `http://<server-ip>` for links to all services

## Hardware Setup

### Recommended Hardware

**HP EliteDesk 800 G4 Mini**

- **CPU**: Intel Core i5-8500 (with Quick Sync for transcoding)
- **Memory**: 16GB RAM
- **Storage**: 256GB M.2 SSD + 2TB M.2 SSD
- **Remote Management**: Intel vPro/AMT support
- **Form Factor**: mini PC

**Storage Expansion**

- **Terramaster D4-320**: USB 3.2 DAS enclosure
- **Configuration**: 2x drives in ZFS mirror for backups

## Initial Hardware Setup

### 1. Configure vPro/AMT Remote Management

Follow this [video guide](https://www.youtube.com/watch?v=VcqZ7D9CNg0&t=182s):

- Enable Intel Management Engine (ME) and AMT in BIOS
- Reset ME password by selecting "Unconfigure AMT on Next Boot"
- Update password from default `admin` in Intel Management Engine BIOS Extension
- Configure DHCP and reserve IP address in router

### 2. Set Up MeshCommander

- Install: `npm install -g meshcommander`
- Connect to reserved IP address
- Use AMT password from previous step
- Remote control and power management now available

### 3. Install Ubuntu Server

**Note**: ISO mounting via MeshCommander IDER doesn't work reliably. Use USB instead.

- Download [Ubuntu Server 24.04 ISO](https://ubuntu.com/download/server)
- Create bootable USB with [Etcher](https://etcher.balena.io/)
- Boot from USB and follow installation prompts
- **Important**: Do not enable LVM for simplicity
- Import SSH key from GitHub during setup (recommended)

## Current Services

After deployment, access the homelab dashboard at `http://<server-ip>` (port 80) for links to all configured services including Plex, Grafana, Portainer, Cockpit, and more.

## Ansible Configuration

### Available Roles

The following roles are available for deployment. Enable or disable roles in [`ansible/playbooks/site.yml`](ansible/playbooks/site.yml) based on your needs:

| Role                 | Description                                                      |
| -------------------- | ---------------------------------------------------------------- |
| `geerlingguy.docker` | Installs and configures Docker container runtime                 |
| `backup-data-zfs`    | Sets up ZFS mirror pool for backup storage on USB DAS            |
| `data-ssd`           | Configures ext4 and mounts second SSD storage                    |
| `backup-directory`   | Creates directory structure for Restic backup storage            |
| `samba`              | Configures file sharing with macOS optimization                  |
| `docker`             | Customizes Docker daemon configuration                           |
| `ufw`                | Configures UFW firewall rules                                    |
| `cockpit`            | Installs system management web interface (port 9090)             |
| `portainer`          | Deploys container management UI (port 9000)                      |
| `plex`               | Sets up Plex media server with hardware transcoding (port 32400) |
| `sabnzbd`            | Configures Usenet downloader (port 8080)                         |
| `jellyfin`           | Deploys alternative open-source media server (port 8096)         |
| `ytdl-sub`           | Sets up YouTube content downloader and organizer (port 8443)     |
| `node-exporter`      | Installs host metrics exporter for Prometheus (port 9100)        |
| `cadvisor`           | Deploys container metrics exporter (port 8081)                   |
| `prometheus`         | Sets up metrics collection and time-series database (port 9091)  |
| `alertmanager`       | Configures alert routing and management (port 9093)              |
| `grafana`            | Deploys monitoring dashboards and visualization (port 3000)      |
| `homelab-dashboard`  | Creates custom landing page with service links (port 80)         |
| `synology-nas`       | Configures Synology NAS integration via SMB mounts               |
| `tailscale`          | Sets up mesh VPN for secure remote access                        |

**Note:** Roles are typically deployed in the order listed above. The `geerlingguy.docker` role must be run before any containerized services.

### Running Specific Roles

You can run individual roles instead of the entire playbook for targeted deployments:

#### Single Role

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

# Run only the Tailscale role
ansible-playbook playbooks/site.yml --tags tailscale

# Run only the NAS mounting role
ansible-playbook playbooks/site.yml --tags nas
```

#### Multiple Roles

```bash
# Run only media-related services
ansible-playbook playbooks/site.yml --tags "plex,sabnzbd,ytdl-sub"

# Run only management interfaces
ansible-playbook playbooks/site.yml --tags "cockpit,portainer,homelab-dashboard"
```

#### Skip Specific Roles

```bash
# Run everything except Plex
ansible-playbook playbooks/site.yml --skip-tags plex

# Skip multiple roles
ansible-playbook playbooks/site.yml --skip-tags "plex,sabnzbd"
```

**Note:** Make sure to run the `docker` role first if running individual service roles, as they depend on Docker being installed.

### Directory Structure

All Docker service data is stored in `/opt/docker-data` (configurable via `docker_data_path` variable):

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

### Service Configuration

#### Docker Data Path

All Docker service data is stored in a configurable directory. The default location is `/opt/docker-data`, but this can be changed by modifying the `docker_data_path` variable in `ansible/inventories/homelab/group_vars/all.yml`:

```yaml
docker_data_path: /opt/docker-data
```

This path is used by all services for storing configuration, data, and media files.

#### Port Customization

Each service uses configurable port variables defined in their respective role defaults. To change a service port:

1. Edit the role's `defaults/main.yml` file:

   - `roles/cockpit/defaults/main.yml` - `cockpit_port: 9090`
   - `roles/portainer/defaults/main.yml` - `portainer_port: 9000`
   - `roles/sabnzbd/defaults/main.yml` - `sabnzbd_port: 8080`
   - `roles/plex/defaults/main.yml` - `plex_port: 32400`
   - `roles/ytdl-sub/defaults/main.yml` - `ytdl_sub_port: 8443`

2. Re-run the playbook to apply changes

The dashboard automatically uses these port variables and will update when ports change.

#### Plex Setup

1. Get a claim token from https://www.plex.tv/claim/
2. Add it to `ansible/inventories/homelab/group_vars/all.yml`:
   ```yaml
   plex_claim_token: 'claim-XXXXXXXXXXXXXXXXXXXX'
   ```

#### SABnzbd Categories

Configure SABnzbd to organize downloads into subdirectories that Plex can recognize:

- Movies: `movies/`
- TV Shows: `tv/`
- Music: `music/`

#### Hardware Transcoding

The Plex container includes `/dev/dri` device mapping for Intel Quick Sync hardware transcoding support on the i5-8500.

### Ansible Vault

Ansible Vault is used to encrypt sensitive data like API keys, passwords, and tokens. This project uses a vault file at `ansible/inventories/homelab/group_vars/vault.yml` to store secrets.

#### Creating a Vault File

A template is provided at `ansible/inventories/homelab/group_vars/vault.example.yml` showing all required vault variables. To create your vault file:

```bash
cd ansible

# Copy the example file
cp inventories/homelab/group_vars/vault.example.yml inventories/homelab/group_vars/vault.yml

# Encrypt it with your vault password
ansible-vault encrypt inventories/homelab/group_vars/vault.yml
```

Or create a new vault file from scratch:

```bash
cd ansible
ansible-vault create inventories/homelab/group_vars/vault.yml
```

This will:

1. Prompt you to create a vault password (save this securely!)
2. Open your default editor to add encrypted variables
3. Save and encrypt the file when you exit

#### Vault Variables Reference

All variables in `vault.example.yml`:

```yaml
vault_tailscale_auth_key: # Tailscale auth key for mesh VPN setup
vault_samba_password: # Samba user password for file sharing
vault_nas_username: # Synology NAS username for NFS mounts
vault_nas_password: # Synology NAS password for authentication
alertmanager_email_from: # Email sender address for alerts
alertmanager_email_to: # Email recipient address for alerts
alertmanager_smtp_smarthost: # SMTP server and port (e.g., smtp.gmail.com:587)
alertmanager_smtp_auth_username: # SMTP authentication username
alertmanager_smtp_auth_password: # SMTP authentication password (use app password for Gmail)
alertmanager_smtp_require_tls: # Enable TLS for SMTP connection (true/false)
grafana_admin_password: # Grafana admin user password
```

#### Editing an Existing Vault

To edit the encrypted vault file:

```bash
cd ansible
ansible-vault edit inventories/homelab/group_vars/vault.yml
```

You'll be prompted for the vault password, then your editor will open with the decrypted contents.

#### Viewing Vault Contents

To view the contents without editing:

```bash
cd ansible
ansible-vault view inventories/homelab/group_vars/vault.yml
```

#### Setting Up Vault Password File

This project is configured to automatically use a password file at `ansible/.ansible-vault-password` (see `ansible.cfg`). To set this up:

```bash
cd ansible

# Create the password file
echo "your-vault-password" > .ansible-vault-password

# Secure the file (recommended)
chmod 600 .ansible-vault-password
```

**Important:** The `.ansible-vault-password` file is already in `.gitignore` to prevent accidentally committing your vault password to version control.

#### Running Playbooks with Vault

With the `.ansible-vault-password` file configured, playbooks will automatically decrypt vault files:

```bash
cd ansible
# Automatically uses .ansible-vault-password
ansible-playbook playbooks/site.yml
```

If you prefer not to use the password file, you can manually specify the vault password:

**Option 1: Prompt for password**

```bash
cd ansible
ansible-playbook playbooks/site.yml --ask-vault-pass
```

**Option 2: Use a different password file**

```bash
cd ansible
ansible-playbook playbooks/site.yml --vault-password-file /path/to/password-file
```

#### Changing Vault Password

To change the vault password:

```bash
cd ansible
ansible-vault rekey inventories/homelab/group_vars/vault.yml
```

### Troubleshooting

#### Check container status

```bash
ansible homelab-1 -m shell -a "docker ps"
```

#### View logs

```bash
ansible homelab-1 -m shell -a "docker logs plex"
ansible homelab-1 -m shell -a "docker logs sabnzbd"
ansible homelab-1 -m shell -a "docker logs grafana"
```

#### Restart services

```bash
ansible homelab-1 -m shell -a "docker restart plex sabnzbd portainer ytdl-sub homelab-dashboard"
```

### Remote Management Notes

**AMT/MeshCommander Tips:**

- AMT allows remote power control and BIOS access without a physical connection
- Remote desktop requires a screen connected (or DisplayPort dummy plug)
- Can power on system and access BIOS remotely without a monitor
- DisplayPort emulator enables remote desktop via AMT without a physical display

## Remote Access

### Tailscale Mesh VPN

The homelab uses Tailscale for secure remote access without exposing services to the public internet.

#### Features Enabled

- **Tailscale SSH**: Secure SSH access without local SSH keys or port forwarding
- **DNS Integration**: Accept DNS settings from Tailscale for easy service discovery
- **Route Acceptance**: Can access other devices on the Tailscale network
- **Hostname**: Uses inventory hostname (`homelab-1`) for easy identification

#### Configuration

- **Auth Key**: Stored securely in `vault_tailscale_auth_key` (Ansible vault)
- **Username**: Configurable via `samba_username` variable (default: `<your-username>`)
- **Password**: Stored securely in `samba_password` (Ansible vault)
- **Installation**: Managed by Ansible role in the main playbook

#### Setup Steps

1. Get auth key from https://login.tailscale.com/admin/settings/keys
2. Add to vault: `ansible-vault edit ansible/inventories/homelab/group_vars/vault.yml`
3. Deploy: `cd ansible && ansible-playbook playbooks/site.yml --tags tailscale --ask-vault-pass`

#### Benefits Over Port Forwarding

- No router configuration required
- End-to-end encryption for all traffic
- Works from any network (coffee shops, hotels, etc.)
- No dynamic DNS setup needed
- Built-in device authentication

### Dynamic DNS (Alternative Approach)

Alternative to Tailscale for remote access using traditional port forwarding:

- Use a dynamic DNS service (e.g., DuckDNS) for consistent domain name
- Configure router to update DNS with current public IP
- [DDNS comparison discussion](https://www.reddit.com/r/selfhosted/comments/1g8s0sh/seeking_reliable_free_dynamic_dns_noip_vs_duckdns/)
- [Cloudflare DDNS solution](https://github.com/timothymiller/cloudflare-ddns) for custom domains
- Detect public IP: <https://www.cloudflare-cn.com/cdn-cgi/trace>

## Architecture & Design

### Design Philosophy

- **Docker-First**: Containers for most services
- **Simplicity**: Avoid over-engineering with complex virtualization
- **Resource Efficiency**: Balance between isolation and performance
- **Automation**: Everything managed through Ansible

### Alternative Platforms

- **Proxmox**: VM/container management, snapshots, ZFS support, clustering
- **Unraid**: Combined NAS and container/VM platform
- **Coolify**: Self-hosted Heroku alternative
- **Portainer vs Yacht**: Mature orchestration vs beginner-friendly templates
- **Other Options**: CapRover, Piku, Dokku

## Storage Architecture

### Interplay with Synology NAS

**Hardware:** 2x 3TB drives (RAID) with Synology DS216play NAS

**Connection Methods:**

- NFS/SMB network shares (recommended)
- Direct USB connection not supported (self-contained system)

#### Synology NAS Mounting (Ansible Role)

The homelab includes an automated Synology NAS mounting solution using systemd mount units for reliable network storage access.

**Role Features:**

- **Security-First**: SMB 3.0 protocol with NTLMv2 authentication
- **Systemd Integration**: Mount units instead of fstab for better logging and management
- **Performance Optimized**: 1MB buffers and network resilience options
- **Health Monitoring**: Built-in health checks and backup share validation

**Configuration:**

- **Mount Points**: `/mnt/nas/backup` and `/mnt/nas/media`
- **Credentials**: Stored securely in Ansible vault (`vault_nas_username`, `vault_nas_password`)
- **Monitoring**: Use `journalctl -u mnt-nas-backup.mount` for detailed logs

**Deployment:**

```bash
# Deploy NAS mounts only
cd ansible && ansible-playbook playbooks/site.yml --tags nas --ask-vault-pass

# Check mount status
ansible homelab-1 -m shell -a "systemctl status mnt-nas-backup.mount"

# Run health checks
cd ansible && ansible-playbook playbooks/site.yml --tags health-check --ask-vault-pass
```

**Mount Options (Security & Performance):**

- SMB 3.0 protocol (`vers=3.0`)
- Large network buffers (1MB read/write)
- Connection keepalive (`echo_interval=60`)
- Dynamic user context (uses actual ansible user UID/GID)
- **Encryption Disabled**: `seal` option removed to significantly improve transfer speeds

### Samba Configuration for macOS Compatibility

The homelab includes a Samba file server that provides access to multiple storage locations with different access policies.

The default shares are configured in [main.yml](./ansible/roles/samba/defaults/main.yml).

#### Network Access

- **Local Network:** 192.168.1.0/24
- **Tailscale VPN:** 100.64.0.0/10 (CGNAT range)
- **All Other Access:** Denied

#### macOS-Specific Samba Settings

To ensure compatibility with macOS clients, the following settings are configured:

```ini
# Disable DFS (Distributed File System) to fix hostname parsing errors
host msdfs = no

# Improve macOS Finder compatibility with Apple-specific metadata handling
vfs objects = catia fruit streams_xattr

# Store macOS metadata in filesystem extended attributes (not ._ files)
fruit:metadata = stream

# Identify as MacSamba for better macOS client recognition
fruit:model = MacSamba
```

#### Common macOS Connection Issues

**Error: "The operation can't be completed because the original item for 'docker-data' can't be found"**

This typically indicates:

1. **Directory permissions issue** - The shared directory may not have proper read permissions
2. **Guest access misconfiguration** - Anonymous access may not be properly enabled
3. **Path resolution problem** - The share path may not be accessible to the samba process

**Troubleshooting Steps:**

1. **Verify directory exists and has proper permissions:**

   ```bash
   ls -la /opt/docker-data/
   # Should show readable permissions for the samba process
   ```

2. **Check Samba service status:**

   ```bash
   systemctl status smbd nmbd
   ```

3. **Test connection from macOS terminal:**

   ```bash
   # Test SMB connection
   smbutil view //<your-server-ip>

   # Mount share manually
   mkdir ~/mnt/homelab
   mount -t smbfs //guest@<your-server-ip>/docker-data ~/mnt/homelab
   ```

4. **Check Samba logs for specific errors:**
   ```bash
   tail -f /var/log/samba/log.smbd
   ```

**DFS Hostname Parsing Error Fix:**

If you see errors like "parse_dfs_path_strict: can't parse hostname from path", the `host msdfs = no` setting should resolve this by disabling DFS functionality that can conflict with macOS SMB clients.

#### Connection Methods

**From macOS Finder:**

- Press `Cmd+K` and enter one of:
  - `smb://<your-server-ip>/docker-data` (guest read-only or authenticated read-write)
  - `smb://<your-server-ip>/data` (requires authentication)
  - `smb://<your-server-ip>/backup-data` (requires authentication)
- For explicit guest access: `smb://guest@<your-server-ip>/docker-data`

**From macOS Terminal:**

```bash
# List available shares
smbutil view //<your-server-ip>

# Mount docker-data with guest access (read-only)
mount -t smbfs //guest@<your-server-ip>/docker-data /path/to/mount/point

# Mount any share with authentication (read-write)
mount -t smbfs //daniel@<your-server-ip>/docker-data /path/to/mount/point
mount -t smbfs //daniel@<your-server-ip>/data /path/to/mount/point
mount -t smbfs //daniel@<your-server-ip>/backup-data /path/to/mount/point
```

## Backups

### Backup Storage

Currently using ZFS mirror pool for backup storage and Restic as backup tool.

**Comparison Resources:**

- [Restic vs Borg Discussion](https://github.com/restic/restic/issues/1875)
- [Borg or Restic Analysis](https://stickleback.dk/borg-or-restic/)
- [2025 Backup Tool Comparison](https://mangohost.net/blog/duplicacy-vs-restic-vs-borg-which-backup-tool-is-right-in-2025/)
- [Community Comparison Discussion](https://forum.duplicati.com/t/big-comparison-borg-vs-restic-vs-arq-5-vs-duplicacy-vs-duplicati/9952)

### ZFS Backup Storage Implementation

The homelab uses ZFS for the backup storage system with a Terramaster D4-320 USB 3.2 DAS (Direct Attached Storage) enclosure containing two disks configured in a mirror configuration.

#### Backup Hardware

- **DAS Enclosure:** Terramaster D4-320 USB 3.2
- **Connection:** USB 3.2 connection to HP EliteDesk
- **Disks:** Two drives in ZFS mirror configuration
- **Disk Identification:**
  - Disk 1: `/dev/disk/by-id/usb-TerraMas_TDAS_ZVY0CGK5-0:0` (sda)
  - Disk 2: `/dev/disk/by-id/usb-TerraMas_TDAS_ZVY0CGPW-0:0` (sdb)

#### ZFS Pool Configuration

**Pool Name:** `backup-data`
**Pool Type:** Mirror (provides redundancy - can survive one disk failure)
**Mount Point:** `/backup-data`

**Pool-Level Properties:**

- `ashift=12`: Optimized for 4K physical sectors (modern HDDs/SSDs)
- `autoexpand=on`: Automatically expand pool when larger disks are installed
- `autoreplace=off`: Manual control over disk replacement

## Additional Resources

### Official Documentation

- [HP EliteDesk 800 G4 Support](https://support.hp.com/gb-en/drivers/hp-elitedesk-800-65w-g4-desktop-mini-pc/21353734)
- [MeshCommander AMT Management](https://www.meshcommander.com/)

### Community Discussions

- [Proxmox vs Alternatives](https://www.reddit.com/r/homelab/comments/1h54vhg/what_are_the_pros_and_cons_for_choosing_proxmox/)
- [Open Source Backup Tools](https://www.reddit.com/r/Backup/comments/1gszsvi/list_of_free_open_source_and_crossplatform_backup/)

### Inspiration & Guides

- [Perfect Media Server](https://perfectmediaserver.com/) - Comprehensive homelab media server guide

### Storage Expansion Options

- [Terramaster D4-320 USB 3.2 DAS Review](https://www.youtube.com/watch?v=ZdEqEWiA2CE)
- [HP EliteDesk 800 G4 4-Drive Setup](https://www.reddit.com/r/homelab/comments/1e913vb/hp_elitedesk_800_g4_mini_the_ultimate_4drive_setup/)
- [3D Printed NAS Enclosure for Mini PCs](https://makerworld.com/en/models/1399535-thinknas-4x-hdd-nas-enclosure-for-lenovo-m920q#profileId-1589394)
- [Stuffing 4x SSDs in HP EliteDesk](https://www.reddit.com/r/homelab/comments/1hnniwe/stuffing_4x_ssds_in_a_hp_elitedesk_800_g4_micro/)
