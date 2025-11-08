# Ansible Role: nfs

Generic NFS server and client configuration role for Linux systems.

## Description

This role provides comprehensive NFS (Network File System) configuration supporting both server and client modes. It handles installation, configuration, and management of NFS exports and mounts.

## Requirements

- Ansible 2.14+
- Target systems: Debian/Ubuntu, RHEL/CentOS
- Network connectivity between NFS server and clients

## Role Variables

See the collection's main README.md for comprehensive variable documentation and examples.

### Key Variables

```yaml
# Server mode
enable_nfs_server: false
nfs_export_paths: []
nfs_allowed_networks: "192.168.0.0/16"

# Client mode
enable_nfs_client: false
nfs_server_host: "nfs-server.example.com"
nfs_mounts: []

# Application ownership
nfs_app_user: "root"
nfs_app_group: "root"
```

## Dependencies

None

## Example Playbook

### NFS Server

```yaml
- hosts: nfs_servers
  become: true
  roles:
    - role: webmarcus.nfs.nfs
      vars:
        enable_nfs_server: true
        nfs_export_paths:
          - path: "/exports/data"
            options: "rw,sync,no_subtree_check"
```

### NFS Client

```yaml
- hosts: nfs_clients
  become: true
  roles:
    - role: webmarcus.nfs.nfs
      vars:
        enable_nfs_client: true
        nfs_server_host: "nfs-server.local"
        nfs_mounts:
          - src: "nfs-server.local:/exports/data"
            dest: "/mnt/data"
            opts: "rw,sync,hard"
```

## License

MIT

## Author

Marcus Hernandez (marcus@webmarcus.com)
