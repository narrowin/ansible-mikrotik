# ansible-mikrotik

[![Discord][discord-svg]][discord-url] [![DevPod][devpod-svg]][devpod-url] [![Codespaces][codespaces-svg]][codespaces-url]  
![w212][w212][Learn more](https://containerlab.dev/macos/#devpod)![w90][w90][Learn more](https://containerlab.dev/manual/codespaces)

[discord-svg]: https://github.com/user-attachments/assets/8f3f9813-fb61-4673-b666-5bb8dd902c05
[discord-url]: https://discord.gg/6pbRj546gr
[devpod-svg]: https://github.com/user-attachments/assets/6c09e524-850f-468d-b6d8-b2d65112b609
[devpod-url]: https://devpod.sh/open#https://github.com/narrowin/ansible-mikrotik
[codespaces-svg]: https://github.com/user-attachments/assets/95df8a45-cf92-4ce4-91dd-2142a04d28a3
[codespaces-url]: https://codespaces.new/narrowin/ansible-mikrotik?quickstart=1&devcontainer_path=.devcontainer%2Fdevcontainer.json
[w212]: https://github.com/user-attachments/assets/7bd9ab55-dee2-436d-9a36-6cef335921e1
[w90]: https://github.com/user-attachments/assets/f955fb52-9ae6-4b4d-9b06-63b8d44769d4

## Automating MikroTik Device Management with Ansible

ansible-mikrotik enables network engineers to automate the configuration and management of [MikroTik](https://mikrotik.com) devices. By leveraging Ansible's idempotent execution model, the provided playbooks simplify network operations significantly, minimize manual errors, and streamline deployment in dynamic network environments. 

If you want to test this in a lab environment with virtualized MikroTik devices, you can use containerlab environment we have prepared.

![mikrotik-ansible-workflow](https://github.com/user-attachments/assets/8479c7cd-35d2-4084-8848-c2895449d06f)


## Example of a network configured with mikrotik-ansible

Check out the [narrowin demo controller](https://demo.narrowin.ch/) for a live demonstration of this repository's capabilities for MikroTik device management and lab environments. This network is fully virtualized and configured with ansible. 

![image](https://github.com/user-attachments/assets/48df496d-afaf-4586-9f40-48a362394852)


---

## Features

- **Automated Configuration:** Quickly deploy and update MikroTik device settings including interfaces, routing, firewall rules, and VPN configurations.
- **Idempotence:** Ensure configurations are applied consistently without unintended changes.
- **Customizable Variables:** Easily override defaults to suit your configuration requirements.
- **Modular Design:** Clean separation between tasks, defaults, and configuration files for easy maintenance and extension.
- **Dynamic Inventory Support:** Integrate with your existing dynamic inventory setups. (coming soon)

---

## Table of Contents
- [Setup and Deployment](#setup-and-deployment)
- [Running the Ansible Playbooks](#running-and-testing-the-ansible-playbooks)
- [Lab Environment](#lab-environment)
- [Troubleshooting](#ansible-debugging)
- [Known Issues](#caveats)
- [Contributing](#contributing)
- [License](#license)
- [About](#about)
- [Additional Resources](#additional-resources)

---

## Setup and Deployment

### Ansible

The provided playbooks depend on specific Python versions and packages (see [requirements.txt](requirements.txt)) and Ansible collections (see [requirements.yml](requirements.yml)). If you have an existing Ansible setup, try your current environment first—it might work without modifications. Otherwise, follow the installation instructions below.

### Quick Start Guide

1. Clone the repository and set up the environment:
   ```bash
   git clone https://github.com/narrowin/ansible-mikrotik.git
   cd ansible-mikrotik
   python3 -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   ansible-galaxy collection install -r requirements.yml -p collections/
   ```

2. Deploy a test lab with containerlab:
   ```bash
   clab deploy -t containerlabs/s3n.clab.yml
   ```

3. Run your first playbook:
   ```bash
   ansible-playbook playbooks/mikrotik-configure.yml --limit mikrotik_s3n
   ```

### Network Requirements

- Ensure connectivity between your Ansible control node and the MikroTik devices for:
  - SSH (Port: 22)
  - API access (Port: 8728)
  - SSL-API (Port: 8729)

```console
[Ansible Playbook] --> [MikroTik RouterOS API:8728]
[Ansible Playbook] --> [MikroTik RouterOS SSL-API:8729]
[Ansible Playbook] --> [MikroTik RouterOS SSH:22] # some playbooks directly connect via ssh/scp
```

### Ansible setup

We provide some examples on how to use these playbooks to fully configure and backup MikroTik devices. The following sections describe the files that provide this magic.

All [behavioral inventory parameters](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#connecting-to-hosts-behavioral-inventory-parameters) are defined in [inventory/mikrotik](inventory/mikrotik). 
Check this file to identify the IPs for all switches and how to connect to them.

#### Ansible Group Variables Structure

The group variables directory contains several important configuration files:

| File/Directory | Description |
|----------------|-------------|
|Adjust to your setup||
| [inventory/group_vars/mikrotik/](inventory/mikrotik/group_vars/mikrotik/) | Base configuration for all MikroTik devices |
| [inventory/group_vars/all.yml](inventory/group_vars/all.yml) | Global variables applied to all devices |
|No need to edit||
| [inventory/group_vars/mikrotik/](inventory/group_vars/mikrotik/) | RouterOS-specific settings for all MikroTik devices |
| [inventory/group_vars/mikrotik_chr_12ports_containerlab/](inventory/group_vars/mikrotik_chr_12ports_containerlab/) | Settings specific to virtualized RouterOS instances with 12 ports |
|Only edit if you know what you are doing||
| [inventory/group_vars/mikrotik_switches/](inventory/group_vars/mikrotik_switches/) | Configuration files specific to MikroTik switching hardware |
| [inventory/group_vars/mikrotik_routers/](inventory/group_vars/mikrotik_routers/) | WIP to come: Configuration files specific to MikroTik routing hardware |


##### Global group_vars for all devices 
[Key configuration options](inventory/group_vars/all.yml) include:

- **mikrotik_user:** Username for authentication.
- **mikrotik_password:** Password for authentication.
- **local_backups_top_folder:** Path to local backup folder on the ansible control host
- **Additional Settings:** Customize dns, ntp, snmp and other configurations according to your environment.


#### Ansible Host Variables Structure

Ansible uses a variable precedence system where more specific variables override more general ones. The host variables (`host_vars`) directory contains configurations that are specific to individual devices and will override any matching variables defined in group variables (`group_vars`).

This creates a powerful inheritance model:

1. **Global settings** defined in `group_vars/all.yml` apply to all devices
2. **Group-specific settings** like those in `group_vars/mikrotik/` apply to all devices in that group
3. **Host-specific settings** in each device's folder under `host_vars/` have the highest priority and override group settings

For example, if `group_vars/mikrotik/interface.yml` defines a standard interface configuration and `host_vars/clab-s3n-sw-dist1/interface.yml` defines specific settings for that device, the host-specific settings will take precedence for that particular device.

This allows you to:
- Define common configurations once at the group level
- Override only what's necessary for specific devices
- Maintain a clean, DRY (Don't Repeat Yourself) configuration structure

When troubleshooting, always check both host_vars and group_vars to understand the final applied configuration. Have a look at the [Ansible debugging section](#ansible-debugging)

| File/Directory | Description |
|----------------|-------------|
| [inventory/host_vars/clab-s3n-sw-dist1/](inventory/host_vars/clab-s3n-sw-dist1/) | Example configuration for a distribution switch |
| [inventory/host_vars/clab-s3n-sw-acc1/](inventory/host_vars/clab-s3n-sw-acc1/) | Example configuration for an access switch |
| [inventory/host_vars/clab-s3n-sw-acc2/](inventory/host_vars/clab-s3n-sw-acc2/) | Example configuration for a second access switch |

#### Playbooks

Here's a reference to all available playbooks:

| Playbook | Description |
|----------|-------------|
| [playbooks/mikrotik-backup-config.yml](playbooks/mikrotik-backup-config.yml) | Backs up RouterOS configuration files (.rsc files)|
| [playbooks/mikrotik-backup-system.yml](playbooks/mikrotik-backup-system.yml) | Backs up RouterOS system (.backup files)|
| [playbooks/mikrotik-configure.yml](playbooks/mikrotik-configure.yml) | Deploys full configuration to MikroTik devices |
|SSL-API||
| [playbooks/mikrotik-generate-ssl-certs.yml](playbooks/mikrotik-generate-ssl-certs.yml) | Generates SSL certificates for API access |
| [playbooks/mikrotik-configure-ssl-api.yml](playbooks/mikrotik-configure-ssl-api.yml) | Configures SSL API access on MikroTik devices |
|Operations||
| [playbooks/mikrotik-upgrade.yml](playbooks/mikrotik-upgrade.yml) | WIP: Upgrades RouterOS to specified version |
| [playbooks/mikrotik-reset-config.yml](playbooks/mikrotik-reset-config.yml) | WIP: Factory resets device configuration |
| [playbooks/mikrotik-reboot.yml](playbooks/mikrotik-reboot.yml) | Safely reboots MikroTik devices |
| [playbooks/mikrotik-check-versions.yml](playbooks/mikrotik-check-versions.yml) | Retrieves and displays current RouterOS versions |
|Facts gathering||
| [playbooks/mikrotik-resources-usage.yml](playbooks/mikrotik-resources-usage.yml) | |
|More to come for routers and firewalls|...|


##### Network Requirements for the playbooks

- **Network Device Access:** Ensure connectivity between your Ansible control node and the MikroTik devices for SSH (Port: 22) and API-access (Port: 8728) or SSL-API (Port: 8729).

```console
[Ansible Playbook] --> [MikroTik RouterOS API:8728]
[Ansible Playbook] --> [MikroTik RouterOS SSL-API:8729]
[Ansible Playbook] --> [MikroTik RouterOS SSH:22] # some playbooks directly connect via ssh/scp
```

#### Naming convention for inventory files and variables

Every device configuration option must be defined in the under [inventory/group_vars](inventory/group_vars) or [inventory/host_vars](inventory/host_vars).

##### Naming convention for inventory file names

It's recommended to split the configuration variables in multiple files. The file name convention is to align the file name in the inventory with the api/cli endpoint to which the config is applied e.g.

file [inventory/host_vars/clab-s3n-sw-dist1/interface_bridge.yml](inventory/host_vars/clab-s3n-sw-dist1/interface_bridge.yml) defines the configuration for api/cli endpoint `/interface/bridge`

##### Naming convention for inventory var names

Every variable must start with prefix `routeros_` and then the api/cli endpoint where the config applies e.g. `routeros_interface_bridge`

In file [inventory/host_vars/clab-s3n-sw-dist1/interface_bridge.yml] you can find the variable named `routeros_interface_bridge`

  * variable starting with prefix `routeros_` indicates it's a variable defining a routeros configuration
  * variable ending with suffix `_interface_bridge` indicates it's a config option which applies to api/cli endpoint `/interface/bridge`


## For production usage

In production make sure you use ssl-api (disable cleartext), ansible-vault and if possible ssh key-based auth. 

### How to enable API with SSL in mikrotik devices

First generate the required SSL certs executing `ansible-playbook playbooks/mikrotik-generate-ssl-certs.yml`

Once you have the certs you can upload them to the devices and enable the API executing `ansible-playbook playbooks/mikrotik-configure-ssl-api.yml`

### Authentication with ssh-key and ansible-vault

Use ssh-keys for authentication for login on the network devices.

- private ssh key used for authentication should be located e.g. in `~/.ssh/id_rsa` # configure in mikrotik group_vars

- Use ansible-vault to provide login- and API-credentials. This addition helps in daily operations to keep the credentials in a secure place.

- ansible vault password should be stored in a text file in `playbook-network-switches/.vault.pass`


---

## Running and testing the ansible playbooks

### Backup config of mikrotik switches

```bash
ansible-playbook playbooks/mikrotik-backup-config.yml --limit mikrotik_s3n
```

MikroTik config files will be stored in the ansible control host in `backups/` unless reconfigured in [inventory/group_vars/all.yml](inventory/group_vars/all.yml)

### Backup system files mikrotik switches

```bash
ansible-playbook playbooks/mikrotik-backup-system.yml --limit mikrotik_s3n
```

MikroTik system files will be stored in the ansible control host in `backups/`

### mikrotik file transfers

Be aware that ansible-pylibssh won't work to transfer files from/to the mikrotik devices using ansible module `ansible.netcommon.net_get`.
You have to install the packages defined in `requirements.txt` inside your venv (paramiko + scp)

### Full device configuration

#### Ansible dry run and show diff

```bash
ansible-playbook playbooks/mikrotik-configure.yml --limit mikrotik_s3n --check --diff
```

#### Push config

```bash
ansible-playbook playbooks/mikrotik-configure.yml --limit mikrotik_s3n
```

run the playbook twice and see the wonders of idempotency (:

#### Push only specific parts/tags

E.g.: bridge_ports in [playbooks/mikrotik-configure.yml](playbooks/mikrotik-configure.yml)

```bash
ansible-playbook playbooks/mikrotik-configure.yml --limit mikrotik_s3n -t bridge_ports
```

---

## Lab Environment

If you want to test the playbooks using the labs we have prepared at: [containerlabs](containerlabs/) you have two options:

- install containerlab on your machine. For this please follow: [containerlab docs](https://containerlab.dev/quickstart/) this guide
- use one of the provided docker envs described in [Dockerized options](#dockerized-options). These contain a full installation of containerlab as well as the vs-code containerlab extension

### Quick start options with containerlab

The labs provided by this repo are:

- [three MikroTik nodes](containerlabs/s3n.clab.yml) interconnected and two Linux clients attached

### Lab Credentials

```
Mikrotik CHRs: 
  User: admin Password: admin
Linux machines: 
  User: user Password: multit00l
```

#### Start containerlab from the terminal
```bash
clab deploy -t containerlabs/s3n.clab.yml
```

#### Start containerlab from the [VS-Code extension](https://containerlab.dev/manual/vsc-extension/)

Navigate on the left to containerlab. Right click on the lab you want to start and choose `Deploy`


## Login to Lab devices


```bash
ssh admin@clab-s3n-sw-acct1
```

Linux clients - User: user Password: multi00l

```bash
ssh user@clab-s3n-linux1 
```

---

## Ansible Debugging

- Check all vars for resolved for a host
```bash
ansible -m debug -a "var=vars" clab-s3n-sw-acct1
```

- Check the ansible groups for a device in the inventory

```bash
ansible -m debug -a var="hostvars[inventory_hostname]['group_names']" clab-s3n-sw-acct1
```

---

## Caveats

### Details about bonding

Make sure the members of a bond don't belong to the bridge. This requires the right order in the playbook. You should first execute the task

- configuring the bridge so the right interfaces are added/removed from/to the bridge

- **afterwards** you can execute the task configuring the bond.

### Details about trunk ports

Every trunk port should have `bpdu_guard: no` in `interface_bridge_ports.yml`

### common error

```console
TASK [Bond] **************************************************************************************************************************************************************************************************************************************************
fatal: [sw-mkt-03]: FAILED! => changed=false
  msg: 'Error while creating entry for name="uplink-bond": failure: sfp-sfpplus3 already in bridge'
  changed: [sw-mkt-02]
```

This happens because the interfaces were manually added to the bridge during the initial setup. To fix it login to the switch and:

```console
[user@sw-mkt-03] /interface/bridge/port> print     
...
some output 
...
56  H sfp-sfpplus1  bridge  yes     1  0x80             10                  10  none   
;;; defconf
57  H sfp-sfpplus2  bridge  yes     1  0x80             10                  10  none   
;;; defconf
58 IH sfp-sfpplus3  bridge  yes     1  0x80             10                  10  none   
;;; defconf
59 IH sfp-sfpplus4  bridge  yes     1  0x80             10                  10  none   

# delete the physical interfaces that are part of the new bond from the bridge
[user@sw-mkt-03] /interface/bridge/port> remove numbers=56,57,58,59

```

## Contributing

Contributions to ansible-mikrotik are welcome! Please follow these guidelines:

- **Code Standards:** Adhere to existing coding conventions and maintain clean, readable YAML and documentation.
- **Issue Reporting:** Open an issue if you find bugs or have feature requests.
- **Pull Requests:** Submit PRs with detailed descriptions of changes and reference any related issues.
- **Documentation:** Update this README and any inline documentation as necessary.

---

## License

This project is licensed under the [Apache License 2.0](LICENSE)

---

## About

The initial effort and development is a collaboration between [narrowin.ch](https://narrowin.ch) and [scicore.ch](https://scicore.ch). Our goal is to make network management easier and more reliable by providing robust, automated infrastructure as code for MikroTik environments.

---

## Additional Resources

- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible RouterOS collection](https://docs.ansible.com/ansible/latest/collections/community/routeros/index.html)
- [Mikrotik RouterOS Documentation](https://help.mikrotik.com/docs/)
- [Containerlab](https://containerlab.dev/)

### Containerlab, devpod, codespaces and VS-code extension

Great videos by [Roman Dodin](https://github.com/hellt):


- [VS Code extension for Containerlab](https://www.youtube.com/watch?v=NIw1PbfCyQ4)
- [Containerlab and DevPod](https://www.youtube.com/watch?v=ceDrFx2K3jE)
- [Running Containerlab on macOS and Windows with Devcontainers](https://www.youtube.com/watch?v=Xue1pLiO0qQ)

---

*Happy automating!*
