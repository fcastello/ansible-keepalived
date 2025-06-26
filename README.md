# Ansible Role: keepalived

A flexible, reusable Ansible role to install and configure Keepalived for high-availability IP failover on Ubuntu 24.04+ systems. Supports multiple VRRP instances, multiple VIPs per instance, and customizable network interfaces.

## Requirements
- Ubuntu 24.04 or later
- Static IP configuration on all participating servers
- Python 3 and Ansible 2.9+

## Role Variables
All configuration is done via the `keepalived_instances` list variable. Each entry defines a VRRP instance with its own interface, VIPs, router ID, priority, authentication, and peers.

| Variable | Description | Required | Example |
|----------|-------------|----------|---------|
| `keepalived_instances` | List of VRRP instance definitions | yes | See below |
| `keepalived_enabled` | Enable keepalived service on boot | no | `true` |

Each item in `keepalived_instances` supports:

| Key | Description | Required | Example |
|-----|-------------|----------|---------|
| `interface` | Network interface to bind | yes | `ens160` |
| `vips` | List of VIPs for this instance | yes | `["192.168.1.100", "192.168.1.101"]` |
| `router_id` | VRRP router ID (unique per instance) | yes | `51` |
| `priority` | Priority for this node (higher = master) | yes | `101` |
| `state` | Initial state (`MASTER` or `BACKUP`) | no | `MASTER` |
| `notify_script` | Script to run on state change | no | `/usr/local/bin/notify.sh` |
| `auth_pass` | Password for VRRP authentication | yes | `mysecretpass` |
| `unicast_peers` | List of peer IPs for this instance | yes | `["192.168.1.102", "192.168.1.103"]` |

## Example Use Cases

### 1. Single VRRP Instance, Single VIP
```yaml
keepalived_instances:
  - interface: "ens160"
    vips:
      - "192.168.1.100"
    router_id: 51
    priority: 101
    state: "MASTER"
    notify_script: "/usr/local/bin/notify.sh"
    auth_pass: "mysecretpass"
    unicast_peers:
      - "192.168.1.102"
      - "192.168.1.103"
```

### 2. Multiple VRRP Instances (Different Interfaces/VIPs)
```yaml
keepalived_instances:
  - interface: "ens160"
    vips:
      - "192.168.1.100"
    router_id: 51
    priority: 101
    state: "MASTER"
    notify_script: "/usr/local/bin/notify.sh"
    auth_pass: "mysecretpass"
    unicast_peers:
      - "192.168.1.102"
      - "192.168.1.103"
  - interface: "ens192"
    vips:
      - "192.168.2.100"
    router_id: 52
    priority: 100
    state: "BACKUP"
    notify_script: ""
    auth_pass: "anotherpass"
    unicast_peers:
      - "192.168.2.101"
      - "192.168.2.102"
```

### 3. Single VRRP Instance, Multiple VIPs
```yaml
keepalived_instances:
  - interface: "ens160"
    vips:
      - "192.168.1.100"
      - "192.168.1.101"
      - "192.168.1.102"
    router_id: 51
    priority: 101
    state: "MASTER"
    notify_script: "/usr/local/bin/notify.sh"
    auth_pass: "mysecretpass"
    unicast_peers:
      - "192.168.1.103"
      - "192.168.1.104"
```

## Example Playbook
```yaml
- hosts: keepalived_nodes
  become: true
  roles:
    - role: keepalived
      vars:
        keepalived_instances:
          - interface: "ens160"
            vips:
              - "192.168.1.100"
            router_id: 51
            priority: 101
            state: "MASTER"
            notify_script: "/usr/local/bin/notify.sh"
            auth_pass: "mysecretpass"
            unicast_peers:
              - "192.168.1.102"
              - "192.168.1.103"
        keepalived_enabled: true
```

## Notify Script Example
A notify script can be used to send alerts or perform actions on state changes. Example:
```bash
#!/bin/bash
STATE=$1
case $STATE in
  MASTER)
    echo "Node is now MASTER" | logger
    ;;
  BACKUP)
    echo "Node is now BACKUP" | logger
    ;;
  FAULT)
    echo "Node is in FAULT state" | logger
    ;;
esac
```

## Galaxy Metadata
See `meta/main.yml` for details. 