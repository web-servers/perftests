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

Additionally, install pywinrm if your JWS server is a Windows server:

`sudo pip install --upgrade --user setuptools && sudo pip install pywinrm`

On the JWS server, if it is a Windows server, Enable [WinRM](https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1).

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

Note that the role expects the groups to be named `jws` and `jmeter`.

## Limitations

### Java Home cannot be on an NFS on Windows
_Problem_: For the Windows platform, if you execute `win_command: java -version` and
your `%path%` contains java binary that is on an NFS, you will receive an error.

_Solution_: To work around the issue:

1. Move Java on local storage
1. Set global path so that it contains the local Java path:

`setx path "C:\Users\hudson.MY-WIN2016-MACH\Desktop\jdk1.8.0_last\bin;%path%"`

_Verification_: Before you start testing, you can verify that Ansible is able
to execute Java with the following:

```
$ ansible jws -m win_command -a "java -version"
10.0.0.1 | CHANGED | rc=0 >>
java version "1.8.0_191"
Java(TM) SE Runtime Environment (build 1.8.0_191-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.191-b12, mixed mode)
```

## Role Variables

The role expects the following variables to be exported:

- `URL_BASE` - the URL base from where you download the JWS.
- `JMETER_TARGET` - the IP address of the JWS server accessible from the JMeter server.
This IP address will be used for performance testing.
- `SSL_CONF` - the name of configuration file that is to be used by JWS. It has to
be present in the `templates` dir. At this moment, it can have values: `apr` or `jsse`.
The `.xml` is added in the task itself. If not specified, `jsse` is used.

Before executing the scripts, be sure to export the variables:

```
export URL_BASE="https://myhost.com/released/jboss/webserver"
export JMETER_TARGET=10.0.0.1
export SSL_CONF=apr
```

## Executing the test

Before executing the tests, verify that:

1. You have exported necessary variables
1. You have configured your inventory file, your servers, and Ansible can ping them
1. If your JWS server is Windows, you have ensured Java is on local storage

To execute the test, issue:

`ansible-playbook execute-role.yml`

## Author Information

Marek Czernek <mczernek at redhat dot com> for Red Hat.
