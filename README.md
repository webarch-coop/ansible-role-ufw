# Webarchitects Uncomplicated Firewall (UFW) Ansible Role

[![pipeline status](https://git.coop/webarch/ufw/badges/master/pipeline.svg)](https://git.coop/webarch/ufw/-/commits/master)

An Ansible role to configure the Ubuntu [UFW - Uncomplicated Firewall](https://manpages.debian.org/ufw/ufw.8.en.html) on Debian and Ubuntu.

## Usage

An example configuration to only allow SSH and web taffic on ports 80 and 443 using IPv4:

```yml
ufw: true
ufw_allow_rules:
  - app: OpenSSH
  - app: WWW Full
ufw_config:
  - path: /etc/default/ufw
    conf:
      IPV6: "no"
```

Note that this role can't currently be used to delete rules, see the [#Notes](notes below) for how to check and delete UFW and other `iptables` rules.

## Role Variables

See the [defaults/main.yml](defaults/main.yml) file for the default variables, the [vars/main.yml](vars/main.yml) file for the preset variables and the [meta/argument_specs.yml](meta/argument_specs.yml) file for the variable specification.

### ufw

A boolean, when `ufw` is `true` the tasks in this role will be run.

### ufw_allow_rules

A list of UFW allow rules, each item in the list must either have a `app` or `port` variable, additional optional variables are `from_ip` and `proto`, for example:

```yml
ufw_allow_rules:
  - app: WWW Full
  - port: 2222
    proto: tcp
    comment: SSH
```

#### app

A string, the name of the app, it must match one of those listed using `ufw app list`.

#### comment

An optional comment that is shown at the end of the rule line, this is appended after `Ansible rule`, for example:

```yml
ufw_allow_rules:
  - port: 64896
    proto: udp
    comment: foo
```

```bash
ufw status
Status: active

To                         Action      From
--                         ------      ----
64896/udp                  ALLOW       Anywhere                   # Ansible rule foo
```

#### from_ip

An optional string to use with the `from_ip` parameter of the [community.general.ufw module](https://docs.ansible.com/ansible/latest/collections/community/general/ufw_module.html#parameter-from_ip).

#### port

An optional string containing a port number or a range of ports seperated with a colon.

### ufw_apps

An optional list of UFW application profiles to create or edit in the `/etc/ufw/applications.d/` directory, each item in the list requires a `app`for the application name, title and file name if `path` is not specified, a `desc` for the applicatiion description and a `ports` string. See the [application integration](https://manpages.debian.org/ufw/ufw.8.en.html#APPLICATION_INTEGRATION) section of the UFW manpage and the following example:

```yml
ufw_apps:
  - app: MariaDB
    desc: The open source relational database.
    ports: 3306/tcp
```

Will generate a file at `/etc/ufw/applications.d/mariadb` wth the contents:

```ini
# Ansible managed

[MariaDB]
title=MariaDB
description=The open source relational database.
ports=3306/tcp
# vim: syntax=dosini
```

#### app

A required string, the application name, which is also used as the application title (note that if this role is used to edit existng apps they might have a `title` that doesn't match the `app` name.

#### desc

A string, an optional application description, the title is used if a description is not provided and if neither is provide the application name is used.

#### path

A string, an optional full path to the application file, if one is not provided then the application name, lower cased and special characters removed. is used as the filename.

#### ports

A string, a `|`-separated list of ports/protocols where the protocol is optional. A comma-separated list or a range (specified with 'start:end') may also be used to specify multiple ports, in which case the protocol is required, see the examples in the [application integration](https://manpages.debian.org/ufw/ufw.8.en.html#APPLICATION_INTEGRATION) section of the UFW manpage.

### ufw_config

A optional list of INI configuration files and configuration to update, see the [configuration files](#configuration-files) documentation below, each itm in the list requires a `path` and a `conf` dictionary. For example:

```yaml
ufw_config:
  - path: /etc/default/ufw
    conf:
      IPV6: "no"
```

Will result in `/etc/default/ufw` containing:

```ini
# Set to yes to apply rules to support IPv6 (no means only IPv6 on loopback
# accepted). You will need to 'disable' and then 'enable' the firewall for
# the changes to take affect.
IPV6=no
```

#### path

A string, the full path to the configuration file.

#### conf

A dictionary, variables and values that should be present in the configuration file.

### ufw_disallow_rules

A optional list of UFW disallow rules, each item in the list must either have a `app` or `port` variable, additional optional variables are `from_ip` and `proto`.

#### app

A string, the name of the app, it must match one of those listed using `ufw app list`.

#### comment

An optional comment that is shown at the end of the rule line.

#### from_ip

An optional string to use with the `from_ip` parameter of the [community.general.ufw module](https://docs.ansible.com/ansible/latest/collections/community/general/ufw_module.html#parameter-from_ip).

#### port

An optional string containing a port number or a range of ports seperated with a colon.

### ufw_default_policy_deny

A boolean, when try set the default policy to deny connections.

### ufw_pkgs

A list of packages to install.

### ufw_verify

A boolean, verify variables that start with `ufw_`.

## Configuration Files

UFW configuration files that use the INI format can be created, deleted and updated using this role see the [ufw-framework](https://manpages.debian.org/ufw/ufw-framework.8.en.html) man page.

### /etc/default/ufw

The `/etc/default/ufw` high level configuration file can be configured using an item in the `ufw_config` list, the existing config can be read as YAML using:

```bash
cat /etc/default/ufw | jc --ini -yp
```
```yaml
---
IPV6: yes
DEFAULT_INPUT_POLICY: DROP
DEFAULT_OUTPUT_POLICY: ACCEPT
DEFAULT_FORWARD_POLICY: DROP
DEFAULT_APPLICATION_POLICY: SKIP
MANAGE_BUILTINS: no
IPT_SYSCTL: /etc/ufw/sysctl.conf
IPT_MODULES: ''
```

### /etc/ufw/sysctl.conf

The `/etc/ufw/sysctl.conf` kernel network tunables file can be configured using an item in the `ufw_config` list, the existing config can be read as YAML using:

```bash
cat /etc/ufw/sysctl.conf | jc --ini -yp
```
```yaml
---
net/ipv4/conf/all/accept_redirects: '0'
net/ipv4/conf/default/accept_redirects: '0'
net/ipv6/conf/all/accept_redirects: '0'
net/ipv6/conf/default/accept_redirects: '0'
net/ipv4/icmp_echo_ignore_broadcasts: '1'
net/ipv4/icmp_ignore_bogus_error_responses: '1'
net/ipv4/icmp_echo_ignore_all: '0'
net/ipv4/conf/all/log_martians: '0'
net/ipv4/conf/default/log_martians: '0'
```

### /etc/ufw/ufw.conf

The `/etc/ufw/ufw.conf` additional high level configuration file can be configured using an item in the `ufw_config` list, the existing config can be read as YAML using:

```bash
cat /etc/ufw/ufw.conf | jc --ini -yp
```
```yaml
---
ENABLED: no
LOGLEVEL: low
```

### /etc/ufw/applications.d/*

UFW [application integration profiles](https://manpages.debian.org/bullseye/ufw/ufw.8.en.html#APPLICATION_INTEGRATION) in the `/etc/ufw/applications.d/*` directory can be configured using the `ufw_apps` list, the existing config can be read as YAML using:

```bash
cat /etc/ufw/applications.d/* | jc --ini -yp
```
```yaml
---
CUPS:
  title: Common UNIX Printing System server
  description: CUPS is a printing system with support for IPP, samba, lpd, and other protocols.
  ports: '631'
mosh:
  title: Mosh (mobile shell)
  description: Mobile shell that supports roaming and intelligent local echo
  ports: 60000:61000/udp
OpenSSH:
  title: Secure shell server, an rshd replacement
  description: OpenSSH is a free implementation of the Secure Shell protocol.
  ports: 22/tcp
```

### Notes

Update UFW directly, deleting firewall rules:

```bash
ufw status numbered
```
```
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] OpenSSH                    ALLOW IN    Anywhere                   # Ansible rule
[ 2] WWW Full                   ALLOW IN    Anywhere                   # Ansible rule
[ 3] Turnserver                 ALLOW IN    Anywhere                   # Ansible rule
[ 4] WWW Cache                  ALLOW IN    Anywhere                   # Ansible rule
[ 5] RabbitMQ                   ALLOW IN    127.0.0.1                  # Ansible rule
[ 6] Munin                      ALLOW IN    81.95.52.37                # Ansible rule
[ 7] Icinga                     ALLOW IN    81.95.52.42                # Ansible rule
[ 8] MariaDB                    ALLOW IN    127.0.0.1                  # Ansible rule
[ 9] Redis                      ALLOW IN    127.0.0.1                  # Ansible rule
[10] Nextcloud Notify Push      ALLOW IN    127.0.0.1                  # Ansible rule
[11] WWW Cache                  ALLOW IN    127.0.0.1                  # Ansible rule
```
```bash
ufw delete 4
```
```
Deleting:
 allow 'WWW Cache' comment 'Ansible rule'
Proceed with operation (y|n)? y
Rule deleted
```
```bash
ufw status numbered
```
```
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] OpenSSH                    ALLOW IN    Anywhere                   # Ansible rule
[ 2] WWW Full                   ALLOW IN    Anywhere                   # Ansible rule
[ 3] Turnserver                 ALLOW IN    Anywhere                   # Ansible rule
[ 4] RabbitMQ                   ALLOW IN    127.0.0.1                  # Ansible rule
[ 5] Munin                      ALLOW IN    81.95.52.37                # Ansible rule
[ 6] Icinga                     ALLOW IN    81.95.52.42                # Ansible rule
[ 7] MariaDB                    ALLOW IN    127.0.0.1                  # Ansible rule
[ 8] Redis                      ALLOW IN    127.0.0.1                  # Ansible rule
[ 9] Nextcloud Notify Push      ALLOW IN    127.0.0.1                  # Ansible rule
[10] WWW Cache                  ALLOW IN    127.0.0.1                  # Ansible rule
```

List rules using `iptables`:

```bash
iptables -L -n
```

Dump, edit and restore using `iptables`:

```bash
iptables-save > /etc/iptables/rules.v4
vi /etc/iptables/rules.v4
iptables-restore < /etc/iptables/rules.v4
```

## References

* https://docs.ansible.com/ansible/latest/collections/community/general/ufw_module.html
* https://kellyjonbrazil.github.io/jc/docs/parsers/ufw
* https://wiki.debian.org/Uncomplicated%20Firewall%20%28ufw%29
* https://wiki.ubuntu.com/UncomplicatedFirewall

## Repository

The primary URL of this repo is [`https://git.coop/webarch/ufw`](https://git.coop/webarch/ufw) however it is also [mirrored to GitHub](https://github.com/webarch-coop/ansible-role-ufw) and [available via Ansible Galaxy](https://galaxy.ansible.com/chriscroome/ufw).

If you use this role please use a tagged release, see [the release notes](https://git.coop/webarch/ufw/-/releases).

## Copyright

Copyright 2020-2023 Luke Murphy and Chris Croome, &lt;[chris@webarchitects.co.uk](mailto:chris@webarchitects.co.uk)&gt;.

This role is released under [the same terms as Ansible itself](https://github.com/ansible/ansible/blob/devel/COPYING), the [GNU GPLv3](LICENSE).
