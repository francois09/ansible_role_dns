DNS
===

Manage DNS & static /etc/resolv.conf

This role manage internal DNS using a dynamic inventory.

The standard for DNS zone is to use *.{{ bind__internal_domain }} for internal resolution and {{ bind__additional_zone }} for external domains.

The dynamic inventory gathers the hostnames, CNAMEs are defined explicitly in the vars. Other definitions in the variables included domains for which to forward and trusted domains that can perform queries.

Requirements
------------

You must set the `groups` openstack instance meta-data key for the nameserver to include the `nameserver` group.
Otherwise the resolv.conf part for each client will fail as it is looking for elements of the nameserver group.

Role Variables
--------------

* `bind__install:` for installing packages
* `bind__configure:` Set to true for DNS configuration
* `bind__client_configure:` Set to true for resolvconf configuration

* `bind__internal_domain:` define domain managed by DNS
* `bind__internal_network:` define where dynamic IP are collected
* `bind__logging:` should activate some logging
* `bind__static_nameservers:` use specific DNS for resolvconf

```
bind__static_ips:
  mybox: 192.168.0.254
  myprinter: 192.168.0.198
  ...
```

```
bind__cnames:
  music: "proxy.{{ internal_domain }}."
  daap: "proxy.{{ internal_domain }}."
  film: "proxy.{{ internal_domain }}."
```

```
bind__rpz_static:
  "*.adserver.com": 127.0.0.1
```

Dependencies
------------

N/A

Example Playbook
----------------

Using this kind of inventory:
```
[nameserver:children]
internal_dns
external_dns

[internal_dns]
dns1.mydomain.com
dns2.mydomain.com

[external_dns]
extdns.mydomain.com

```

You can run the following playbook:

```
hosts: all
  become: yes
  roles:
  - role: dns
    bind__client_configure: True

hosts: nameserver
  become: yes
  roles:
    - role: dns
      bind__install: True
      bind__configure: True
      bind__client_configure: True
```

/etc/resolv.conf is configured two different ways:

* By default, add IP addresses of all nameservers into the group ansible named 'nameserver'

* Define an ordered list of IP addresses to use for nameservers:

```
  bind__static_nameservers:
    - 1.1.1.1
    - 8.8.8.8
```

License
-------

GPL v3

Author Information
------------------

François TOURDE
