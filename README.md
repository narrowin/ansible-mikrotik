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

## Automating Mikrotik Device Configuration with Ansible

ansible-mikrotik enables network engineers to automate the configuration and management of [Mikrotik](https://mikrotik.com) devices. By leveraging Ansible’s idempotent execution model, this role simplifies network operations, minimizes manual errors, and streamlines deployment in dynamic network environments.

For some quick impressions of using this repo with a mikrotik containerlab setup check out the [narrowin demo controller](https://demo.narrowin.ch/).

---

## Features

- **Automated Configuration:** Quickly deploy and update Mikrotik device settings including interfaces, routing, firewall rules, and VPN configurations.
- **Idempotence:** Ensure configurations are applied consistently without unintended changes.
- **Customizable Variables:** Easily override defaults to suit your configuration requirements.
- **Modular Design:** Clean separation between tasks, defaults, and configuration files for easy maintenance and extension.
- **Dynamic Inventory Support:** Integrate with your existing dynamic inventory setups. (coming soon)

---

## Requirements

- **Network Device Access:** Ensure connectivity between your Ansible control node and the Mikrotik devices for ssh (Port: 22) and api-access (Port: 8728) or ssl-api (Port: 8729).

---

## Setup

To run completely in your browser use [codespaces](https://docs.github.com/en/codespaces)

**Be patient while the environment is spinnung up for the first time since it needs to pull some MBytes of containers for the system to be up and running (2-5 Minutes)**

To run locally without any need for software installation using [devpods](https://devpod.sh). 

**WARNING for Apple Silicon users (ARM based MACs) also wanting to try the lab: The containerlab with Mikrotik docker image that is provided for the lab is build with [vrnetlab](https://github.com/vrnetlab/vrnetlab/tree/master/routeros) and does not yet support ARM based MACs. You should use github codespaces instead for now!**


### Local installation

```bash
git clone https://github.com/narrowin/ansible-mikrotik.git
python3 -m venv venv
source venv/bin/activate
(venv) $> pip install -r requirements.txt
(venv) $> ansible-galaxy collection install -r requirements.yml -p collections/
```

To install contgainerlab for testing the playbooks follow: [containerlab docs](https://containerlab.dev/quickstart/)

## Running and testing the playbooks

### Quick start options with containerlab

Spin up labs in the [containerlabs](containerlabs) folder to test ansible playbooks with no local software dependencies - fully self-contained environment with a lab topology, ansible and this repo installed. All batteries included!

The labs provided by this repo are:

- [three Mikrotik nodes and two Linux clients](containerlabs/simple.clab.yml)
- [single Mikrotik node to test with](containerlabs/simple.clab.yml) - to run this with ansible you need to enable the clab-simple-n1 definition in [inventory/mikrotik](inventory/mikrotik)

#### Start containerlab from the terminal
```bash
clab deploy -t containerlabs/s3n.clab.yml
```

#### Start containerlab from the [VS-Code extension](https://containerlab.dev/manual/vsc-extension/)

Navigate on the left to containerlab. Right click on the lab you want to start and choose `Deploy`


### More inforamtion on containerlab and devpod, codespaces and the VS-code extension

Great videos by [Roman Dodin](https://github.com/hellt):


- [VS Code extension for Containerlab](https://www.youtube.com/watch?v=NIw1PbfCyQ4)
- [Containerlab and DevPod](https://www.youtube.com/watch?v=ceDrFx2K3jE)
- [Running Containlerab on macOS and Windows with Devcontainers](https://www.youtube.com/watch?v=Xue1pLiO0qQ)

## Configuration when running against other Mikrotik devices

[Key configuration options](group_vars/all.yml) include:

- **mikrotik_user:** Username for authentication.
- **mikrotik_password:** Password for authentication.
- **local_backups_top_folder:** Path to local backup folder on the ansible control host
- **Additional Settings:** Customize dns, ntp, snmp and otgher configurations according to your environment.

All [behavioral inventory parameters](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#connecting-to-hosts-behavioral-inventory-parameters) are defined in [inventory/mikrotik](inventory/mikrotik). 
Check this file to identify the IPs for all switches and how to connect to them.

*Connecting Requirements:*  

```conole
[Ansible Playbook] --> [Mikrotik RouterOS API]
[Ansible Playbook] --> [Mikrotik RouterOS SSL-API]
[Ansible Playbook] --> [Mikrotik RouterOS SSH] # some playbooks directly connect via ssh/scp
```

### Naming convention for inventory files and variables

Every device configuration option must be defined in the under [inventory/group_vars](inventory/group_vars) or [inventory/host_vars](inventory/host_vars).

#### Naming convention for inventory file names

It's recommended to split the configuration variables in multiple files. The file name convention is to align the file name in the inventory with the api/cli endpoint to which the config is applied e.g.

file [inventory/host_vars/clab-s3n-sw-dist1/interface_bridge.yml](inventory/host_vars/clab-s3n-sw-dist1/interface_bridge.yml) defines the configuration for api/cli endpoint `/interface/bridge`

#### Naming convention for inventory var names

Every variable must start with prefix `routeros_` and then the api/cli endpoint where the config applies e.g. `routeros_interface_bridge`

In file [inventory/host_vars/clab-s3n-sw-dist1/interface_bridge.yml] you can find the variable named `routeros_interface_bridge`

  * variable starting with prefix `routeros_` indicates it's a variable defining a routeros configuration
  * variable ending with suffix `_interface_bridge` indicates it's a config option which applies to api/cli endpoint `/interface/bridge`

---

## Running and testing the ansible playbooks

### Backup config of mikrotik switches

```bash
ansible-playbook playbooks/mikrotik-backup-config.yml --limit mikrotik_s3n
```

Mikrotik config files will be stored in the ansible control host in `backups/` unless reconfigured in [group_vars/all.yml](group_vars/all.yml)

### Backup system files mikrotik switches

```bash
ansible-playbook playbooks/mikrotik-backup-system.yml --limit mikrotik_s3n
```

Mikrotik system files will be stored in the ansible control host in `playbook-network-switches/backups/`

### mikrotik file transfers

Be aware that ansible-pyblibssh won't work to transfer files from/to the mikrotik devices using ansible module `ansible.netcommon.net_get`.
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

E.g.: bridge_ports in [playbook/mirotik-configure.yml](playbook/mirotik-configure.yml)

```bash
ansible-playbook playbooks/mikrotik-configure.yml --limit mikrotik_s3n -t bridge_ports
```

---

## Login to Lab devices

Switches - User: admin - Password: multi00l

```bash
ssh admin@clab
```

Linux clients - User: user Password: multi00l

```bash
ssh user@clab-s3n-linux1 
```

---

## Debugging

### Check the ansible groups for a device in the inventory

```bash
# all groups
ansible -m debug -a var="hostvars[inventory_hostname]['group_names']" sw-mkt-cluster-05
```

```bash
# all vars
ansible -m debug -a "var=vars" clab-simple-n1
```

---

## Caveats

### Details about bonding

Make sure the members of a bond don't belong to the bridge. This requires the right order in the playbook. You should first execute the task

- configuring the bridge so the right interfaces are added/removed from/to the bridge

- **afterwards** you can execute the task configuring the bond.

### Details about trunk ports

Every trunk port should have `bpdu_guard: no` in `interface_brige_ports.yml`

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

This project is licensed under the [Apachae License 2.0](LICENSE)

---

## About

The initial effort and development is a collaboration between [narrowin.ch](https://narrowin.ch) and [scicore.ch](https://scicore.ch). Our goal is to make network management easier and more reliable by providing robust, automated infrastructure as code for MikroTik environments.

---

## Additional Resources

- [Ansible Documentation](https://docs.ansible.com/)
- [Ansible RouerOS collection](https://docs.ansible.com/ansible/latest/collections/community/routeros/index.html)
- [Mikrotik RouterOS Documentation](https://help.mikrotik.com/docs/)
- [Containerlab](https://containerlab.dev/)

### How to enable API with SSL in mikrotik devices

First generate the required SSL certs executing `ansible-playbook playbooks/mikrotik-generate-ssl-certs.yml`

Once you have the certs you can upload them to the devices and enable the API executing `ansible-playbook playbooks/mikrotik-generate-ssl-certs.yml`

### Authentication with ssh-key and ansible-vault

Use ssh-keys for authentication for login on the network devices.

- private ssh key used for authentication should be located e.g. in `~/.ssh/id_rsa` # configure in mikrotik group_vars

- Use ansible-vault to provide login- and API-credentials. This addition helps in daily operations to keep the credentials in a securere standard place.

- ansible vault password should be stored in a text file in `playbook-network-switches/.vault.pass`

---

*Happy automating!*
