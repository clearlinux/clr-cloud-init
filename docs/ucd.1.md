micro-config-drive(1) -- Configure a cloud instance using config-drive data
===========================================================================

## SYNOPSIS

`/usr/bin/ucd`

`ucd` [OPTIONS...]

## DESCRIPTION

`ucd` runs at boot time with the purpose of configuring a cloud instance
for use of the end user.

ucd can perform any of the following (optional) tasks:

 * create a default account with sudo privileges
 * lock root and default account, the only way to login is using ssh keys (security feature)
 * maintain the cloud instance in a right state (resize/fix partitions and filesystem)
 * get and process userdata and metadata from an attached config-drive block device

Userdata formats supported:

 * `cloud-config`: begins with `#cloud-config` and is used to execute certain tasks in a human friendly format
 * `shell-script`: begins with `#!` and is used to execute a shell script

Metadata formats supported:

 * `openstack`

## OPENSTACK DATASOURCES

Datasources are used to retrive instance-specific data in order to configure a new cloud instance.
Openstack attaches two datasources to the cloud instance when it boots.

 * `config-drive`: is a disk formatted with vfat or iso9660 and have a label of config-2
 * `metadata service`: instances can access to it at http://169.254.169.254 `[NOT SUPPORTED]`

more information about openstack datasources can be found in:

 * http://docs.openstack.org/user-guide/cli_config_drive.html
 * http://docs.openstack.org/admin-guide/compute-networking-nova.html#metadata-service

## WORKFLOW

    `|` Parse options
    `|` Init openstack datasource
    `|->` if config-drive is found then define it as datasource otherwise exit
    `|` Start openstack datasource
    `|->` Get userdata and metadata from datasource
    `|` Run async tasks
    `|->` resize disk
    `|->` resize filesystem
    `|` If a datasource was found
    `|->` if VM first boot was detected
    `|   |->` Run async tasks
    `|   |   |->` lock root account
    `|   |   |->` Create sudoers file for default user account
    `|   |->` Create default user account
    `|->` Process openstack metadata
    `|->` Process openstack userdata
    `|` Join async tasks
    `|` Finish openstack datasource (free resources)

## OPTIONS

The following options are understood:

  * `-h`, `--help`:

    Prints a help message.

  * `-u` FILE, `--user-data-file` FILE:

    Path to a cloud-config user data file\&. If omitted, `ucd` will
    attempt to fetch user-data from the openstack link-local connected data
    service URL.

  * `--openstack-metadata-file` FILE:

    Path to an openstack metadata file.
    more information: http://docs.openstack.org/user-guide/cli_config_drive.html#openstack-metadata-format

  * `--openstack-config-drive` PATH:

    Path to openstack config drive (iso9660 or vfat filesystem),
    it will be used to process metadata and userdata.
    more information:
    http://docs.openstack.org/user-guide/cli_config_drive.html#configuration-drive-contents

  * `--user-data`:

    Get and process user data from data sources.

  * `--user-data-once`:

    Only on first boot get and process user data from data sources.
    Note this option is ignored if `--user-data` is enabled.

  * `--metadata`:

    Get and process metadata from data sources.

  * `--fix-disk`:

    Fix disk and filesystem if it is needed.

  * `-v`, `--version`:

    Prints version information.

  * `--first-boot-setup`:

    Setup the instance in its first boot.
    When first boot of the instance is detected, `ucd` will perform
    the following tasks: create a default user account, create sudoers
    file for default user account, lock the root account.

## EXIT STATUS

On success, 0 is returned, a non-zero failure code otherwise.

## COPYRIGHT

 * Copyright (C) 2017 Intel Corporation, License: CC-BY-SA-3.0

## SEE ALSO

This project is a limited-functionality implementation of the cloud-init
specification. The full documentation of the generic implementation is
available online and can be referenced here:

`https://cloudinit.readthedocs.org/en/latest/`

The cloud-config format that `ucd`(1) supports is documented
in `cloud-config`(5). This lists the options that are supported,
their structure and function.

## NOTES

Creative Commons Attribution-ShareAlike 3.0 Unported

 * http://creativecommons.org/licenses/by-sa/3.0/
