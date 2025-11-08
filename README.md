# Ansible Collection - webmarcus.nfs

Generic NFS server and client configuration for Linux systems.

## Overview

This collection provides a comprehensive NFS role that can configure:
- **NFS Server** - Export directories via NFS
- **NFS Client** - Mount remote NFS shares

Designed to be generic and infrastructure-agnostic with sensible RFC1918 defaults.

## Requirements

- Ansible 2.14+
- Target systems: Debian/Ubuntu, RHEL/CentOS
- Network connectivity between NFS server and clients

## Installation

### From Ansible Galaxy

```bash
ansible-galaxy collection install webmarcus.nfs
```

### From Source (Local Development)

```bash
# Clone this repository
git clone https://github.com/webmarcus/ansible-collection-nfs.git

# Install locally
cd ansible-collection-nfs
ansible-galaxy collection build
ansible-galaxy collection install webmarcus-nfs-*.tar.gz --force
```

## Role Variables

### NFS Server

```yaml
# Enable NFS server
enable_nfs_server: true

# Network allowed to access NFS exports (RFC1918 default)
nfs_allowed_networks: "192.168.0.0/16"

# Export paths
nfs_export_paths:
  - path: "/srv/nfs/shared"
    options: "rw,sync,no_subtree_check"

# NFS server user/group
nfs_server_user: "nfsuser"
nfs_server_group: "nfsgroup"
nfs_server_uid: 2000
nfs_server_gid: 2000
```

### NFS Client

```yaml
# Enable NFS client
enable_nfs_client: true

# NFS server hostname or IP
nfs_server_host: "nfs-server.example.com"

# Mount points
nfs_mounts:
  - src: "{{ nfs_server_host }}:/srv/nfs/shared"
    dest: "/mnt/nfs/shared"
    opts: "rw,sync,hard,intr"

# NFS client user/group (matching server)
nfs_client_user: "nfsuser"
nfs_client_group: "nfsgroup"
nfs_client_uid: 2000
nfs_client_gid: 2000
```

## Usage

### NFS Server Playbook

```yaml
---
- name: Configure NFS Server
  hosts: nfs_servers
  become: true

  vars:
    enable_nfs_server: true
    nfs_allowed_networks: "192.168.0.0/16"
    nfs_export_paths:
      - path: "/data/shared"
        options: "rw,sync,no_subtree_check"

  roles:
    - role: webmarcus.nfs.nfs
```

### NFS Client Playbook

```yaml
---
- name: Configure NFS Client
  hosts: nfs_clients
  become: true

  vars:
    enable_nfs_client: true
    nfs_server_host: "192.168.1.100"
    nfs_mounts:
      - src: "{{ nfs_server_host }}:/data/shared"
        dest: "/mnt/shared"
        opts: "rw,sync,hard,intr"

  roles:
    - role: webmarcus.nfs.nfs
```

### Tailscale Integration Example

Override defaults in `group_vars` for Tailscale mesh networking:

```yaml
# group_vars/nfs_servers.yml
enable_nfs_server: true
nfs_allowed_networks: "100.64.0.0/10"  # Tailscale CGNAT range

# group_vars/nfs_clients.yml
enable_nfs_client: true
nfs_server_host: "{{ groups['nfs_servers'][0] }}"  # Dynamic from inventory
```

## Testing

```bash
# Syntax check
ansible-playbook --syntax-check playbook.yml

# Dry run
ansible-playbook --check playbook.yml

# Deploy
ansible-playbook playbook.yml
```

## Verification

### Server

```bash
# Check exports
exportfs -v

# Check NFS service
systemctl status nfs-kernel-server
```

### Client

```bash
# Check mounts
df -h | grep nfs

# Test mount
showmount -e <server-hostname>
```

## License

MIT

## Author

Marcus Hernandez (marcus@webmarcus.com)
