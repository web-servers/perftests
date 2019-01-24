Role Name
=========

TODO

Requirements
------------

. Enable [WinRM](https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1).
. Install pywinrm: `pip install --upgrade --user setuptools && pip install pywinrm`
. setup variables:

```[jws:vars]
ansible_user=$user
ansible_password=$password
ansible_connection=winrm
ansible_winrm_server_cert_validation=ignore
```

Role Variables
--------------

TODO


Author Information
------------------

Marek Czernek <mczernek at redhat dot com> for Red Hat.
