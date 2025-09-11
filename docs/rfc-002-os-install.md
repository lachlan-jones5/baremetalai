# RFC-002: Automated Base OS Installation

- **Status**: Draft
- **Author(s)**: Lachlan Jones
- **Date**: 2025-09-10

## Summary

This RFC proposes the creation of a fully automated workflow for the base OS installation on cluster nodes. This workflow will use pre-built "golden images" that are customized per-node and then flashed to storage media. The target operating system is Arch Linux (or Arch Linux ARM). This supports a "wipe and reinstall" strategy for node management.

## Motivation

Managing a cluster of machines requires a consistent and reliable way to provision the base operating system. Manual installations are time-consuming, error-prone, and lead to configuration drift. An automated process will:
- Increase reliability and repeatability.
- Reduce manual effort and time for node setup.
- Ensure all nodes have a consistent base configuration.
- Enable a "wipe and reinstall" approach to easily recover or reset nodes to a known good state.

## Goals

- Define the architecture for creating and customizing golden images for automated base OS installation.
- Define the interface for the image customization process, including parameters for hostname and IP address configuration.
- Detail the strategy for handling different node types (ARM-based Raspberry Pi 4 vs. x86-based workers).
- Create a clear plan for supporting the "'wipe and reinstall' workflow".
- Specify the expected final state of a node after installation, ensuring it is ready for Ansible.
- Ensure all supported nodes are PXE boot capable for future network-based installation support.

## Non-Goals

- This RFC does not cover the configuration of applications or services on top of the base OS. That will be handled by a separate configuration management process (e.g., Ansible).
- This RFC does not cover the provisioning of the physical hardware itself.
- This RFC will not cover network infrastructure setup (e.g., DHCP), although the installation process may rely on existing network connectivity.
- This RFC does not cover the configuration of the master node as a router or the setup of routing/NAT services. These will be handled by Ansible in the post-installation phase.

## Network Architecture

The cluster will use a simple network topology:
- **Master node**: Connected to the home router wirelessly and acts as a router/gateway for worker nodes
- **Worker nodes**: Connected to an unmanaged network switch along with the master node's wired interface
- **IP addressing**: Static IP addresses only - DHCP is not supported in this initial implementation
- **Internet access**: Worker nodes access the internet through the master node acting as a router

The installation process will configure static IP addresses on all nodes, with the master node configured to route traffic for the worker nodes in the post-installation Ansible phase.

**Note**: Future implementations may support DHCP for worker nodes and PXE booting, which would enable the master node to handle automated OS installation for worker nodes without requiring physical USB access to each machine.

## Detailed Design

The core of this proposal is to use pre-built "golden images" that contain a complete, minimal Arch Linux installation ready for Ansible handoff. These images are customized per-node with hostname and IP address configuration, then flashed directly to the target storage media.

### Golden Image Strategy

**Base Images:**
We will maintain pre-built golden images for each supported architecture:
- **x86_64**: Standard Arch Linux image for x86 worker nodes
- **aarch64**: Arch Linux ARM image for Raspberry Pi 4 nodes

Each golden image contains:
- Minimal Arch Linux installation with essential packages
- SSH server configuration with key-based authentication
- Network services configured for static IP (systemd-networkd)
- Management user account with sudo privileges
- Python installed (required for Ansible)
- All necessary configuration except hostname and IP address

**Hardware Requirements:**
All nodes must support PXE boot capabilities, even though this RFC uses direct flashing. This ensures future compatibility when PXE boot support is added. Specifically:
- **Raspberry Pi nodes**: Only Raspberry Pi 4 models are supported (PXE boot capable)
- **x86 worker nodes**: Must support UEFI or BIOS PXE boot

### Image Customization Interface

The primary interface for customizing images will be a script that mounts, modifies, and unmounts golden images.

**Invocation:**
```bash
./customize-image.sh --golden-image <base-image> --hostname <node-name> --ip-address <address>
```

**Process:**
1. Copy the golden image to create a working copy
2. Mount the image filesystem using loop device
3. Write hostname and network configuration to specific locations:
   - `/etc/hostname`: Node hostname
   - `/etc/systemd/network/10-static.network`: Static IP configuration
4. Unmount the image
5. Output the customized image with filename based on hostname and base image name (e.g., `worker01-arch-x86_64.img`)

**Configuration File Format:**
The network configuration will use systemd-networkd format:
```ini
[Match]
Name=eth0

[Network]
Address=<ip-address>/24
Gateway=<gateway-ip>
DNS=<dns-server>
```

### Disk Partitioning

A `disk_layout.json` file will define the partitioning scheme for the primary boot disk. The layout will be determined based on the system's boot firmware:

**UEFI Systems:**
- A boot partition (EFI System Partition)
- A single root partition using the rest of the disk space with an ext4 filesystem

**BIOS/Legacy Systems:**
- A small boot partition (for GRUB)
- A single root partition using the rest of the disk space with an ext4 filesystem

### Idempotency and "Wipe and Reinstall" Workflow

The "wipe and reinstall" nature of the workflow is the core strategy for ensuring a consistent state. The installation process consists of:
1. Creating a customized image from the golden image
2. Flashing the customized image directly to the target storage media
3. Booting the node from the flashed storage

If a node's state becomes corrupted or needs to be reset, the operator can simply re-flash the customized image to bring it to a known-good base state. This makes the overall process idempotent at the node level.

### Final Node State

After a successful installation, a node will be in the following state, ready for Ansible to take over all further configuration:
- Minimal Arch Linux (or Arch Linux ARM) installation with only essential packages
- Network is configured with static IP and active
- SSH server is running and accessible with predefined keys from the master node
- A management user account is created with sudo privileges
- Python is installed (required dependency for Ansible)
- No additional services or applications configured - this is the handoff point to Ansible

This minimal state ensures that the master node can immediately begin Ansible-based configuration management without any manual intervention. All cluster-specific configuration, service installation, and role assignment will be handled by Ansible in the post-installation phase.

## Alternatives

1. **Interactive Installation**: Manual installation of each node would provide maximum flexibility but is time-consuming, error-prone, and doesn't scale well for cluster management.
2. **Other Linux Distributions**: Distributions like Ubuntu or CentOS offer automated installation methods (e.g., Preseed, Kickstart). However, Arch Linux was chosen for its simplicity, minimal base, and rolling-release model, which aligns with the goal of having a modern and lean OS.
3. **archinstall Scripting**: Using Arch's built-in installer would provide automation but lacks the flexibility needed for Raspberry Pi support and cross-architecture deployment.

## Secrets Management Strategy

For secure handling of credentials during installation, we will adopt industry-standard practices:

**Primary Approach:**
- **Ansible Vault** will be used to encrypt the `creds.json` file, ensuring secrets are encrypted at rest. The `customize-image.sh` script will prompt the user for the vault password during image customization.
- SSH public keys will be embedded in the golden images, with corresponding private keys securely stored on the management machine
- No plaintext secrets will be committed to version control

**Alternative Approaches (for future consideration):**
- **Network-based key distribution** - Fetching keys from a secure service during installation
- **Hardware Security Modules (HSMs)** - For environments requiring higher security
- **Certificate-based authentication** - PKI infrastructure for enterprise deployments

## Unresolved Questions

- Future consideration: SSH key rotation process across the cluster (separate operational feature).
- Future consideration: Multiple authentication methods (keys + certificates) for different security requirements.
- Future consideration: DHCP support for worker nodes to simplify network configuration.
- Future consideration: PXE boot support for network-based installation, enabling the master node to handle worker node OS installation without physical USB access.
