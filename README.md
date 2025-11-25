# Homelab Knowledge Base and Ansible Playbooks

This Ansible roles in this repo were written for the HP EliteDesk 800 G4 Mini with two NVMe M.2 SSDs and an external Terramaster D4-320.

## Initial Setup

1. Reset vPro/AMT to factory defaults

- Folowed this guide https://www.youtube.com/watch?v=VcqZ7D9CNg0&t=182s
- Enabled Intel Management Engine (ME) and AMT in BIOS
- Reset ME password by selecting "Unconfigure AMT on Next Boot"
- Go to the Intel Management Engine BIOS Extension and update password from default `admin`
- Leave DHCP and reserve IP address in router

2. Try out MeshCommander for remote management

- Download from npm
- Connect to the reserved IP address of the EliteDesk
- Use the AMT password set in the previous step
- Remote control works like a breeze 🎉

3. ⛔️ Try using MeshCommander to install Ubuntu Server

- Used the "Remote Desktop" feature to mount an ISO
- It doesn't work. Mounting both the `ubuntu-24.04.2-live-server-amd64.iso` and ubuntu `mini.iso` ISOs via MeshCommander IDER feature didn't work.

4. Use a USB drive to install Ubuntu Server

- Download the `ubuntu-24.04.2-live-server-amd64.iso` from the official site
- Create a bootable USB drive on Mac with [etcher](https://etcher.balena.io/)
- Boot the EliteDesk from the USB drive and install Ubuntu Server
- Follow the prompts to set up the server, including network configuration and user accounts.
- Do not enable lvm for simplicity
- Import SSH key from github for easy access (Nice feature)

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

## Ansible setup

See [Ansible README](./ansible/README.md)

### notes on AMT and MeshCommander

- AMT (Active Management Technology) allows remote management of the system, including power control.
- Remote desktop works pretty well, but it requires that a screen is connected to the EliteDesk which is a bit annoying.
- You can still power on the system and access the BIOS remotely without a screen.
- A DisplayPort emulator (dummy plug) to enable remote desktop via AMT without needing a physical monitor connected.

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
- **Username**: Configurable via `samba_username` variable (default: `daniel`)
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

- Use a dynamic DNS service to access the home network remotely with a consistent domain name.
- Setup router to update the dynamic DNS service with the current public IP address.
- DuckDNS seems to be a good option for free dynamic DNS.
- https://www.reddit.com/r/selfhosted/comments/1g8s0sh/seeking_reliable_free_dynamic_dns_noip_vs_duckdns/
- Nice solution with cutsom domains: https://github.com/timothymiller/cloudflare-ddns
- Detect your own public IP address with https://www.cloudflare-cn.com/cdn-cgi/trace

## Hardware Specifications

### HP EliteDesk 800 G4 Mini

**CPU:** Intel Core i5-8500
**vPro Support:** Yes (AMT/IDER remote management)
**Form Factor:** Compact mini PC
**Memory:** 16GB
**Storage:** 256GB M.2 SSD + 1TB M.2 SSD
**Expandability:** Additional smaller M.2 slot available for Wi-Fi

### Remote Management

Intel vPro with Active Management Technology (AMT) provides:

- Headless, out-of-band management over LAN
- Remote control without physical keyboard/screen access
- Recommended tool: [MeshCommander](https://www.meshcommander.com/)

### Design Philosophy

- Docker-first approach
- Emphasis on simplicity over complex virtualization
- Balance between resource efficiency and service isolation

## Alternative Hardware Options

### Lenovo ThinkCentre Series

**M920q:** Current generation with USB-C and improved power efficiency
**M920x:** Enhanced version with discrete GPU options and expanded I/O
**M720q:** Similar to M920q with older chipset features

### Budget Mini PCs

**Chinese Mini PCs (Celeron N5095/C150Z):**

- Lower power consumption and cost
- Reduced performance compared to business-class options
- Suitable for basic media streaming or lightweight Docker workloads

## Operating System Options

### Ubuntu Server

**Primary Choice:** Ubuntu Server
**Automation:** Ansible for provisioning and configuration management
**Use Cases:** Docker deployment, Plex setup, general server automation

### Proxmox Virtual Environment

**Advantages:**

- VM and container management interface
- Snapshot functionality for easy rollbacks
- Clustering support for multi-node setups
- Resource isolation between services
- ZFS integration (ZRAID support)
- Ideal for experimentation and testing

**Best For:** Services requiring VM isolation (e.g., Home Assistant)

## Virtualization Strategy

### Container-First Approach

**Primary Platform:** Docker with management interfaces
**Benefits:** Resource efficiency, simplified deployment
**Limitations:** Some services require OS-level support

### Virtual Machine Use Cases

**Advantages:** Complete isolation, support for special ISOs
**Considerations:** Higher resource overhead, potential for "pet server" anti-patterns
**Recommended For:** Services that don't containerize well

## Container Management Platforms

### Portainer

**Type:** Mature container orchestration platform
**Features:** Kubernetes and Docker Swarm support
**Target:** Advanced users requiring comprehensive management

### Yacht

**Type:** Template-based container management
**Features:** Beginner-friendly interface, media server focus
**Target:** Users wanting simplified deployment

### Alternative Platforms

**Unraid:** Combined NAS and container/VM platform
**Coolify:** Self-hosted Heroku alternative
**Other Options:** CapRover, Piku, Dokku

## System Administration

### Web Management Interfaces

**Ubuntu:** No native web UI (Cockpit available as add-on)
**Proxmox:** Built-in web interface for VM/container management

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
- **Authenticated Access:** Read-write for `daniel` user
- **Network Restriction:** Limited to 192.168.1.x subnet

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
   smbutil view //192.168.1.36

   # Mount share manually
   mkdir ~/mnt/homelab
   mount -t smbfs //guest@192.168.1.36/docker-data ~/mnt/homelab
   ```

4. **Check Samba logs for specific errors:**
   ```bash
   tail -f /var/log/samba/log.smbd
   ```

**DFS Hostname Parsing Error Fix:**

If you see errors like "parse_dfs_path_strict: can't parse hostname from path", the `host msdfs = no` setting should resolve this by disabling DFS functionality that can conflict with macOS SMB clients.

#### Connection Methods

**From macOS Finder:**

- Press `Cmd+K` and enter: `smb://192.168.1.36/docker-data`
- Or: `smb://guest@192.168.1.36/docker-data` for explicit guest access

**From macOS Terminal:**

```bash
# List available shares
smbutil view //192.168.1.36

# Mount with guest access
mount -t smbfs //guest@192.168.1.36/docker-data /path/to/mount/point

# Mount with authentication
mount -t smbfs //daniel@192.168.1.36/docker-data /path/to/mount/point
```

## Backups

Still undecided on the backup strategy, but considering the following options:

- [restic](https://github.com/restic/restic) for backup synchronization
- [BorgBackup](https://github.com/borgbackup/borg)
- Restic vs BorgBackup: https://github.com/restic/restic/issues/1875
- https://stickleback.dk/borg-or-restic/
- Maybe https://github.com/gilbertchen/duplicacy?tab=readme-ov-file
- https://mangohost.net/blog/duplicacy-vs-restic-vs-borg-which-backup-tool-is-right-in-2025/
- https://forum.duplicati.com/t/big-comparison-borg-vs-restic-vs-arq-5-vs-duplicacy-vs-duplicati/9952?u=tophee
- https://forum.duplicacy.com/t/comparison-duplicacy-borg-restic-arq-duplicati/4210/12

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

## References

### Documentation

- [HP EliteDesk 800 G4 Support](https://support.hp.com/gb-en/drivers/hp-elitedesk-800-65w-g4-desktop-mini-pc/21353734)
- [MeshCommander AMT Management](https://www.meshcommander.com/)

### Community Resources

- [Proxmox vs Alternatives Discussion](https://www.reddit.com/r/homelab/comments/1h54vhg/what_are_the_pros_and_cons_for_choosing_proxmox/)
- [Backup tools](https://www.reddit.com/r/Backup/comments/1gszsvi/list_of_free_open_source_and_crossplatform_backup/)

### Inspiration

- https://perfectmediaserver.com/

#### Adding More Drives to the EliteDesk

- [Terramaster D4-320 USB 3.2 DAS Review](https://www.youtube.com/watch?v=ZdEqEWiA2CE)
- [hp_elitedesk_800_g4_mini_the_ultimate_4drive_setup/](https://www.reddit.com/r/homelab/comments/1e913vb/hp_elitedesk_800_g4_mini_the_ultimate_4drive_setup/)
- 3D print an enclosure for the EliteDesk: https://makerworld.com/en/models/1399535-thinknas-4x-hdd-nas-enclosure-for-lenovo-m920q#profileId-1589394
- https://www.reddit.com/r/homelab/comments/1hnniwe/stuffing_4x_ssds_in_a_hp_elitedesk_800_g4_micro/
