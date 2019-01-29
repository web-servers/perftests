# Webservers Performance Tests

The following is repository contains ansible role as well as an ansible execute script
used for JWS performance testing using JMeter.

## Prerequisites

On the host server, install Ansible:

```
$ ansible --version
ansible 2.7.5
  config file = /tmp/ansible/ansible.cfg
  configured module search path = [u'/home/hudson/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Sep 12 2018, 05:31:16) [GCC 4.8.5 20150623 (Red Hat 4.8.5-36)]
```

On the JWS server, if it is a Windows server:

1. Enable [WinRM](https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1).
1. Install pywinrm: `sudo pip install --upgrade --user setuptools && sudo pip install pywinrm`

## Inventory file

The `execute-role.yml` playbook is configured such that it looks for an inventory
file in the `hosts` directory. The role is created such that it accepts both Windows
as well as RHEL as the JWS server OS, but you have to specify it in the inventory file.

If your JWS runs on Windows, the `hosts/inventory` file will, for example, look as such:

```
[jws]
10.0.0.1

[jmeter]
10.0.0.2

[jws:vars]
ansible_user=username
ansible_password=password
ansible_connection=winrm
ansible_winrm_server_cert_validation=ignore

[jmeter:vars]
ansible_connection=ssh
private_key_file=/path/to/id_rsa
ansible_ssh_private_key_file=/path/to/id_rsa
ansible_user=username
```

For Windows, `winrm` must be specified. You can test whether the connection is
working with:

```
$ ansible jws -m win_ping
10.0.0.1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

$ ansible jmeter -m ping
10.0.0.2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Role Variables

The role expects the following variables to be exported:

- `URL_BASE` - the URL base from where you download the JWS.
- `JMETER_TARGET` - the IP address of the JWS server accessible from the JMeter server.
This IP address will be used for performance testing.


## Author Information

Marek Czernek <mczernek at redhat dot com> for Red Hat.
