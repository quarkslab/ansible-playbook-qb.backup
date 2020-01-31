# qb.backup - Example playbook

This repository contains an example playbook that configures three hosts:

- One FreeBSD 11.3 jail running on a FreeNAS 11.2-U7 host and serving as the
  backup server,
- Two Debian (>= 8) servers that need to be backuped.

It serves as an example of how to use Ansible roles
[qb.backup](https://github.com/quarkslab/ansible-role-qb.backup) and
[qb.backup_server](https://github.com/quarkslab/ansible-role-qb.backup_server).

## Requirements

### For the Ansible controller
This repository contains submodules, please ensure that they are initialized:
``` console
$ git submodule init
$ git submodule update
```

The `inventory.yml` file must be modified  with the correct values for your
setup, as well as all the various files in the `group_vars` folder.

This playbook has been tested with `ansible>=2.8.0`.

### For the FreeNAS server

Note: If you do not have a FreeNAS server, you can still setup the server-side
script of qb.backup on any other host. You will have to manually do the
installation yourself. You can adapt the role `qb.backup_server` for your needs.

The configuration of the FreeNAS server is not in the scope of this example. The
pre-requisites on the FreeNAS server are simple:

- The person running this Ansible playbook must be able to connect using a SSH
  key to the `root` user of the FreeNAS server,
- The FreeBSD jail must be created and running.
- The dataset used to store backups must be mounted in the jail in
  `/mnt/backup` using FreeNAS's "Mount points" jail option.

### For the FreeBSD Jail

Python 3.7 must be manually installed in the jail:
```console
# pkg bootstrap
# pkg update -f
# pkg install python37
```

### For the backuped hosts

Note: The role configuring backuped hosts targets Debian >= 8 servers. Any other
OS that can run SSH, borg and borgmatic can be used in this backup solution by
manually configuring those hosts.

You can manually setup hosts that need to be backuped by following the
instructions of the `README.md` file in the role `qb.backup`.

The following SSH server options must be enabled on the servers that need to be
backuped:
```sshd-config
# File: /etc/ssh/sshd_config
PermitRootLogin forced-commands-only # Or greater level of access
PermitUserEnvironment yes
```

Note: since qb.backup aims to take whole server backups, we connect using the
user `root` directly to avoid permission issues, so `sshd` must not deny SSH
connections to `root`.

The minimum level of authentication we support for `PermitRootLogin` is
`forced-commands-only`, but any level above would work (`prohibit-password` or
`yes`).

## Instructions

Once all the requirements are met, you can execute the playbook:
```console
$ ansible-playbook -i inventory.yml backup-playbook.yml
```

The playbook will fail the first time, just follow the instructions in the error
message to edit the variable `backup_clt__srv_ssh_pubkey`.

**Important note**: The backup client setup role saves on the Ansible controller
running the playbook one borg repository encryption key passphrase per backuped
host. By default, those passphrases are saved in `/tmp/backup-passphrases/`
which is a temporary folder erased at reboot, so make sure to move those
passphrases to a secure place as they will be needed to recover backups in case
of a loss of data!
