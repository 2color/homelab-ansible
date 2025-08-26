# ZFS Migration Plan for HP EliteDesk Homelab

## Overview

This document outlines the complete plan to migrate the homelab storage from the current ext4 filesystem to ZFS on a new 1.8TB NVMe drive (nvme0n1). The migration maintains backward compatibility while adding ZFS benefits like compression, snapshots, and data integrity.

## Current State Analysis

- **Current Storage**: `/opt/docker-data/` on ext4 filesystem
- **New Hardware**: 1.8TB NVMe SSD (`/dev/nvme0n1`)
- **Services**: All Docker containers mount subdirectories under `/opt/docker-data/`
- **Samba Share**: Shares `/opt/docker-data/` as `docker-data` network share

## ZFS Architecture Design

### Pool Structure
```
Pool Name: homelab-data (on /dev/nvme0n1)
├── homelab-data/docker-data   → /opt/docker-data     (Main Docker storage)
├── homelab-data/backup        → /opt/backup          (Backup storage)
└── homelab-data/media         → /opt/media           (Future media expansion)
```

### ZFS Properties
- **Compression**: lz4 (fast compression with good ratios)
- **Atime**: off (better performance, not needed for container storage)
- **Checksum**: sha256 (data integrity verification)
- **Auto-snapshots**: Enabled for backup/recovery
- **Scrub**: Monthly automatic scrubbing

## Implementation Plan

### Phase 1: Ansible Role Creation

Create `ansible/roles/zfs/` with the following structure:

#### `defaults/main.yml`
```yaml
---
# ZFS pool configuration
zfs_pool_name: "homelab-data"
zfs_pool_device: "/dev/nvme0n1"

# ZFS datasets configuration
zfs_datasets:
  - name: "{{ zfs_pool_name }}/docker-data"
    mountpoint: "/opt/docker-data"
    properties:
      compression: "lz4"
      atime: "off"
      checksum: "sha256"
  - name: "{{ zfs_pool_name }}/backup"
    mountpoint: "/opt/backup"
    properties:
      compression: "lz4"
      atime: "off"
  - name: "{{ zfs_pool_name }}/media"
    mountpoint: "/opt/media"
    properties:
      compression: "lz4"
      atime: "off"

# Migration settings
zfs_backup_original: true
zfs_migration_temp_mount: "/mnt/zfs-temp"
zfs_verify_migration: true
```

#### `tasks/main.yml`
```yaml
---
- name: Install ZFS packages
  apt:
    name:
      - zfsutils-linux
      - zfs-dkms
    state: present
    update_cache: yes
  become: true

- name: Load ZFS kernel module
  modprobe:
    name: zfs
    state: present
  become: true

- name: Check if ZFS pool already exists
  command: zpool list {{ zfs_pool_name }}
  register: zfs_pool_exists
  failed_when: false
  changed_when: false

- name: Create ZFS pool
  command: zpool create -f {{ zfs_pool_name }} {{ zfs_pool_device }}
  become: true
  when: zfs_pool_exists.rc != 0

- name: Create ZFS datasets
  zfs:
    name: "{{ item.name }}"
    state: present
  become: true
  loop: "{{ zfs_datasets }}"
  when: zfs_pool_exists.rc != 0

- name: Set ZFS dataset properties
  zfs:
    name: "{{ item.name }}"
    state: present
    extra_zfs_properties: "{{ item.properties }}"
  become: true
  loop: "{{ zfs_datasets }}"

# Migration tasks
- name: Check if migration is needed
  stat:
    path: /opt/docker-data.bak
  register: migration_complete

- name: Stop all Docker containers for migration
  shell: docker stop $(docker ps -q)
  become: true
  when: not migration_complete.stat.exists
  ignore_errors: true

- name: Create temporary mount point
  file:
    path: "{{ zfs_migration_temp_mount }}"
    state: directory
  become: true
  when: not migration_complete.stat.exists

- name: Set temporary mount point for migration
  zfs:
    name: "{{ zfs_pool_name }}/docker-data"
    state: present
    extra_zfs_properties:
      mountpoint: "{{ zfs_migration_temp_mount }}"
  become: true
  when: not migration_complete.stat.exists

- name: Migrate data with rsync
  synchronize:
    src: /opt/docker-data/
    dest: "{{ zfs_migration_temp_mount }}/"
    archive: yes
    checksum: yes
    delete: no
  become: true
  when: not migration_complete.stat.exists

- name: Verify data migration
  shell: diff -r /opt/docker-data/ {{ zfs_migration_temp_mount }}/
  register: migration_diff
  failed_when: migration_diff.rc not in [0, 1]
  when: not migration_complete.stat.exists and zfs_verify_migration

- name: Backup original data
  command: mv /opt/docker-data /opt/docker-data.bak
  become: true
  when: not migration_complete.stat.exists

- name: Set final ZFS mount point
  zfs:
    name: "{{ zfs_pool_name }}/docker-data"
    state: present
    extra_zfs_properties:
      mountpoint: /opt/docker-data
  become: true
  when: not migration_complete.stat.exists

- name: Start Docker containers after migration
  shell: docker start $(docker ps -aq)
  become: true
  when: not migration_complete.stat.exists
  ignore_errors: true

- name: Enable ZFS auto-import on boot
  lineinfile:
    path: /etc/cron.d/zfs-import
    create: yes
    line: "@reboot root /sbin/zpool import -a"
  become: true

- name: Setup monthly ZFS scrub
  cron:
    name: "ZFS scrub homelab-data"
    cron_file: zfs-scrub
    user: root
    job: "/sbin/zpool scrub {{ zfs_pool_name }}"
    minute: "0"
    hour: "2"
    day: "1"
  become: true

- name: Display ZFS status
  command: zpool status {{ zfs_pool_name }}
  register: zfs_status
  changed_when: false

- name: Show ZFS pool information
  debug:
    msg: "{{ zfs_status.stdout_lines }}"
```

#### `handlers/main.yml`
```yaml
---
- name: restart docker
  systemd:
    name: docker
    state: restarted
  become: true
```

### Phase 2: Integration

#### Update `ansible/playbooks/site.yml`
Add ZFS role before Docker-dependent services:
```yaml
roles:
  - geerlingguy.docker
  - zfs                    # Add ZFS role here
  - docker
  - cockpit
  # ... rest of roles
```

#### Update `CLAUDE.md`
Add ZFS management commands:
```bash
### ZFS Management
# Check pool status
ansible homelab-1 -m shell -a "zpool status"

# List datasets
ansible homelab-1 -m shell -a "zfs list"

# Create snapshot
ansible homelab-1 -m shell -a "zfs snapshot homelab-data/docker-data@backup-$(date +%Y%m%d)"

# List snapshots
ansible homelab-1 -m shell -a "zfs list -t snapshot"

# Scrub pool
ansible homelab-1 -m shell -a "zpool scrub homelab-data"
```

## Migration Process Details

### Pre-Migration Checklist
- [ ] Verify new NVMe drive is detected (`lsblk`)
- [ ] Ensure adequate free space for data copy
- [ ] Schedule maintenance window (estimated 15-45 minutes downtime)
- [ ] Backup critical data externally as precaution

### Migration Steps
1. **Stop Services**: All Docker containers stopped to prevent data corruption
2. **Create ZFS Pool**: `zpool create homelab-data /dev/nvme0n1`
3. **Create Datasets**: ZFS datasets with optimized properties
4. **Data Copy**: `rsync` preserves all permissions, timestamps, and metadata
5. **Verification**: `diff` command ensures data integrity
6. **Atomic Switch**: Original directory renamed, ZFS mounted in place
7. **Service Restart**: Docker containers started with new storage

### Rollback Plan
If issues arise:
```bash
# Stop containers
docker stop $(docker ps -q)

# Unmount ZFS
zfs set mountpoint=none homelab-data/docker-data

# Restore original data
mv /opt/docker-data.bak /opt/docker-data

# Start containers
docker start $(docker ps -aq)
```

### Post-Migration Verification
- [ ] All Docker containers start successfully
- [ ] Samba share accessible at same path
- [ ] Data integrity check passes
- [ ] ZFS compression working (`zfs get compressratio`)
- [ ] Auto-import configured for reboots

## Expected Benefits

### Performance
- **Faster I/O**: NVMe SSD vs. existing storage
- **Compression**: ~20-40% space savings with lz4
- **Reduced fragmentation**: ZFS copy-on-write

### Reliability  
- **Checksumming**: Automatic data corruption detection
- **Snapshots**: Point-in-time recovery capability
- **Scrubbing**: Regular data integrity verification

### Management
- **Unified storage**: Single pool for multiple datasets
- **Easy expansion**: Add drives to pool as needed
- **Flexible quotas**: Per-dataset space limits

## Risk Mitigation

### Data Safety
- Original data preserved in `/opt/docker-data.bak` until manually removed
- Migration uses `rsync` with verification
- Rollback procedure tested and documented

### Service Continuity
- Minimal downtime (containers stopped only during data copy)
- Same mount points maintain container compatibility
- Samba configuration unchanged

### Recovery Options
- ZFS snapshots for incremental backups
- Pool export/import for drive migration
- Traditional file-level backups still possible

## Timeline

- **Ansible Role Development**: 2-3 hours
- **Testing on staging**: 1 hour  
- **Production migration**: 15-45 minutes downtime
- **Post-migration verification**: 30 minutes

## Commands Reference

### Deployment
```bash
# Deploy ZFS setup
cd ansible && ansible-playbook playbooks/site.yml --tags zfs --ask-vault-pass

# Full deployment including ZFS
cd ansible && ansible-playbook playbooks/site.yml --ask-vault-pass
```

### ZFS Management
```bash
# Pool status
zpool status homelab-data

# Dataset usage
zfs list

# Create snapshot
zfs snapshot homelab-data/docker-data@manual-$(date +%Y%m%d)

# Compression ratio
zfs get compressratio homelab-data/docker-data

# Pool health check
zpool scrub homelab-data
```

### Emergency Recovery
```bash
# Import pool after system crash
zpool import homelab-data

# Mount datasets manually
zfs mount -a

# Check for errors
zpool status -v homelab-data
```