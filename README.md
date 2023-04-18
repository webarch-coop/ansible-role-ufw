# Webarchitects Uncomplicated Firewall (UFW) Ansible Role

[![pipeline status](https://git.coop/webarch/ufw/badges/master/pipeline.svg)](https://git.coop/webarch/ufw/-/commits/master)

An Ansible role to configure the Ubuntu [UFW - Uncomplicated Firewall](https://manpages.debian.org/ufw/ufw.8.en.html).

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

## Configuration

UFW configuration files that use the INI format can be created, deleted and updated using this role see the [ufw-framework](https://manpages.debian.org/ufw/ufw-framework.8.en.html) man page.

### /etc/default/ufw

The `/etc/default/ufw` high level configuration files can be configured using an item in the `ufw_config` list, the existing config can be read as YAML using:

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

UFW [application integration](https://manpages.debian.org/bullseye/ufw/ufw.8.en.html#APPLICATION_INTEGRATION) profiles in the `/etc/ufw/applications.d/*` directory can be configured using the `ufw_apps` list, the existing config can be read as YAML using:

```bash
cat /etc/ufw/applications.d/* | jc --ini -yp
```
```yaml
---
CUPS:
  title: Common UNIX Printing System server
  description: CUPS is a printing system with support for IPP, samba, lpd, and other
    protocols.
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

## Role Variables

See the [defaults/main.yml](defaults/main.yml) file for the default variables, the [vars/main.yml](vars/main.yml) file for the preset variables and the [meta/argument_specs.yml](meta/argument_specs.yml) file for the variable specification.

### ufw

A boolean, when `ufw` is `true` the tasks in this role will be run.

### ufw_allow_rules

A list of UFW allow rules, each item in the list must either have a `app` or `post` variable, additional optional variables are `from_ip` and `proto`, for example:

```yml
ufw_allow_rules:
  - app: WWW Full
  - port 2222
    proto: tcp
    comment: SSH
```

#### app

A string, the name of the app, it must match one of those listed using `ufw app list`.

##### comment

An optional comment that is shown at the end of the rule line, this is appended after `Ansible rule`, for example:

```bash
ufw status
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                   # Ansible rule
WWW Full                   ALLOW       Anywhere                   # Ansible rule
```

#### from_ip

An optional string to use with the `from_ip` parameter of the [community.general.ufw module](https://docs.ansible.com/ansible/latest/collections/community/general/ufw_module.html#parameter-from_ip).

### ufw_apps

### ufw_config

### ufw_disallow_rules

### ufw_default_policy_deny

### ufw_pkgs

### ufw_verify

### Notes

Updating UFW directly:

```bash
ufw status numbered
ufw delete 2
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
