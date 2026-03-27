# AWX Receptor

[![CI](https://github.com/kevinmagni/awx-receptor-install/actions/workflows/ci.yml/badge.svg)](https://github.com/kevinmagni/awx-receptor-install/actions/workflows/ci.yml)
[![Ansible Galaxy](https://img.shields.io/badge/galaxy-kevinmagni.awx__receptor-blue.svg)](https://galaxy.ansible.com/kevinmagni/awx_receptor)
[![License](https://img.shields.io/github/license/kevinmagni/awx-receptor-install)](LICENSE)

An Ansible role that fully automates the deployment of **AWX Receptor** execution nodes, including AWX API registration, package installation, TLS certificate deployment, and service verification.

## What is Ansible Receptor?

**Ansible Receptor** is AWX's execution node service that enables distributed job execution across your infrastructure. Instead of running all Ansible jobs on your AWX server, Receptor nodes handle the workload while AWX orchestrates and monitors execution.

**Benefits:**
- Scale job execution across multiple nodes
- Isolate workloads in different network segments
- Reduce load on your AWX control plane
- Execute jobs closer to target systems

## What This Role Does

1. **Creates AWX execution instance** via the AWX API
2. **Creates/joins instance group** and associates the node
3. **Downloads the install bundle** from AWX
4. **Installs receptor and dependencies** (supports Debian and RHEL families)
5. **Deploys TLS certificates** for secure communication
6. **Configures and verifies** the receptor service

## Requirements

- Ansible >= 2.14
- Target: Ubuntu 20.04+, Debian 11+, RHEL/CentOS 8+
- Network connectivity:
  - **AWX Server** -> **Receptor Node** on TCP `27199`
  - **Receptor Node** -> **AWX Server** on TCP `443` (HTTPS API)
- Valid AWX API token with **Write** permissions

## Installation

```bash
ansible-galaxy install kevinmagni.awx_receptor
```

Or add to `requirements.yml`:

```yaml
roles:
  - name: kevinmagni.awx_receptor
```

## Role Variables

| Variable | Required | Default | Description |
|----------|:--------:|---------|-------------|
| `awx_receptor_api_host` | yes | — | AWX server hostname |
| `awx_receptor_api_token` | yes | — | AWX API token with admin privileges |
| `awx_receptor_instance_hostname` | yes | — | DNS name or IP for this receptor node |
| `awx_receptor_instance_group_name` | yes | — | AWX instance group to create/join |
| `awx_receptor_validate_certs` | no | `false` | Validate TLS certificates for AWX API calls |
| `awx_receptor_instance_listener_port` | no | `27199` | TCP port for receptor listener |
| `awx_receptor_instance_enabled` | no | `true` | Whether the AWX execution instance is enabled |
| `awx_receptor_instance_managed_by_policy` | no | `true` | Whether the instance is managed by AWX policies |
| `awx_receptor_instance_peers_from_control_nodes` | no | `true` | Allow control nodes to peer automatically |
| `awx_receptor_dir` | no | `/etc/receptor` | Receptor configuration directory |
| `awx_receptor_download_dir` | no | `/tmp/awx_bundle` | Temporary directory for install bundle |
| `awx_receptor_log_level` | no | `debug` | Receptor service log level |
| `awx_receptor_extra_packages` | no | `[]` | Additional packages to install |
| `awx_receptor_cleanup_bundle` | no | `true` | Remove downloaded bundle after installation |

## Usage

### Basic Playbook

```yaml
- hosts: receptor_nodes
  become: true
  roles:
    - role: kevinmagni.awx_receptor
      awx_receptor_api_host: "awx.company.com"
      awx_receptor_api_token: "{{ vault_awx_token }}"
      awx_receptor_instance_hostname: "receptor-node-01"
      awx_receptor_instance_group_name: "production-nodes"
```

### Inventory Example

```ini
[receptor_nodes]
node1 ansible_host=10.0.1.100 awx_receptor_instance_hostname=receptor-node-01
node2 ansible_host=10.0.1.101 awx_receptor_instance_hostname=receptor-node-02

[receptor_nodes:vars]
awx_receptor_api_host=awx.company.com
awx_receptor_api_token=your-secure-token
awx_receptor_instance_group_name=production-nodes
```

### Run

```bash
ansible-playbook -i inventory site.yml
```

## Troubleshooting

**Check service status:**
```bash
sudo systemctl status receptor
sudo journalctl -u receptor -f
```

**Verify connectivity (Receptor -> AWX):**
```bash
curl -k https://your-awx-host/api/v2/ping/
```

**Verify connectivity (AWX -> Receptor):**
```bash
telnet your-receptor-node 27199
```

**Common issues:**
- Ensure AWX token has admin privileges
- Check firewall allows port 27199
- Verify DNS resolution between AWX and receptor nodes

## Architecture

```
AWX Control Plane  -->  Receptor Node  -->  Target Systems
    (Port 443)          (Port 27199)          (Various)
```

## CI/CD

This role includes GitHub Actions pipelines:

- **CI** (`ci.yml`): Runs `yamllint`, `ansible-lint`, and Molecule tests on every push/PR
- **Release** (`release.yml`): Publishes to Ansible Galaxy on version tags (`v*`)

### Publishing a New Version

1. Tag the release: `git tag v1.0.0`
2. Push the tag: `git push origin v1.0.0`
3. The release pipeline imports the role to Galaxy automatically

> Set the `GALAXY_API_KEY` secret in your GitHub repository settings.
> Get your API key from https://galaxy.ansible.com/me/preferences

## License

MIT

## Author

Kevin Magni
