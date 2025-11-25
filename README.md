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
  - [Alternative Hardware](#alternative-hardware)
- [Initial Hardware Setup](#initial-hardware-setup)
  - [1. Configure vPro/AMT Remote Management](#1-configure-vproamt-remote-management)
  - [2. Set Up MeshCommander](#2-set-up-meshcommander)
  - [3. Install Ubuntu Server](#3-install-ubuntu-server)
- [Current Services](#current-services)
  - [Core Infrastructure](#core-infrastructure)
  - [Media Services](#media-services)
  - [Dashboard](#dashboard)
- [Ansible Configuration](#ansible-configuration)
  - [Remote Management Notes](#remote-management-notes)
- [Remote Access](#remote-access)
  - [Tailscale Mesh VPN](#tailscale-mesh-vpn)
  - [Dynamic DNS (Alternative Approach)](#dynamic-dns-alternative-approach)
- [Architecture \& Design](#architecture--design)
  - [Design Philosophy](#design-philosophy)
  - [Alternative Platforms](#alternative-platforms)
- [Storage Architecture](#storage-architecture)
  - [Interplay with Synology NAS](#interplay-with-synology-nas)
  - [File Sharing for Mac Clients](#file-sharing-for-mac-clients)
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

2. **Configure inventory**

   - Edit `ansible/inventories/homelab/hosts.yml` with your server IP
   - Update variables in `ansible/inventories/homelab/group_vars/all.yml`

3. **Set up secrets**

   ```bash
   cd ansible
   ansible-vault create inventories/homelab/group_vars/vault.yml
   # Add your secrets (see vault.example.yml)
   ```

4. **Deploy everything**

   ```bash
   cd ansible
   ansible-playbook playbooks/site.yml --ask-vault-pass
   ```

5. **Access your services**
   - Dashboard: `http://<server-ip>`
   - Grafana: `http://<server-ip>:3000` (admin/admin)
   - Portainer: `http://<server-ip>:9000`
   - See full service list in the [Current Services](#current-services) section

## Hardware Setup

### Recommended Hardware

**HP EliteDesk 800 G4 Mini**

- **CPU**: Intel Core i5-8500 (with Quick Sync for transcoding)
- **Memory**: 16GB RAM
- **Storage**: 256GB M.2 SSD + 1TB M.2 SSD
- **Remote Management**: Intel vPro/AMT support
- **Form Factor**: Compact mini PC

**Storage Expansion**

- **Terramaster D4-320**: USB 3.2 DAS enclosure
- **Configuration**: 2x drives in ZFS mirror for backups
- **Optional**: Synology NAS for network storage

### Alternative Hardware

**Lenovo ThinkCentre Series**

- M920q: Current generation, USB-C, efficient
- M920x: Enhanced with discrete GPU options
- M720q: Similar specs, older chipset

**Budget Options**

- Chinese mini PCs (Celeron N5095/C150Z)
- Lower power consumption and cost
- Suitable for basic streaming and Docker workloads

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

The homelab runs the following services managed by Ansible:

### Core Infrastructure

- **Docker**: Container runtime (geerlingguy.docker role)
- **Cockpit**: System management web interface (port 9090)
- **Portainer**: Container management interface (port 9000)
- **Tailscale**: Mesh VPN for secure remote access
- **Samba**: File sharing server for `/opt/docker-data/` directory
- **Synology NAS**: Network-attached storage mounts via systemd units

### Media Services

- **Plex Media Server**: Media streaming with Intel Quick Sync transcoding (port 32400)
- **Jellyfin**: Open-source media server and streaming platform (port 8096)
- **SABnzbd**: Usenet downloader (port 8080)
- **ytdl-sub**: YouTube downloader service (port 8443)

### Dashboard

- **Homelab Dashboard**: Main landing page served by Caddy (port 80)
  - Uses Caddy Alpine container as a static file server
  - HTML generated from Jinja2 template (`index.html.j2`)
  - Template dynamically includes service URLs using Ansible variables
  - Responsive grid layout with hover effects
  - Displays server hostname and IP address
  - Links to all configured services with descriptions

## Ansible Configuration

For detailed Ansible setup instructions, see [Ansible README](./ansible/README.md)

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

- **Docker-First**: Containers for most services, VMs only when necessary
- **Simplicity**: Avoid over-engineering with complex virtualization
- **Resource Efficiency**: Balance between isolation and performance
- **Automation**: Everything managed through Ansible

### Alternative Platforms

**If you're considering other approaches:**

- **Proxmox**: VM/container management, snapshots, ZFS support, clustering (best for experimentation)
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

**Performance Optimization Update:**

- **Encryption Disabled**: Removed `seal` option to significantly improve transfer speeds

### File Sharing for Mac Clients

When sharing files from the homelab to Mac clients, there are several protocol options with different performance characteristics:

#### NFS (Network File System)

- **Performance:** Generally fastest for large file transfers
- **macOS Support:** Native support, but requires manual configuration
- **Setup:** More complex configuration, requires NFS server setup
- **Use Case:** Best for bulk data transfers and server-to-server communication

#### SMB/CIFS (Server Message Block)

- **Performance:** Good performance, especially SMB3+ with modern implementations.
- **macOS Support:** Native Finder integration, easy mounting

#### AFP (Apple Filing Protocol)

- **Performance:** Historically optimized for Mac but now deprecated
- **macOS Support:** Legacy support only (deprecated since macOS 10.9)

#### Performance Considerations

- **macOS Finder Limitations:** Be aware that Finder has known performance issues with network file copies
- **Reference:** [macOS Finder is still bad at network file copies](https://www.jeffgeerling.com/blog/2024/macos-finder-still-bad-network-file-copies)
- **Workaround:** Use command-line tools (`rsync`, `cp`) or third-party file managers for better performance

### Samba Configuration for macOS Compatibility

The homelab includes a Samba file server that shares `/opt/docker-data/` with both guest and authenticated access.

#### Configuration Details

- **Share Name:** `docker-data`
- **Path:** `/opt/docker-data/`
- **Guest Access:** Read-only for anonymous users
- **Authenticated Access:** Read-write for `<your-username>` user
- **Network Restriction:** Limited to your local subnet

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

- Press `Cmd+K` and enter: `smb://<your-server-ip>/docker-data`
- Or: `smb://guest@<your-server-ip>/docker-data` for explicit guest access

**From macOS Terminal:**

```bash
# List available shares
smbutil view //<your-server-ip>

# Mount with guest access
mount -t smbfs //guest@<your-server-ip>/docker-data /path/to/mount/point

# Mount with authentication
mount -t smbfs //<your-username>@<your-server-ip>/docker-data /path/to/mount/point
```

## Backups

### Backup Storage

Currently using ZFS for backup storage infrastructure. The backup software/strategy is still being evaluated.

**Software Options Under Consideration:**

- [Restic](https://github.com/restic/restic) - Modern backup with encryption
- [BorgBackup](https://github.com/borgbackup/borg) - Deduplicating archiver
- [Duplicacy](https://github.com/gilbertchen/duplicacy) - Lock-free deduplication

**Comparison Resources:**

- [Restic vs Borg Discussion](https://github.com/restic/restic/issues/1875)
- [Borg or Restic Analysis](https://stickleback.dk/borg-or-restic/)
- [2025 Backup Tool Comparison](https://mangohost.net/blog/duplicacy-vs-restic-vs-borg-which-backup-tool-is-right-in-2025/)
- [Community Comparison Discussion](https://forum.duplicati.com/t/big-comparison-borg-vs-restic-vs-arq-5-vs-duplicacy-vs-duplicati/9952)

### ZFS Backup Storage Implementation

The homelab uses ZFS for the backup storage system with a Terramaster D4-320 USB 3.2 DAS (Direct Attached Storage) enclosure containing two disks configured in a mirror configuration.

#### Hardware Setup

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

#### ZFS Dataset Properties

The pool uses optimized filesystem properties for backup workloads:

- **compression: lz4** - Fast compression reduces storage usage with minimal CPU overhead
- **atime: off** - Disabled access time updates to reduce write amplification
- **checksum: blake3** - Modern, fast checksum algorithm for data integrity verification
- **primarycache: all** - Cache both metadata and data in ARC (RAM)
- **secondarycache: all** - Use L2ARC if available for extended caching
- **sync: standard** - Standard synchronous write handling
- **recordsize: 128k** - Balanced block size for mixed backup workloads

#### Automated Maintenance

**Monthly ZFS Scrub:**

- Scheduled: First day of each month at 2:00 AM
- Purpose: Verify data integrity and detect silent data corruption
- Command: `zpool scrub backup-data`

**Automatic Import:**

- Systemd service (`zfs-import.service`) ensures the pool is automatically imported at boot
- Runs after `systemd-udev-settle.service` to ensure USB devices are available

#### Benefits of ZFS for Backups

- **Data Integrity:** Blake3 checksums detect silent data corruption
- **Compression:** LZ4 compression reduces storage requirements automatically
- **Redundancy:** Mirror configuration survives single disk failure
- **Snapshots:** Fast, space-efficient snapshots for point-in-time recovery
- **Self-Healing:** Automatically repairs corrupted data using mirror copy
- **Easy Expansion:** Add drives or replace with larger ones seamlessly

#### Design Decisions

- **Mirror vs RAIDZ:** Chose mirror for better random I/O performance and simpler disk replacement
- **USB Connection:** Acceptable for backup workloads; provides flexibility for future expansion
- **Blake3 Checksum:** Modern algorithm offers better performance than SHA256 while maintaining strong data integrity
- **LZ4 Compression:** Provides good compression ratios with negligible CPU overhead, ideal for backup data

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
