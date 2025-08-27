# Ansible Playbook to deploy Gnoland Node

This Ansible playbook automates the deployment and configuration of Gnoland Node. It ensures that the necessary dependencies, configuration files, and services are properly set up and running.

## Table of Contents

- [Requirements](#requirements)
- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [Variables](#variables)
- [Usage](#usage)

## Requirements

Before using this playbook, ensure the following requirements are met:

1. **Ansible version**: Make sure you have Ansible 2.15+ installed.
2. **SSH Access**: Passwordless SSH access to all target servers.
3. **Python**: Python 3.x installed on the control node and all target hosts.
4. **Privileges**: The user running the playbook must have sudo privileges on the target machines.

## Prerequisites

**Install HashiCorp Vault**

This playbook relies on HashiCorp Vault to securely retrieve sensitive files, such as identity and vote account keys. Follow the [HashiCorp Vault Installation Guide](https://developer.hashicorp.com/vault/tutorials/getting-started/getting-started-install) to set up Vault on your infrastructure.

**Note on Secrets Management**

The playbook dynamically retrieves secrets from HashiCorp Vault. The keys are expected to follow a structured path format:
`<environment>/<project>/<organization>/<type>/<file_name>`

i.e.:
- `testnet/gnoland/encapsulate/validator/node_key.json`
- `testnet/gnoland/encapsulate/validator/priv_validator_key.json`

This structure ensures easy organization and secure retrieval of secrets.

## Setup

### 1. Install Ansible

If Ansible is not installed, visit the official documentation for detailed instructions on how to install Ansible on various Linux distributions:

[Ansible Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html)

### 2. Clone the repository

Clone this repository to your Ansible control node:

```bash
git clone https://github.com/encapsulate-xyz/gnoland-ansible.git
cd gnoland-ansible
```

### 3. Inventory

Define your target servers' IP address or DNS in the inventory folder, and select either `mainnet.yml` or `testnet.yml` to update.

Example for testnet.yml

```yaml
---
# maintains testnet inventory grouped by project names
all:
  vars:
    env: testnet
  children:
    gnoland:
      hosts:
        validator.gnoland.testnet.encapsulate.xyz:
          type: validator
```

## Variables

This playbook allows customization through several variables. You can define these variables in the following locations:

- **`group_vars/all.yml`**: Contains all the source urls, ports and configurations.

| Name                                         | Default Value                                                       | Description                                                  |
|----------------------------------------------|----------------------------------------------------------------------|--------------------------------------------------------------|
| `org`                                        | `encapsulate`                                                       | Organization name used for internal config                   |
| `domain_regex`                               | `^([a-zA-Z0-9-]+\.)+[a-zA-Z]{2,}$`                                   | Regex pattern to validate domain names                       |
| `monitor_server_dns`                         | `monitor.encapsulate.xyz`                                           | DNS address of the monitoring server                         |
| `ansible_port`                               | `8192`                                                               | SSH port used by Ansible to connect to nodes                 |
| `gnoland.validator.source_url`                | `https://github.com/gnolang/gnod` | Source URL to download the Gnoland node binary      |
| `gnoland.validator.ports.prefix`            | `207`                                                               | Port Prefix used for Gnoland Node                          |

- **`group_vars/mainnet.yml`** or **`group_vars/testnet.yml`**: Contains network specific variables.

| Name                              | Default Value                                          | Description                                           |
|-----------------------------------|-------------------------------------------------------|-------------------------------------------------------|
| `gnoland.chain_id`                | `test7`                                               | Chain ID of the gnoland network                  |
| `gnoland.validator.version`       | `chain/test7.2`                                       | Version of the gnoland node binary               |
| `gnoland.validator.go.version`    | `1.23.1`                                              | Go toolchain version required for building binary  |
| `gnoland.validator.exporter_endpoint` | `http://{{ monitor_server_dns }}:4323`                 | Otel metrics exporter endpoint for monitoring   |
| `gnoland.snapshot_url`            | `https://snapshot.shazoes.xyz/testnets/snapshot-gnoland.tar.lz4` | URL for latest gnoland snapshot               |                                               |

- **`group_vars/ports.yml`**: Contains port suffix for node.

| Name                 | Default Value | Description                                                                 |
|----------------------|---------------|-----------------------------------------------------------------------------|
| `ports.suffix.p2p`   | `56`          | Port suffix used for both `p2p_laddr` and `p2p_external_address`            |
| `ports.suffix.rpc`   | `57`          | Port suffix used for the RPC endpoint                                       |
| `ports.suffix.abci`  | `58`          | Port suffix used for the ABCI connection |

- **`group_vars/vault.yml`**: Store secret variables, such as HCP Vault url, passwords in this file.

| Name                                      | Default Value                | Description                                                |
|-------------------------------------------|-------------------------------|------------------------------------------------------------|
| `vault.default.hcp.vault.url`             | `"your_hcp_vault_url"`        | URL of the HCP Vault used for secure secret storage        |

### Usage

1. First, install the dependencies:

  ```bash
   ansible-galaxy install -r collections/requirements.yml
  ```

2. Create a `ansible_vault_password` file containing ansible-vault password

3. Then run the playbook:

  ```bash
  ansible-playbook setup_node.yml -i inventory/testnet.yml -e "fetch_validator_keys=true"
  ```

**Note**: The variable `fetch_validator_keys` enables fetching secrets from HCP Vault.

After you run the playbook, it will ask for confirmation, displaying all the variables and the IP address or DNS of the server you are going to deploy.

Example output:

```bash
PLAY [Set up gnoland node] *******************************************************************************************************
ok: [validator.gnoland.testnet.encapsulate.xyz]

TASK [Gathering Facts] ***********************************************************************************************************
ok: [validator.gnoland.testnet.encapsulate.xyz]

TASK [Include environment setup tasks] *******************************************************************************************
included: ../tasks/setup_environment.yml for validator.gnoland.testnet.encapsulate.xyz

TASK [Set project and service identifier in facts] *******************************************************************************
ok: [validator.gnoland.testnet.encapsulate.xyz]

TASK [Set architecture variable based on system architecture] ********************************************************************
ok: [validator.gnoland.testnet.encapsulate.xyz]

TASK [Display environment being deployed to] *************************************************************************************
ok: [validator.gnoland.testnet.encapsulate.xyz] => {
    "msg": [
        "Deploying to Host: validator.gnoland.testnet.encapsulate.xyz",
        "Groups: ['gnoland']",
        "Project: gnoland",
        "Environment: testnet",
        "Type: validator",
        "Version: chain/test7.2",
        "Username: gnoland",
        "Service Name: gnoland",
        "Operating System: linux",
        "System Architecture: amd64"
    ]
}

TASK [Confirm deployment details] ********************************************************************************************************************
Pausing for 40 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [validator.gnoland.testnet.encapsulate.xyz]

TASK [Please confirm again] ********************************************************************************************************************
ok: [validator.gnoland.testnet.encapsulate.xyz] => {
    "msg": [
        "Deploying to Host: validator.gnoland.testnet.encapsulate.xyz",
        "Project: gnoland",
        "Environment: testnet",
        "Type: validator"
    ]
}

TASK [Confirm deployment details] **************************************************************************************************************
Pausing for 20 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [validator.gnoland.testnet.encapsulate.xyz]
```
