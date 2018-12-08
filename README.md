Lsyncd Configurations
=====================

[![Build Status](https://travis-ci.org/hanru/ansible-lsyncd.svg?branch=master)](https://travis-ci.org/hanru/ansible-lsyncd)

Overview
--------

This Ansible role configures synchronization from *master* host to *slave* host(s). [Lsyncd](https://github.com/axkibe/lsyncd) daemon on master is responsible for synchronization. There's only one master whereas there can be more than one slaves. There can also be more than one synchronizing *sources* (on master) and *targets* (on slave).

On master, the lsyncd daemon is running as *root*. On slaves, a dedicated *lsyncd* user is created. This user is capable of executing `sudo rsync` so that synced files can have permissions preserved. When synchronizing, root `ssh` into slaves as lsyncd user and executing `rsync` command there.

Because lsyncd is running in the background, a passwordless SSH key pair is generated for root on master. The corresponding public key is dispatched to every slave.

On master, lsyncd status file locates at `/var/lib/lsyncd/status`, lsyncd log locates at `/var/log/lsyncd.log`. You should check these files regularly in case of possible problems.

This role has been successfully tested on Debian Jessie (8.x), Debian Stretch (9.x) and Ubuntu Trusty (14.04), Ubuntu Xenial (16.04).

Role Variables
--------------

    lsyncd_inotify_max_watches: 65536

The maximum possible number of directories lsyncd watches. This is a Linux kernel limit (fs.inotify.max_user_watches) which defaults to 8192 on Debian. Consider raise this number when lsyncd is watching huge number of files.

    lsyncd_master_hostname:

The hostname of the master. Should be one `inventory_hostname` in inventory. There's only one master. See example below.

    lsyncd_master_identity_file: /root/.ssh/id_rsa_lsyncd

The filename of root user's SSH private key on master. A new key pair will be generated if the file doesn't exist.

    lsyncd_slave_hosts: []

A list of FQDN or IP address of slave hosts. There can be multiple slaves.

    lsyncd_slave_username: lsyncd

The username on the slaves which master uses to login via SSH.

    lsyncd_sources_and_targets: []

A list of sources on master which are to sync with targets on slaves. There can be multiple source and target pairs. See example below.

    lsyncd_settings_status_interval: 10
    lsyncd_sync_delay: 15

Lsyncd fine tuning parameters. Don't change these unless you know what you are doing.

Dependencies
------------

This role has no dependencies.

Example Playbook
----------------

First prepare an [inventory](https://docs.ansible.com/ansible/latest/intro_inventory.html). We use the following inventory in our example.

    [lsyncd]
    test1       ansible_host=10.0.0.1       ansible_user=root
    test2       ansible_host=10.0.0.2       ansible_user=root
    test3       ansible_host=10.0.0.3       ansible_user=root

Note that Ansible `inventory_hostname` variables are `test1`, `test2` and `test3` for this inventory. One of them must be the master host. We choose `test1` as our master. The other two will serve as slavs.

Here's an example playbook.

    - hosts: lsyncd
      vars:
        lsyncd_master_hostname: test1
        lsyncd_slave_hosts:
          - 10.0.0.2
          - 10.0.0.3
        lsyncd_sources_and_targets:
          - source: /var/www
            target: /var/www
          - source: /home
            target: /backup/home
      roles:
        - { role: hanru.lsyncd }

License
-------

MIT

Reference
---------

* [Lsyncd manual](https://axkibe.github.io/lsyncd/)
* [Lsyncd issue: synchronizing multiple directories in multiple servers](https://github.com/axkibe/lsyncd/issues/376)
* [Lsyncd and rsync remote sudo settings](https://snickerjp.blogspot.com/2015/10/lsyncd-rsync-sudo.html)