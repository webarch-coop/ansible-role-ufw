# Webarchitects Uncomplicated Firewall (UFW) Ansible Role

[![pipeline status](https://git.coop/webarch/ufw/badges/master/pipeline.svg)](https://git.coop/webarch/ufw/-/commits/master)

Ansible role to configure the Ubuntu [UFW - Uncomplicated Firewall](https://help.ubuntu.com/community/UFW).

## Usage

An example configuration to only allow SSH and Mosh:

```yml
ufw: true
ufw_allow_rules:
  - port: ssh
    proto: tcp
  - port: 60000:61000
    proto: udp
```

## References

* https://docs.ansible.com/ansible/latest/collections/community/general/ufw_module.html
* https://kellyjonbrazil.github.io/jc/docs/parsers/ufw
* https://wiki.ubuntu.com/UncomplicatedFirewall

## Repository

The primary URL of this repo is [`https://git.coop/webarch/ufw`](https://git.coop/webarch/ufw) however it is also [mirrored to GitHub](https://github.com/webarch-coop/ansible-role-ufw) and [available via Ansible Galaxy](https://galaxy.ansible.com/chriscroome/ufw).

If you use this role please use a tagged release, see [the release notes](https://git.coop/webarch/ufw/-/releases).

## Copyright

Copyright 2020-2023 Luke Murphy and Chris Croome, &lt;[chris@webarchitects.co.uk](mailto:chris@webarchitects.co.uk)&gt;.

This role is released under [the same terms as Ansible itself](https://github.com/ansible/ansible/blob/devel/COPYING), the [GNU GPLv3](LICENSE).
