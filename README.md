# Spark Cluster Management

Ansible project for managing a small cluster with high-performance networking backbone.

## Features

- Automatically setup SSH keys
- Configure cluster backbone network with jumbo frames and bonding
- Automated network performance testing
- Easy cluster management with Ansible

## Prerequisites

- **uv** - Fast Python package installer ([Install uv](https://github.com/astral-sh/uv))
- Two NVIDIA DGX Sparks connected by a QSFP cable

## Getting Started

### 1. Install Dependencies

```bash
uv sync
```

This installs Ansible and all required collections.

### 2. Configure Your Inventory

Edit `inventory/hosts.yml` to match your cluster:

**Update hostnames and IP addresses:**
```yaml
spark-0:
  ansible_host: 192.168.0.194  # Change to your node's IP/hostname
spark-1:
  ansible_host: 192.168.0.221  # Change to your node's IP/hostname
```

**Select correct network interfaces for bonding:**

The bond members should be the interfaces you want to bond together. Interface naming:
- `np0` = left port
- `np1` = right port

Example:
```yaml
cluster_bond:
  members:
    - enp1s0f0np0    # First interface, left port
    - enP2p1s0f0np0  # Second interface, left port
  address: 172.16.0.1/30  # Cluster backbone IP
  mtu: 9000               # Jumbo frames
```

**Verify interface names** on your nodes with: `ip link show`

### 3. Run Initial Setup

Configure SSH keys and passwordless sudo:

```bash
uv run ./playbooks/setup.yml
```

Enter your SSH password when prompted. This will:
- Update all packages and reboot if needed
- Generate SSH keypair in `data/`
- Configure passwordless SSH access
- Enable passwordless sudo

<details>

<summary>Undo these changes.</summary>

```
uv run ./playbooks/setup.yml -e "state=absent"
```

</details>

### 4. Configure Cluster Backbone

Set up network bonding with performance testing:

```bash
uv run ./playbooks/cluster-backbone.yml
```

This will:
- Install NetworkManager and iperf3
- Create bonded network interfaces (mode: balance-xor)
- Configure jumbo frames (MTU 9000)
- Run automated performance tests between nodes

<details>

<summary>Undo these changes.</summary>

```
uv run ./playbooks/cluster-backbone.yml -e "state=absent"
```

</details>

### 5. (Optional) Configure Local SSH Client

Make it easy to SSH to cluster nodes from your workstation:

```bash
uv run ./playbooks/configure-ssh-client.yml
```

After this, you can simply: `ssh spark-0` or `ssh spark-1`

<details>

<summary>Undo these changes.</summary>

```
uv run ./playbooks/configure-ssh-client.yml -e "state=absent"
```

</details>

## Maintenance

### Update and Reboot Nodes

Keep your cluster up to date:

```bash
uv run ./playbooks/update.yml
```

This will:
- Update package cache
- Upgrade all packages
- Refresh firmware metadata and upgrade firmware
- Check if reboot is required
- Automatically reboot nodes that need it
- Wait for nodes to come back online

### Install NVIDIA Workbench

Install NVIDIA Workbench on all cluster nodes:

```bash
uv run ./playbooks/install-workbench.yml
```

This will:
- Download the latest nvwb-cli for your platform
- Install NVIDIA Workbench with Docker support
- Run in non-interactive mode

<details>

<summary>Uninstall NVIDIA Workbench.</summary>

```
uv run ./playbooks/install-workbench.yml -e "state=absent"
```

</details>
