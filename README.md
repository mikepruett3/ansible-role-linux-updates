Ansible Role: Linux Updates
=========

Ansible role to patch Linux servers (RHEL-family via `yum`/`dnf`, Debian/Ubuntu
via `apt`), reboot if the kernel changed, clean up old kernels and unneeded
dependencies, and email a report of the available updates.

What it actually does, per OS family:

- **RedHat family** (`tasks/RedHat.yaml`): collects the list of available
  updates (`yum list updates` / `dnf list --updates`), installs them,
  reboots if the running kernel no longer matches the latest installed one,
  then (if `kernel_cleanup` is true) removes old kernels/dependencies and
  cleans the package cache.
- **Debian family** (`tasks/Debian.yaml`): installs `needrestart` and sets it
  to auto-restart services non-interactively, deploys a dpkg options file
  (`--force-confdef`/`--force-confold`, to avoid interactive conffile
  prompts), collects the list of upgradable packages, runs a
  regular/full/dist upgrade, and on Ubuntu specifically, if none of those
  three upgrades changed anything, also checks for and performs a release
  upgrade (`do-release-upgrade`). Reboots if the kernel changed, then always
  runs `apt autoremove --purge` and `apt autoclean` regardless of
  `kernel_cleanup` (that variable only gates kernel/dependency cleanup on
  the RedHat side, see below).
- **Both**: sends an email report via `notification.yaml`. Note the report
  is built from the *available-updates* listing collected before the
  upgrade runs, not a diff of what was actually installed, so the email
  reports what was out of date going into the run.

Requirements
------------

- The `community.general` collection, for the `mail` module used in
  `notification.yaml` (this isn't an `ansible.builtin` module). The role
  itself has no `requirements.yml` or `meta/main.yml` dependency entry for
  this, so install it separately: `ansible-galaxy collection install
  community.general`.
- Network access from the Ansible control node to the SMTP server/port you
  configure, the notification task always attempts to send mail,
  `delegate_to: localhost`.
- On RHEL-family hosts: nothing beyond a working `yum`/`dnf`, the role
  installs `yum-utils` itself when needed (and removes it if present where
  it shouldn't be, i.e. a `dnf`-based host).

Role Variables
---------------

Available variables are listed below, along with default values (see `defaults/main.yml`):

``` yaml
smtp_host: "smtp.example.org"
smtp_port: 25
sender: "myhost@example.org"
recipient: "it@example.org"

kernel_cleanup: true
```

`smtp_host` **(Required)** The hostname of a local or remote SMTP server to use.

`smtp_port` **(Required)** The port to use of the SMTP server (usually **25**).

`sender` **(Required)** The email address to use for the sender (i.e. **FROM**).

`recipient` **(Required)** The email address to send the report to (i.e. **TO**).

`kernel_cleanup` **(Required)** Whether to remove old kernels/unneeded dependencies.
Only gates that cleanup on RedHat-family hosts (`tasks/kernel.yaml`), Debian/Ubuntu
hosts always run `apt autoremove --purge`/`apt autoclean` regardless of this setting.

Role variables can be stored with the `hosts.yaml` file, or in the main variables file.

A `vars/` directory (`RedHat_5.yaml`, `RedHat_7.yaml`, `RedHat_8.yaml`,
`RedHat_9.yaml`, `default.yaml`) also exists in this repo from an earlier
version of the role that varied its kernel-cleanup command and journal query
per RHEL major version. No task currently includes these (no `vars_files:`
or `include_vars:` anywhere), the logic they once drove is now inlined
directly in `tasks/RedHat.yaml`/`tasks/kernel.yaml` instead. Likewise,
`handlers/main.yml`'s reboot handler is never `notify:`'d from any task,
each OS-family task reboots inline on its own instead.

Dependencies
------------

- Collection: `community.general` (for the `mail` module).
- No other Galaxy roles.

Example Playbook
----------------

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
