# RFC-002: Automated Base OS Installation

- **Status**: Draft
- **Author(s)**: Lachlan Jones
- **Date**: 2025-09-10

## Summary

This RFC proposes the creation of a fully automated workflow for the base OS installation on cluster nodes. This workflow will leverage `archinstall` scripting to ensure a consistent, repeatable, and idempotent setup for every node. The target operating system is Arch Linux (or Arch Linux ARM). This supports a "wipe and reinstall" strategy for node management.

## Motivation

Managing a cluster of machines requires a consistent and reliable way to provision the base operating system. Manual installations are time-consuming, error-prone, and lead to configuration drift. An automated process will:
- Increase reliability and repeatability.
- Reduce manual effort and time for node setup.
- Ensure all nodes have a consistent base configuration.
- Enable a "wipe and reinstall" approach to easily recover or reset nodes to a known good state.

## Goals

- Define the architecture for the `archinstall` scripting for automated base OS installation.
- Define the interface for the installation scripts, including parameters and configuration files.
- Detail the strategy for handling different node types (ARM-based Raspberry Pi vs. x86-based workers).
- Create a clear plan for supporting the "'wipe and reinstall' workflow".
- Specify the expected final state of a node after installation, ensuring it is ready for Ansible.

## Non-Goals

- This RFC does not cover the configuration of applications or services on top of the base OS. That will be handled by a separate configuration management process (e.g., Ansible).
- This RFC does not cover the provisioning of the physical hardware itself.
- This RFC will not cover network infrastructure setup (e.g., DHCP, PXE booting), although the installation process may rely on it.

## Detailed Design

The core of this proposal is to use the guided installer of Arch Linux, `archinstall`, in its scripted mode. This defines the architecture for our automated OS installation. The process will be driven by declarative configuration files, ensuring a consistent and repeatable setup.

### Installation Interface

The primary interface for the installation process will be a set of JSON configuration files and a wrapper script.

**Configuration Files:**
- `config.json`: Specifies all installation options.
- `disk_layout.json`: Defines the disk partitioning scheme.
- `creds.json`: Contains secrets like user passwords or encrypted SSH keys. This file will be managed securely and not committed to the repository.

**Parameters defined in `config.json` will include:**
- Hostname
- User accounts and credentials (references to `creds.json`)
- Network configuration (static or DHCP)
- Keyboard layout, locale, and timezone
- List of packages to install (e.g., `base`, `linux`, `openssh`, `python`)

**Invocation:**
A wrapper script (`install.sh`) will be created to simplify the process:
```bash
./install.sh --hostname <node-name> --ip-address <address> --node-type [master|worker]
```
This script will dynamically generate the necessary `config.json` from a template and then execute `archinstall`.

```bash
# Example of the underlying archinstall command
archinstall --config generated_config.json --disk-layout disk_layout.json --creds creds.json --silent
```

### Disk Partitioning

A `disk_layout.json` file will define the partitioning scheme for the primary boot disk. A simple and robust layout will be used:
- A boot partition (EFI System Partition).
- A single root partition using the rest of the disk space with an ext4 filesystem.

This declarative approach to partitioning ensures consistency across all nodes. 

**Note**: This RFC only covers partitioning of the primary boot disk. Additional storage devices (external SSDs, data drives, etc.) will be handled by Ansible during the post-install configuration phase, allowing for role-specific storage configurations.

### Post-installation Steps

`archinstall` allows for running post-installation scripts via the `post-install` hook. We will use this feature to:
1.  Enable the SSH daemon (`sshd.service`).
2.  Enable network services (`systemd-networkd.service` and `systemd-resolved.service`).
3.  Configure SSH key-based authentication by copying authorized SSH public keys for the management user, allowing secure access for Ansible and administrative tasks.

**Note**: SSH will be configured to use key-based authentication only. Password authentication will be disabled for security. The SSH public keys will be sourced from the `creds.json` file during installation.

### Final Node State

After a successful installation, a node will be in the following state:
- Minimal Arch Linux (or Arch Linux ARM) installation.
- Network is configured and active.
- SSH server is running and accessible with predefined keys.
- A management user account is created.
- Python is installed, making the node ready for Ansible to take over for further configuration.

### Idempotency and "Wipe and Reinstall" Workflow

The "wipe and reinstall" nature of the workflow is the core strategy for ensuring a consistent state. The installation process will always assume it is running on a blank disk. If a node's state becomes corrupted or needs to be reset, the operator can simply re-run the installation process on the same hardware to bring it to a known-good base state. This makes the overall process idempotent at the node level.

### Node Type Strategy (Architecture Support)

To support different node types across multiple architectures, we will use separate `archinstall` configuration profiles.
- **ARM-based nodes (aarch64):** An `aarch64` profile will be used, specifying ARM-specific packages (`linux-aarch64`) and bootloader configurations.
- **x86-based nodes (x86_64):** A standard `x86_64` profile will be used with appropriate kernel and bootloader settings.

The `install.sh` wrapper script will select the appropriate profile based on the `--node-type` parameter. This allows for a unified workflow while accommodating hardware differences.

### Installation Media Strategy

The installation will be delivered via USB drive using a multi-partition approach:

**Partition Layout:**
- **Partition 1**: Bootable Arch Linux ISO (FAT32, bootable)
- **Partition 2**: Installation scripts and configuration files (FAT32, data partition)

**USB Drive Contents:**
```
/dev/sdb1 (bootable)    - Arch Linux ISO
/dev/sdb2 (data)        - /install/
                           ├── install.sh
                           ├── config_templates/
                           │   ├── config_template.json
                           │   ├── disk_layout.json
                           │   └── profiles/
                           │       ├── aarch64.json
                           │       └── x86_64.json
                           └── creds/
                               └── creds.json.vault
```

This approach allows a single USB drive to both boot the Arch installer and provide all necessary automation scripts and configuration files. The installer environment can mount the data partition to access the installation scripts.

## Alternatives

1.  **Custom Shell Scripts**: We could write our own shell scripts to perform the installation using `pacstrap` and `arch-chroot`. This offers more flexibility but is significantly more complex to create and maintain. `archinstall` abstracts away many of these complexities.
2.  **Other Linux Distributions**: Distributions like Ubuntu or CentOS offer automated installation methods (e.g., Preseed, Kickstart). However, Arch Linux was chosen for its simplicity, minimal base, and rolling-release model, which aligns with the goal of having a modern and lean OS.

## Secrets Management Strategy

For secure handling of credentials during installation, we will adopt industry-standard practices:

**Primary Approach:**
- **Ansible Vault** will be used to encrypt the `creds.json` file, ensuring secrets are encrypted at rest
- SSH public keys will be embedded in the installation media, with corresponding private keys securely stored on the management machine
- No plaintext secrets will be committed to version control

**Alternative Approaches (for future consideration):**
- **Network-based key distribution** - Fetching keys from a secure service during installation
- **Hardware Security Modules (HSMs)** - For environments requiring higher security
- **Certificate-based authentication** - PKI infrastructure for enterprise deployments

## Unresolved Questions

- Future consideration: SSH key rotation process across the cluster (separate operational feature).
- Future consideration: PXE boot support for network-based installation (nice-to-have feature).
- Future consideration: Multiple authentication methods (keys + certificates) for different security requirements.
