# How to Install and Connect AWX Receptor to AWX

[![Ansible](https://img.shields.io/badge/Ansible-EE0000?style=for-the-badge&logo=ansible&logoColor=white)](https://www.ansible.com/)
[![AWX](https://img.shields.io/badge/AWX-EE0000?style=for-the-badge&logo=ansible&logoColor=white)](https://github.com/ansible/awx)
[![Receptor](https://img.shields.io/badge/Receptor-EE0000?style=for-the-badge&logo=ansible&logoColor=white)](https://github.com/ansible/receptor)

## What is Ansible Receptor?

**Ansible Receptor** is AWX's execution node service that enables distributed job execution across your infrastructure. Instead of running all Ansible jobs on your AWX server, Receptor nodes handle the workload while AWX orchestrates and monitors execution.

**Benefits:**
- Scale job execution across multiple nodes
- Isolate workloads in different network segments
- Reduce load on your AWX control plane
- Execute jobs closer to target systems

## What This Playbook Does

This playbook fully automates Receptor node deployment:

1. **Creates AWX execution instance** via API
2. **Sets up instance group** and associates the node
3. **Downloads install bundle** from AWX
4. **Installs and configures** Receptor service
5. **Deploys TLS certificates** for secure communication
6. **Verifies** service is running correctly

## Quick Start

### 1. Set Required Variables

```yaml
# Required configuration
awx_api_host: "your-awx-server.com"
awx_token: "your-awx-api-token"
awx_instance_hostname: "receptor-node-01" or "192.168.1.1"
awx_instance_group_name: "production-nodes"
```

### 2. Run the Playbook

```bash
ansible-playbook -i inventory install_Bundle_Receptor.yml
```

## Configuration Variables

## Configuration Variables

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `awx_api_host` | Your AWX server hostname (used to build API URL) | ✅ | — |
| `awx_token` | AWX API token with admin privileges | ✅ | — |
| `download_dir` | Temporary path for storing the install bundle | ❌ | `/tmp/awx_bundle` |
| `receptor_dir` | Destination directory for receptor configuration and certificates | ❌ | `/etc/receptor` |
| `awx_instance_hostname` | DNS Name or IP for this receptor node | ✅ |  |
| `awx_instance_group_name` | Name of the AWX instance group to create/join | ✅ | |
| `awx_instance_peers_listener_port` | TCP port where receptor listens | ❌ | `27199` |
| `enabled` | Whether the AWX execution instance is enabled | ❌ | `true` |
| `awx_instance_peers_managed_by_policy` | Whether the instance is managed by AWX policies | ❌ | `true` |
| `awx_instance_peers_peers_from_control_nodes` | Allow control nodes to peer automatically | ❌ | `true` |


## Prerequisites

- Ubuntu/RHEL/CentOS system with sudo access
- Network connectivity requirements:
  - **AWX Server** → **Receptor Node** on TCP port `27199`
  - **Receptor Node** → **AWX Server** on TCP port `443` (HTTPS API)
- Valid AWX API token with **Write** permissions

## Usage Example

**Single node with inventory:**
```ini
[receptor_nodes]
node1 ansible_host=10.0.1.100 awx_instance_hostname=receptor-node-01

[receptor_nodes:vars]
awx_api_host=awx.company.com
awx_instance_hostname=dnsname_or_ip
awx_instance_group_name=production-nodes
awx_token=your-secure-token
```

```bash
ansible-playbook -i inventory Install_AWX_Receptor.yaml
```

## Troubleshooting

**Check service status:**
```bash
sudo systemctl status receptor
sudo journalctl -u receptor -f
```

# 🔗 Verify AWX-Receptor Connectivity

## Test 1: Receptor → AWX API
**From Receptor Node:**
```bash
curl -k https://your-awx-host/api/v2/ping/
```

## Test 2: AWX → Receptor Port
**From AWX Server:**
```bash
telnet your-receptor-node 27199
```

---

**Replace:**
- `your-awx-host` → AWX server IP/hostname
- `your-receptor-node` → Receptor node IP/hostname

**Common issues:**
- Ensure AWX token has admin privileges
- Check firewall allows port 27199
- Verify DNS resolution between AWX and receptor nodes

## Architecture

```
AWX Control Plane → Receptor Node → Target Systems
    (Port 443)        (Port 27199)      (Various)
```

After installation, your Receptor node will appear in AWX under **Administration > Instances** and be ready to execute jobs.
