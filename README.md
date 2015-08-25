[![Build Status](https://travis-ci.org/willshersystems/ansible-sshd.svg?branch=master)](https://travis-ci.org/willshersystems/ansible-sshd) [![Ansible Galaxy](http://img.shields.io/badge/galaxy-mattwillsher.sshd-660198.svg?style=flat)](https://galaxy.ansible.com/list#/roles/4868)

OpenSSH Server
==============

This role configures the OpenSSH daemon. It:

* By default configures the SSH daemon with the normal OS defaults.
* Works across a variety of UN*X like distributions
* Can be configured by dict or simple variables
* Supports Match sets
* Supports all sshd_config options. Templates are programmatically generated.
  (see [meta/make_option_list](meta/make_option_list))
* Tests the sshd_config before reloading sshd.

**WARNING** Misconfiguration of this role can lock you out of your server!
Please test your configuration and its interaction with your users configuration
before using in production!

**WARNING** Digital Ocean allows root with passwords via SSH on Debian and
Ubuntu. This is not the default assigned by this module - it will set
`PermitRootLogin without-password` which will allow access via SSH key but not
via simple password. If you need this functionality, be sure to set
`ssh_PermitRootLogin yes` for those hosts.

Requirements
------------

Tested on:

* Ubuntu precise, trusty
* Debian wheezy, jessie
* FreeBSD 10.1
* EL 6,7 derived distributions
* Fedora 20, 22

It will likely work on other flavours and more direct support via suitable
[vars/](vars/) files is welcome.

Role variables
---------------

Unconfigured, this role will provide a sshd_config that matches the OS default,
minus the comments and in a different order.

* sshd_skip_defaults

If set to True, don't apply default values. This means that you must have a
complete set of configuration defaults via either the sshd dict, or sshd_Key
variables. Defaults to *False*.

* sshd_manage_service

If set to False, the service/daemon won't be touched at all, i.e. will not try
to enable on boot or start or reload the service.  Defaults to *True* unless
running inside a docker container (it is assumed ansible is used during build
phase).

* sshd_allow_reload

If set to False, a reload of sshd wont happen on change. This can help with
troubleshooting. You'll need to manually reload sshd if you want to apply the
changed configuration. Defaults to the same value as ``sshd_manage_service``.

* sshd

A dict containing configuration.  e.g.

```yaml
sshd:
  Compression: delayed
  ListenAddress:
    - 0.0.0.0
```

* ssh_...

Simple variables can be used rather than a dict. Simple values override dict
values. e.g.:

```yaml
sshd_Compression: off
```

In all cases, booleans correctly rendered as yes and no in sshd configuration.
Lists can be used for multiline configuration items. e.g.

```yaml
sshd_ListenAddress:
  - 0.0.0.0
  - '::'
```

Renders as:

```
ListenAddress 0.0.0.0
ListenAddress ::
```

* sshd_match

A list of dicts for a match section. See the example playbook.

* sshd_match_1 through sshd_match_9

A list of dicts or just a dict for a Match section.

Dependencies
------------

None

Example Playbook
----------------

```yaml
---
- hosts: all
  vars:
    sshd_skip_defaults: true
    sshd:
      Compression: true
      ListenAddress:
        - "0.0.0.0"
        - "::"
      GSSAPIAuthentication: no
      Match:
        - Condition: "Group user"
          GSSAPIAuthentication: yes
    sshd_UsePrivilegeSeparation: sandbox
    sshd_match:
        - Condition: "Group xusers"
          X11Forwarding: yes
  roles:
    - role: willshersystems.sshd
```

Results in:

```
# Ansible managed: ...
Compression yes
GSSAPIAuthentication no
UsePrivilegeSeparation sandbox
Match Group user
  GSSAPIAuthentication yes
Match Group xusers
  X11Forwarding yes
```

Template Generation
-------------------

The [sshd_config.j2](templates/sshd_config.j2) template is programatically
generated by the scripts in meta. New options should be added to the
options_body or options_match.

To regenerate the template, from within the meta/ directory run:
`./make_option_list >../templates/sshd_config.j2`

License
-------

LGPLv3


Author
------

Matt Willsher <matt@willsher.systems>

&copy; 2014,2015 Willsher Systems Ltd.
