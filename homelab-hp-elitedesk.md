# HP EliteDesk 800 G4 Mini Homelab Knowledge Base

## Setup guide

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
- Create a bootable USB drive on Mac with etcher https://etcher.balena.io/
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

### Media Services
- **Plex Media Server**: Media streaming with Intel Quick Sync transcoding (port 32400)
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

### Service URLs
- Dashboard: http://192.168.1.36
- Cockpit: https://192.168.1.36:9090
- Portainer: http://192.168.1.36:9000
- SABnzbd: http://192.168.1.36:8080
- Plex: http://192.168.1.36:32400/web
- ytdl-sub: http://192.168.1.36:8443

## Ansible setup

See other notes.

### notes on AMT and MeshCommander

- AMT (Active Management Technology) allows remote management of the system, including power control.
- Remote desktop works pretty well, but it requires that a screen is connected to the EliteDesk which is a bit of a bummer.
- You can still power on the system and access the BIOS remotely without a screen.
- Solution: Use a dummy HDMI plug to trick the system into thinking a monitor is connected.

## Dynamic DNS

- Use a dynamic DNS service to access the home network remotely with a consistent domain name.
- Setup router to update the dynamic DNS service with the current public IP address.
- DuckDNS seems to be a good option for free dynamic DNS.
- https://www.reddit.com/r/selfhosted/comments/1g8s0sh/seeking_reliable_free_dynamic_dns_noip_vs_duckdns/
- Nice solution with cutsom domains: https://github.com/timothymiller/cloudflare-ddns
- Detect your own public IP address with https://www.cloudflare-cn.com/cdn-cgi/trace

## Hardware Specifications

### HP EliteDesk 800 G4 Mini

**CPU:** Intel Core i5-8500

- Alternative: i5-8500T (lower TDP variant)
  **vPro Support:** Yes (AMT/IDER remote management)
  **Form Factor:** Compact mini PC
  **Memory:** 8GB default, expandable to 32GB or 64GB
  **Storage:** 256GB M.2 SSD with dual M.2 NVMe slots
  **Expandability:** Additional M.2 slot available for storage or Wi-Fi

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

## Backups

Still undecided on the backup strategy, but considering the following options:

- [restic](https://github.com/restic/restic) for backup synchronization
- [BorgBackup](https://github.com/borgbackup/borg)
- Restic vs BorgBackup: https://github.com/restic/restic/issues/1875
- https://stickleback.dk/borg-or-restic/

### ZFS Features that might be useful

It may be nice to try ZFS for the second SSD drive in the HP EliteDesk. Seems a little complex for the boot drive, but could be useful for data storage.

**RAID Support:** ZRAID configurations
**Advanced Features:** Snapshots, compression, deduplication
**Integration:** Available through Proxmox
**Benefits:** Data integrity, flexible storage management

## References

### Documentation

- [HP EliteDesk 800 G4 Support](https://support.hp.com/gb-en/drivers/hp-elitedesk-800-65w-g4-desktop-mini-pc/21353734)
- [MeshCommander AMT Management](https://www.meshcommander.com/)

### Community Resources

- [Proxmox vs Alternatives Discussion](https://www.reddit.com/r/homelab/comments/1h54vhg/what_are_the_pros_and_cons_for_choosing_proxmox/)
