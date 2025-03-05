# ansible-mikrotik

[![Discord][discord-svg]][discord-url] [![DevPod][devpod-svg]][devpod-url] [![Codespaces][codespaces-svg]][codespaces-url]  
![w212][w212][Learn more](https://containerlab.dev/macos/#devpod)![w90][w90][Learn more](https://containerlab.dev/manual/codespaces)


[discord-svg]: https://raw.githubusercontent.com/narrowin/ansible-mikrotik/main/images/join-discord-btn.svg
[discord-url]: https://discord.gg/6pbRj546gr
[devpod-svg]: https://raw.githubusercontent.com/narrowin/ansible-mikrotik/main/images/open-in-devpod-btn.svg
[devpod-url]: https://devpod.sh/open#https://github.com/narrowin/ansible-mikrotik
[codespaces-svg]: https://raw.githubusercontent.com/narrowin/ansible-mikrotik/main/images/run-codespaces-btn.svg
[codespaces-url]: https://codespaces.new/narrowin/ansible-mikrotik?quickstart=1&devcontainer_path=.devcontainer%2Fdevcontainer.json
[w212]: https://raw.githubusercontent.com/narrowin/ansible-mikrotik/main/images/w212h1.svg
[w90]: https://raw.githubusercontent.com/narrowin/ansible-mikrotik/main/images/w90h1.svg


**Automating Mikrotik Device Configuration with Ansible**

ansible-mikrotik enables network engineers to automate the configuration and management of [Mikrotik](https://mikrotik.com) devices. By leveraging Ansible’s idempotent execution model, this role simplifies network operations, minimizes manual errors, and streamlines deployment in dynamic network environments.

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

## Quick start options

Spin up a lab in the [containerlabs](containerlabs) folder. No local software dependencies - fully self-contained environment with a lab topology, ansible and this repo installed. All batteries included! 

Either use the 

## Installation


```bash
git clone https://github.com/narrowin/ansible-mikrotik.git
python3 -m venv venv
source venv/bin/activate
(venv) $> pip install -r requirements.txt
(venv) $> ansible-galaxy collection install -r requirements.yml -p collections/
```


## Configuration

[Key configuration options](group_vars/all.yml) include:

- **mikrotik_user:** Username for authentication.
- **mikrotik_password:** Password for authentication.
- **local_backups_top_folder:** Path to local backup folder on the ansible control host
- **Additional Settings:** Customize dns, ntp, snmp and otgher configurations according to your environment.

All [behavioral inventory parameters](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#connecting-to-hosts-behavioral-inventory-parameters) are defined in [inventory/mikrotik](inventory/mikrotik). 
Check this file to identify the IPs for all switches and how to connect to them.


*Connecting Requirements:*  

```
 [Ansible Playbook] --> [Mikrotik RouterOS API]
 [Ansible Playbook] --> [Mikrotik RouterOS SSL-API]
 [Ansible Playbook] --> [Mikrotik RouterOS SSH] # some playbooks directly connect via ssh/scp
```
---

## Usage

### Backup config mikrotik switches
```
$> ansible-playbook playbooks/backup-mikrotik-config.yml --limit sw-mkt-mgmt-01

```
Mikrotik config files will be stored in the ansible control host in `playbook-network-switches/backups/`

### Backup system files mikrotik switches
```
$> ansible-playbook playbooks/backup-mikrotik-system.yml --limit sw-mkt-mgmt-01

```
Mikrotik system files will be stored in the ansible control host in `playbook-network-switches/backups/`

### mikrotik file transfers

Be aware that ansible-pyblibssh won't work to transfer files from/to the mikrotik devices using ansible module `ansible.netcommon.net_get`. 

You have to install the packages defined in `requirements.txt` inside your venv (paramiko + scp)



---

## Example runs

### Ansible dry run and show diff

```bash
ansible-playbook playbooks/mikrotik-configure.yml --limit clab-nw-1mkt-simple-n1 --check --diff

```

---

## Debugging

### Check the ansible groups for a device in the inventory

```
$> ansible -i inventory/ -m debug -a var="hostvars[inventory_hostname]['group_names']" sw-mkt-cluster-05
```

---

## Caveats

### Details about bonding

Make sure the members of a bond don't belong to the bridge. This requires the right order in the playbook. You should first execute the task
configuring the bridge so the right interfaces are added/removed from/to the bridge and **afterwards** you can execute the task configuring the bond.

### Details about trunk ports

Every trunk port should have `bpdu_guard: no` in `interface_brige_ports.yml`

### common error

```
TASK [Bond] **************************************************************************************************************************************************************************************************************************************************
fatal: [sw-mkt-03]: FAILED! => changed=false
  msg: 'Error while creating entry for name="uplink-bond": failure: sfp-sfpplus3 already in bridge'
  changed: [sw-mkt-02]
```

This happens because the interfaces were manually added to the bridge during the initial setup. To fix it login to the switch and:

```
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

  * private ssh key used for authentication should be located in `~/.ssh/id_rsa_priv`

  Use ansible-vault to provide login- and API-credentials.
  This addition helps in daily operations to keep the credentials i a securere standard place.

  * ansible vault password should be stored in a text file in `playbook-network-switches/.vault.pass`
  * The ansible vault password is stored in the password db with name "playbook-network-switches vault"

---

*Happy automating!*
