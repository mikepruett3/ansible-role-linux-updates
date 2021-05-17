Ansible Role: Linux Updates
=========

Ansible role to deploy system updates on Linux Servers.

Requirements
------------

The role does not require anyting to run on RHEL and its derivatives.

Role Variables
--------------

Available variables are listed below, along with default values (see ```defaults/main.yml```):

``` yaml
smtp_host: "smtp.example.org"
smtp_port: 25
sender: "myhost@example.org"
recipient: "it@example.org"
```

```smtp_host``` **(Required)** The hostname of a local or remote SMTP server to use.

```smtp_port``` **(Required)** The port to use of the SMTP server (usually **25**)

```sender``` **(Required)** The email address to use for the sender (i.e. **FROM**).

```recipient``` **(Required)** The email address to send the report to (i.e. **TO**).

Role variables can be stored with the hosts.yaml file, or in the main variables file.

Dependencies
------------

None.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

``` yaml
    - hosts: servers
      roles:
         - role: mikepruett3.linux-updates
```

License
-------

MIT

Author Information
------------------

Role created by [mikepruett3](https://github.com/mikepruett3) on [Github.com](https://github.com/mikepruett3/ansible-role-linux-updates)
