# DFD Debian Bare Metal Cloud-Init

Daniel F. Dickinson's Cloud-Init deployment for a bare-metal host for a
'personal infrastructure'. This repository provides the base configuration
upon which Ansible will build.

The base configuration is very basic and aims to act like an OVH Debian 11
(Bullseye) dedicated server's base configuration.

This repository should not include any private or even particularly personalised
information in unencrypted form, and is designed for DFD's uses, so likely not
a 'plug and play' option for others. It is Daniel's hope, however, that it still
proves informative and
useful.

_Aside_: This method of deployment is much faster at actual deploy time than
using a `debian-installer` based installation. Of course that is because adding
packages, setting up users, etc, is work you've done ahead of time, and gets
applied automagically while you (e.g.) have a coffee. See [my related Ansible
repository](https://gitlab.com/danielfdickinson/debian-libvirt-ansible-dfd) for
an example of the main install.

## Metadata

### Demo and/or documentation site or page

Not yet created.

### Repository URL

<https://gitlab.com/danielfdickinson/debian-bare-metal-cloud-init-dfd>

## Features and default configuration

* Sample files for a
	[`NoCloud`](https://cloudinit.readthedocs.io/en/20.4.1/topics/datasources/nocloud.html)
	partition for a bare-metal server deployed using a 'cloud' image and
	cloud-init data.
* Currently a documentation rather than code project.
	* (Daniel's initial plan was to automate more complex user-data, but
	that proves unnecessary for the basic configuration that is being imitated).

## Using this repository

This repository is meant for the case where you are only rarely deploying on
physical hosts (in Daniel's case the goal is to 'pre-test' deployments that will
be on dedicated servers on OVH for the real deal). If one frequently needs to do
this type of deployment, it is instead recommended to set up a proper MAAS
(Metal As A Service) environment.

### Preparing the 'cloud-init' (cloud-config) data

* Add your hostname, FQDN, and an SSH public key (for which you have the private
	key) in both `meta-data` and `user-data`
* Update the `network-config` to suit your machine
* Optionally, add more disk and filesystem setup to the `user-data` file
* Optionally, unlock the initial deploy user and add a password (if you want to
	allow console logins immediately, this would be required).

### Deploying on the host

1. Obtain a Debian 'generic' cloud image such as
<https://cloud.debian.org/images/cloud/bullseye/latest/debian-11-generic-amd64.tar.xz>
	If you will have a network available when accessing the target host's
	storage you may be able to download the image directly onto the desired
	disk. For example you could do (assuming the host's root partition belongs on
	/dev/sdb):

	```bash
	curl https://cloud.debian.org/images/cloud/bullseye/latest/debian-11-generic-amd64.tar.xz | tar -O -xJf - disk.raw | dd of=/dev/sdb bs=1M
	```

	* **Note** that using this technique writes a whole disk image to the target
	disk, not a single partition or individual files.
	* **Note 2** If you have access to the storage hardware by itself, you may
	not need to	boot the system. If the system needs to be booted to access the
	hardware, a 	system rescue image such as <https://www.system-rescue.org/>
	can be help. The rescue image can also be handy for other command line system
	recovery situations).
2. At the _end_ of the boot _disk_ (not partition) add a small FAT32 partition
	with the filesystem volume label `CIDATA`. In that partition you should add
	the files `meta-data`, `user-data`, and optional `network-config` to the root
	of the filesystem. For more information see the [cloud-init 20.4.1 'NoCloud
	documentation](https://cloudinit.readthedocs.io/en/20.4.1/topics/datasources/nocloud.html).
	(We follow the 20.4.1 documentation because that is the version used with
	Debian Bullseye cloud images).
	* Example session to be added

## Getting help, discussing, and/or modifying

TBD

-------

## Colophon

* [Copyright and licensing](LICENSE)
* [Inspirations, information, and source material](ACKNOWLEDGEMENTS.md)
* [Notes](README-NOTES.md)
