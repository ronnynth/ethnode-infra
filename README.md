# Ethereum Infrastructure Ansible Playbook

This repository contains Ansible playbooks for deploying and managing Ethereum infrastructure, including execution clients (Geth), consensus clients (Lighthouse/Prysm), and MEV-Boost relays.

## 🏗️ Architecture Overview

The playbook deploys a complete Ethereum node infrastructure with the following components:

```
┌─────────────────────────────────────────────────────────────┐
│                    Ethereum Node Infrastructure             │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────────────┐ │
│  │   Geth      │  │  Lighthouse  │  │     MEV-Boost       │ │
│  │ (Execution) │◄─┤ (Consensus)  │◄─┤     (Relay)         │ │
│  │             │  │              │  │                     │ │
│  │ Port: 8545  │  │ Port: 5052   │  │ Port: 18550         │ │
│  │ Port: 8546  │  │              │  │                     │ │
│  │ Port: 8551  │  │              │  │                     │ │
│  └─────────────┘  └──────────────┘  └─────────────────────┘ │
│                                                             │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │              Base System Components                     │ │
│  │  • Docker                                               │ │
│  │  • Go language runtime                                  │ │
│  │  • System optimization (kernel params, limits)         │ │
│  │  • Disk management and mounting                        │ │
│  │  • Monitoring tools (htop, iotop, prometheus-exporter) │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

## 📋 Prerequisites

### System Requirements
- **OS**: Ubuntu 20.04 LTS or 22.04 LTS
- **RAM**: Minimum 32GB (64GB recommended for mainnet)
- **Storage**: 
  - Execution client: 2TB+ NVMe SSD
  - Consensus client: 500GB+ SSD
- **Network**: Stable internet connection with sufficient bandwidth
- **Ports**: 30303 (Geth P2P), 9000 (Lighthouse P2P), 18550 (MEV-Boost)

### Local Requirements
- **Ansible**: Version 2.9 or higher
- **Python**: Version 3.8 or higher
- **SSH Access**: Key-based authentication to target hosts

## 🚀 Quick Start

### 1. Install Dependencies

```bash
# Install required Ansible collections
ansible-galaxy collection install community.general
ansible-galaxy collection install ansible.posix
ansible-galaxy collection install community.docker
```

### 2. Configure Inventory

Edit the `hosts` file to define your target servers:

```ini
[ethereum_nodes]
eth-node-01 ansible_host=192.168.1.10 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/eth_key

[qcloud]
qcloud-eth-01 ansible_host=10.0.0.10 ansible_user=ubuntu
```

### 3. Update Configuration

Modify `deploy.yml` to match your environment:

```yaml
vars:
  execution_disk: /dev/vdb    # Your actual disk device
  consensus_disk: /dev/vdc    # Secondary disk or same as execution
```

### 4. Deploy Infrastructure

```bash
# Deploy everything
ansible-playbook -i hosts deploy.yml

# Deploy specific components
ansible-playbook -i hosts deploy.yml --tags "base,geth"
ansible-playbook -i hosts deploy.yml --tags "lighthouse"
ansible-playbook -i hosts deploy.yml --tags "mev"
```

## 🔧 Configuration Guide

### Network Configuration

The playbook supports multiple Ethereum networks. Configure in role variables:

```yaml
# For Mainnet (default)
network: mainnet
checkpoint: https://mainnet.checkpoint.sigp.io

# For Holesky testnet
network: holesky
checkpoint: https://holesky.checkpoint.sigp.io
```

### MEV Configuration

MEV-Boost is configured with multiple relays for optimal performance:

- Flashbots
- Titan Relay
- Agnostic Relay
- Ultra Sound Money
- Aestus
- SecureRPC
- BloxRoute (Max Profit & Regulated)
- Eden Network

### Fee Recipient

**Important**: Update the fee recipient address in role variables:

```yaml
# lighthouse/vars/main.yml
recipient: "0xYOUR_ETHEREUM_ADDRESS_HERE"

# prysm/vars/main.yml  
recipient: "0xYOUR_ETHEREUM_ADDRESS_HERE"
```

## 📁 Project Structure

```
spec/
├── deploy.yml              # Main deployment playbook
├── hosts                   # Inventory file with examples
├── README.MD              # This documentation
└── roles/
    ├── base/              # Base system setup
    │   ├── files/         # Static files (daemon.json)
    │   ├── tasks/         # Ansible tasks
    │   └── vars/          # Variables
    ├── disk/              # Disk management
    ├── geth/              # Geth execution client
    │   ├── files/         # JWT token
    │   ├── templates/     # Service files and scripts
    │   └── vars/          # Geth configuration
    ├── lighthouse/        # Lighthouse consensus client
    │   ├── files/         # JWT token
    │   ├── templates/     # Service files and scripts
    │   └── vars/          # Lighthouse configuration
    ├── mevbuilder/        # MEV-Boost setup
    │   ├── templates/     # Service files and scripts
    │   └── vars/          # MEV configuration
    ├── prysm/             # Prysm consensus client (alternative)
    │   ├── files/         # JWT token
    │   ├── templates/     # Service files and scripts
    │   └── vars/          # Prysm configuration
    └── update/            # Update tasks for existing deployments
        ├── tasks/         # Update procedures
        ├── templates/     # Updated scripts
        └── vars/          # Update configuration
```

## 🔐 Security Features

### JWT Authentication
Each client uses unique JWT tokens for secure communication:
- Execution ↔ Consensus client authentication
- All JWT tokens are randomly generated and unique per deployment

### System Hardening
- Optimized kernel parameters for network performance
- Increased file descriptor limits
- Docker daemon security configuration
- Firewall considerations (manual setup required)

### Best Practices
- Services run as dedicated users
- Proper file permissions and ownership
- Systematic logging and monitoring capabilities
- Graceful service restart procedures

## 🛠️ Operations Guide

### Service Management

```bash
# Check service status
systemctl status geth
systemctl status lighthouse
systemctl status mev-boost

# View logs
journalctl -u geth -f
journalctl -u lighthouse -f
journalctl -u mev-boost -f

# Restart services
systemctl restart geth
systemctl restart lighthouse
systemctl restart mev-boost
```

### Updates

Use the update role for upgrading clients:

```bash
# Update Geth
ansible-playbook -i hosts deploy.yml --tags "update.geth"

# Update Lighthouse
ansible-playbook -i hosts deploy.yml --tags "update.lighthouse"

# Update MEV-Boost
ansible-playbook -i hosts deploy.yml --tags "update.mev"
```

### Monitoring

The playbook installs basic monitoring tools:
- `htop` - Process monitoring
- `iotop` - I/O monitoring
- `iftop` - Network monitoring
- `prometheus-node-exporter` - Metrics export

### Backup Considerations

**Critical data to backup:**
- JWT tokens: `{base_dir}/{client}/.jwt.hex`
- Validator keys (if running validators)
- Configuration files: `/etc/systemd/system/*.service`

## 🔄 Client Alternatives

### Consensus Client Options

The playbook supports multiple consensus clients:

1. **Lighthouse** (default) - Rust-based, memory efficient
2. **Prysm** - Go-based, feature-rich

To use Prysm instead of Lighthouse:

```yaml
# In deploy.yml, replace lighthouse role with:
- role: prysm
  vars:
    base_dir: "{{ consensus_data_dir }}"
  tags: ['prysm', 'consensus']
```

## 🐛 Troubleshooting

### Common Issues

1. **Disk not found**: Update `execution_disk` and `consensus_disk` variables
2. **Permission denied**: Ensure SSH key authentication is working
3. **Port conflicts**: Check if ports 8545, 8546, 8551, 5052, 9000, 18550 are available
4. **Sync issues**: Verify network connectivity and disk space

### Log Analysis

```bash
# Check for errors in logs
journalctl -u geth --since "1 hour ago" | grep -i error
journalctl -u lighthouse --since "1 hour ago" | grep -i error

# Monitor sync progress
journalctl -u geth -f | grep "Imported new chain segment"
journalctl -u lighthouse -f | grep "Synced"
```

### Performance Tuning

1. **I/O Performance**: Ensure SSD disks, enable noatime mount option
2. **Network**: Optimize peer connections, use fast internet
3. **Memory**: Consider increasing if experiencing OOM issues
4. **CPU**: Modern multi-core processor recommended

## 📚 Additional Resources

- [Ethereum Foundation Documentation](https://ethereum.org/en/developers/docs/)
- [Geth Documentation](https://geth.ethereum.org/docs/)
- [Lighthouse Documentation](https://lighthouse-book.sigmaprime.io/)
- [MEV-Boost Documentation](https://boost.flashbots.net/)
- [Ansible Documentation](https://docs.ansible.com/)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test on a test network first
5. Submit a pull request

## ⚠️ Disclaimer

This playbook is provided as-is. Always test on testnets before deploying to mainnet. Running Ethereum infrastructure involves risks including but not limited to:
- Financial loss from slashing
- Hardware failures
- Network security vulnerabilities
- Regulatory compliance requirements

Ensure you understand these risks and have proper backup and security procedures in place.

## 📄 License

This project is licensed under the MIT License. See LICENSE file for details.