# Ansible role for configuring UFW

[![pipeline status](https://git.coop/webarch/ufw/badges/master/pipeline.svg)](https://git.coop/webarch/ufw/-/commits/master)

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
