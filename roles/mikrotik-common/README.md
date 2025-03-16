Role Name
=========

Configure common MikroTik settings:
- Hostname
- DNS
- System time
- Syslog

Requirements
------------

None.

Role Variables
--------------

- `routeros_system_identity` - hostname.
- `routeros_ip_dns` - DNS settings.
- `routeros_system_clock` - system clock settings.
- `routeros_system_ntp_client` - NTP settings.
- `routeros_system_logging_action` - system logging actions configuration.
- `routeros_system_logging` - system logging configuration (events to actions mapping).

Dependencies
------------

None.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: mikrotik
      roles:
         - mikrotik-common

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
