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
	hardware, a system rescue image such as <https://www.system-rescue.org/>
	can be help. The rescue image can also be handy for other command line
	system recovery situations).

2. At the _end_ of the boot _disk_ (not partition) add a small FAT32 partition
	with the filesystem volume label `CIDATA`. In that partition you should add
	the files `meta-data`, `user-data`, and optional `network-config` to the root
	of the filesystem. For more information see the [cloud-init 20.4.1 'NoCloud
	documentation](https://cloudinit.readthedocs.io/en/20.4.1/topics/datasources/nocloud.html).
	(We follow the 20.4.1 documentation because that is the version used with
	Debian Bullseye cloud images).

3. <details>
	<summary>Example install session followed by first boot</summary>

	``` text
	GNU GRUB  version 2:2.06.r334.g340377470-1
	+----------------------------------------------------------------------------+
	|  Boot SystemRescue using default options                                   |
	|  Boot SystemRescue and copy system to RAM (copytoram)                      |
	| *Boot with serial and copy system to RAM (copytoram)                       |
	|  Boot SystemRescue and verify integrity of the medium (checksum)           |
	|  [...]                                                                     |
	+----------------------------------------------------------------------------+
	Use the ^ and v keys to select which entry is highlighted.
	Press enter to boot the selected OS, `e' to edit the commands
	before booting or `c' for a command-line.

	The highlighted entry will be executed automatically in 90s

	:: running early hook [udev]
	Starting version 251.6-2-arch

	[...]

	:: Mounting '/dev/disk/by-label/RESCUE905' to '/run/archiso/bootmnt'
	:: Device '/dev/disk/by-label/RESCUE905' mounted successfully.

	[...]

	Welcome to SystemRescue 9.05!

	[  OK  ] Created slice Slice /system/getty.
	[  OK  ] Created slice Slice /system/modprobe.
	[...]
	Starting D-Bus System Message Bus...
	Starting SSH Key Generation...
	Starting SystemRescue Init&hellip;lization, before networking...
	Starting User Login Management...
	[  OK  ] Started D-Bus System Message Bus.
	[  OK  ] Started User Login Management.
	[  OK  ] Listening on Load/Save RF &hellip;itch Status /dev/rfkill Watch.
	[...]
	[  OK  ] Reached target Preparation for Logins.
	[  OK  ] Started Getty on tty1.
	[  OK  ] Started Serial Getty on ttyS0.
	[  OK  ] Reached target Login Prompts.
	[  OK  ] Reached target Multi-User System.

	========= SystemRescue 9.05 (x86_64) ======== ttyS0/6 =========
	https://www.system-rescue.org/

	* Console environment :
		Run setkmap to choose the keyboard layout (also accessible with the arrow up key)
		Run manual to read the documentation of SystemRescue

	* Graphical environment :
		Type startx to run the graphical environment
		X.Org comes with the XFCE environment and several graphical tools:
		- Partition manager: .. gparted
		- Web browser: ........ firefox
		- Text editor: ........ featherpad

	sysrescue login: root (automatic login)

	[root@sysrescue ~]# lsblk -o NAME,SIZE,FSTYPE,LABEL,UUID,MOUNTPOINT
	NAME   SIZE FSTYPE LABEL     UUID              MOUNTPOINT
	loop0  656.6M squash                          /run/archi
	sda    931.5G
	├─sda1 930.5G ext4           e9d180ca-4d05-4e4f-b99e-8c49d6ff5ee7
	├─sda2 953M vfat   CIDATA    0E6B-8C93
	├─sda14 3M
	└─sda15 124M vfat            D2A8-932D
	sdb    1.8T
	└─sdb1 1.8T LVM2_m           etC2dD-f1ph-2Ilc-s9IS-0b9f-k3Zf-D40IOW
	sdc    1.8T
	└─sdc1 1.8T LVM2_m           fvu0Ic-eyX1-3Tpp-02k4-B4zv-tImL-bZ01et
	sdd    0B
	sde    0B
	sdf    0B
	sdg    0B
	sdh    1.7G iso966 RESCUE905 2023-01-31-01-59-58-00
	├─sdh1 730M iso966 RESCUE905 2023-01-31-01-59-58-00
	└─sdh2 1.4M vfat             7F86-83CD
	sr0    1024M

	[root@sysrescue ~]# eject /dev/sdh
	[root@sysrescue ~]# lsblk -oeNAME,SIZE,FSTYPE,LABEL,UUID,MOUNTPOINT
	NAME   SIZE FSTYPE LABEL     UUID              MOUNTPOINT
	loop0  656.6M squash                          /run/archi
	sda    931.5G
	├─sda1 930.5G ext4           e9d180ca-4d05-4e4f-b99e-8c49d6ff5ee7
	├─sda2 953M vfat   CIDATA    0E6B-8C93
	├─sda14 3M
	└─sda15 124M vfat            D2A8-932D
	sdb    1.8T
	└─sdb1 1.8T LVM2_m           etC2dD-f1ph-2Ilc-s9IS-0b9f-k3Zf-D40IOW
	sdc    1.8T
	└─sdc1 1.8T LVM2_m           fvu0Ic-eyX1-3Tpp-02k4-B4zv-tImL-bZ01et
	sdd    0B
	sde    0B
	sdf    0B
	sdg    0B
	sr0    1024M
	[root@sysrescue ~]# lsblk -o NAME,SIZE,FSTYPE,LABEL,UUID,MOUNTPOINT
	NAME   SIZE FSTYPE LABEL     UUID              MOUNTPOINT
	loop0  656.6M squash                          /run/archi
	sda    931.5G
	├─sda1 930.5G ext4           e9d180ca-4d05-4e4f-b99e-8c49d6ff5ee7
	├─sda2 953M vfat   CIDATA    0E6B-8C93
	├─sda14 3M
	└─sda15 124M vfat            D2A8-932D
	sdb    1.8T
	└─sdb1 1.8T LVM2_m           etC2dD-f1ph-2Ilc-s9IS-0b9f-k3Zf-D40IOW
	sdc    1.8T
	└─sdc1 1.8T LVM2_m           fvu0Ic-eyX1-3Tpp-02k4-B4zv-tImL-bZ01et
	sdd    0B
	sde    0B
	sdf    0B
	sdg    0B
	sr0    1024M
	sdh    7.3G ext4   Bare-metal via c   xxxxxxxxxxx-xxxxxx-xxxxx-xxxxx
	sr0   1024M
	[root@sysrescue ~]# mount /dev/sdh /mnt
	[root@sysrescue ~]# cd /mnt
	[root@sysrescue /mnt]# ls
	debian-11-generic-amd64-20230124-1270.tar.xz  lost+found  nocloud
	[root@sysrescue /mnt]# mkdir /media
	[root@sysrescue /mnt]# tar -O -xvJf debian-11-generic-amd64-20230124-1270.tar.xz disk.raw | dd of=/dev/sda1 bs=1M
	disk.raw
	0+190212 records in
	0+190212 records out
	2147483648 bytes (2.1 GB, 2.0 GiB) copied, 30.1568 s, 71.2 MB/s
	[root@sysrescue /mnt]# parted /dev/sda
	GNU Parted 3.5
	Using /dev/sda
	Welcome to GNU Parted! Type 'help' to view a list of commands.
	(parted) print
	Warning: Not all of the space available to /dev/sda appears to be used, you can
	fix the GPT to use all of the space (an extra 1949330864 blocks) or continue
	with the current setting?
	Fix/Ignore? fix
	Model: ATA CT1000BX100SSD1 (scsi)
	Disk /dev/sda: 1000GB
	Sector size (logical/physical): 512B/512B
	Partition Table: gpt
	Disk Flags:

	Number Start    End       Size      File system  Name  Flags
	14     1049kB   4194kB    3146kB    bios_grub
	15     4194kB    134MB    130MB     fat16              boot, esp
	1       134MB   2147MB   2013MB     ext4

	(parted) mkpart "" fat32 -1G -1
	(parted) print
	Model: ATA CT1000BX100SSD1 (scsi)
	Disk /dev/sda: 1000GB
	Sector size (logical/physical): 512B/512B
	Partition Table: gpt
	Disk Flags:

	Number Start    End       Size      File system  Name  Flags
	14     1049kB   4194kB    3146kB    bios_grub
	15     4194kB    134MB    130MB     fat16              boot, esp
	1       134MB   2147MB   2013MB     ext4
	2       999GB   1000GB    999MB     fat32              msftdata

	(parted) quit
	Information: You may need to update /etc/fstab.

	[root@sysrescue /mnt]# mkfs.vfat -F 32 -n CIDATA /dev/sda2
	mkfs.fat 4.2 (2021-01-31)
	[root@sysrescue /mnt]# mount /dev/sda2 /media
	[root@sysrescue /mnt]# ls /media/
	[root@sysrescue /mnt]# mount
	proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
	sys on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
	[...]
	tmpfs on /run/user/0 type tmpfs (rw,nosuid,nodev,relatime,size=3180068k,nr_inodes=795017,mode=700,inode64)
	/dev/sdh on /mnt type ext4 (rw,relatime)
	/dev/sda2 on /media type vfat (rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro)
	[root@sysrescue /mnt]# ls nocloud/
	meta-data  network-config  user-data
	[root@sysrescue /mnt]# cp nocloud/* /media/
	[root@sysrescue /mnt]# ls /media/
	meta-data  network-config  user-data
	[root@sysrescue /mnt]# umount /media
	[root@sysrescue /mnt]# cd /
	[rooejectt/dev/sdhloud/*o/media/a
	[root@sysrescue /]# mount
	proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
	sys on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
	[...]
	tmpfs on /run/user/0 type tmpfs (rw,nosuid,nodev,relatime,size=3180068k,nr_inodes=795017,mode=700,inode64)

	tar -O -xvJf debian-11-generic-amd64-20230124-1270.tar.xz disk.raw | dd of=/dev/mkdirs/media
	lsblk -o NAME,SIZE,FSTYPE,LABEL,UUID,MOUNTPOINT
	NAME    SIZE    FSTYPE  LABEL         UUID             MOUNTPOINT
	loop0    656.6M squashf                               /run/archis
	sda      931.5G
	├─sda1     1.9G ext4                    e9d180ca-4d05-4e4f-b99e-8c49d6ff5ee7
	├─sda2     953M vfat    CIDATA        9B59-C16F
	├─sda14      3M
	└─sda15    124M vfat                  D2A8-932D
	sdb        1.8T
	└─sdb1     1.8T LVM2_me               ftC6dA-fOpg-1Ikc-s0YS-0b9f-k3Zf-D40IOW
	sdc        1.8T
	└─sdc1     1.8T LVM2_me               fbu0Pc-esXk-3Too-01t5-B4zv-tImL-bZ01et
	sdd          0B
	sde          0B
	sdf          0B
	sdg          0B
	sr0       1024M
	[root@sysrescue /]# reboot
	[root@sysrescue /]#
		Stopping Session 1 of User root...
		Stopping Session 2 of User root...
	[  OK  ] Removed slice Slice /system/modprobe.
	[  OK  ] Stopped target Multi-User System.
	[...]
	[  OK  ] Reached target System Shutdown.
	[  OK  ] Reached target Late Shutdown Services.
	[  OK  ] Finished System Reboot.
	[  OK  ] Reached target System Reboot.
	[...]
	[  393.223783] shutdown[1]: Failed to finalize file systems, loop devices, ignoring.
	[  393.508673] reboot: Restarting system
	--------------------------------------------------------------------------------
	[    0.000000] Linux version 5.10.0-21-amd64 (debian-kernel@lists.debian.org) (gcc-10 (Debian 10.2.1-6) 10.2.1 20210110, GNU ld (GNU Binutils for Debian) 2.35.2) #1 SMP Debian 5.10.162-1 (2023-01-21)
	[    0.000000] Command line: BOOT_IMAGE=/boot/vmlinuz-5.10.0-21-amd64 root=UUID=xxxxxx-xxxxxx-xxxxxx-xxxx ro console=tty0 console=ttyS0,115200 earlyprintk=ttyS0,115200 consoleblank=0
	[    0.000000] x86/fpu: Supporting XSAVE feature 0x001: 'x87 floating point registers'
	[    0.000000] x86/fpu: Supporting XSAVE feature 0x002: 'SSE registers'
	[    0.000000] x86/fpu: Supporting XSAVE feature 0x004: 'AVX registers'
	[    0.000000] x86/fpu: xstate_offset[2]:  576, xstate_sizes[2]:  256
	[    0.000000] x86/fpu: Enabled xstate features 0x7, context size is 832 bytes, using 'standard' format.
	[    0.000000] BIOS-provided physical RAM map:
	[...]
	[    5.499258] usbcore: registered new interface driver uas
	[    5.503608] hub 8-0:1.0: 5 ports detected
	Begin: Loading essential drivers ... done.
	[...]
	Begin: Running /scripts/init-premount ... done.
	[...]
	Welcome to Debian GNU/Linux 11 (bullseye)!

	[    6.642082] systemd[1]: Set hostname to <debian>.
	[    6.648738] random: systemd: uninitialized urandom read (16 bytes read)
	[...]
	[  OK  ] Finished Load AppArmor profiles.
	[  OK  ] Started ifup for enp1s0.
	[  OK  ] Started ifup for enp3s0.
		Starting Raise network interfaces...
	[*     ] A start job is running for Raise network interfaces (5s / 5min 3s)
	[   12.698722] e1000e 0000:03:00.0 enp3s0: NIC Link is Up 1000 Mbps Full Duplex, Flow Control: Rx/Tx
	[   12.709838] IPv6: ADDRCONF(NETDEV_CHANGE): enp3s0: link becomes ready
	[   12.782696] e1000e 0000:01:00.0 enp1s0: NIC Link is Up 1000 Mbps Full Duplex, Flow Control: Rx/Tx
	[   12.794432] IPv6: ADDRCONF(NETDEV_CHANGE): enp1s0: link becomes ready
	[...]
	[  OK  ] Finished Raise network interfaces.
	[  OK  ] Reached target Network.
		Starting Initial cloud-ini&hellip; (metadata service crawler)...
	[   20.902510] cloud-init[727]: Cloud-init v. 20.4.1 running 'init' at Tue, 31 Jan 2023 14:49:11 +0000. Up 20.87 seconds.
	[   20.924705] cloud-init[727]: ci-info: +++++++++++++++++++++++++++++++++++++++Net device info+++++++++++++++++++++++++++++++++++++++
	[   20.951306] cloud-init[727]: ci-info: +--------+------+------------------------------+--------------+--------+-------------------+
	[   20.975323] cloud-init[727]: ci-info: | Device |  Up  |     Address                  |      Mask    | Scope  |   Hw-Address      |
	[   20.999314] cloud-init[727]: ci-info: +--------+------+------------------------------+--------------+--------+-------------------+
	[...]
	[   23.502767] cloud-init[808]: Cloud-init v. 20.4.1 running 'modules:config' at Tue, 31 Jan 2023 14:49:13 +0000. Up 22.35 seconds.
	[   23.528529] cloud-init[824]: Cloud-init v. 20.4.1 running 'modules:final' at Tue, 31 Jan 2023 14:49:14 +0000. Up 23.34 seconds.
	[   23.565264] cloud-init[824]: Get:1 http://security.debian.org/debian-security bullseye-security InRelease [48.4 kB]
	[   23.583425] cloud-init[824]: Get:2 http://deb.debian.org/debian bullseye InRelease [116 kB]
	[   23.626995] cloud-init[824]: Get:3 http://deb.debian.org/debian bullseye-updates InRelease [44.1 kB]
	[...]

	Debian GNU/Linux 11 dlcicl01 ttyS0

	dlcicl01 login:

	[   28.875554] cloud-init[824]: Fetched 25.0 MB in 5s (4629 kB/s)
	[   29.773719] cloud-init[824]: Reading package lists...
	[   29.858709] cloud-init[824]: Reading package lists...
	[   30.074568] cloud-init[824]: Building dependency tree...
	[   30.074802] cloud-init[824]: Reading state information...
	[   30.339086] cloud-init[824]: Calculating upgrade...
	[   30.629502] cloud-init[824]: The following packages will be upgraded:
	[   30.632259] cloud-init[824]:     bind9-host bind9-libs curl libcurl3-gnutls libcurl4
	[   30.747156] cloud-init[824]: 5 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
	[...]
	[   33.768063] cloud-init[824]: Cloud-init v. 20.4.1 finished at Tue, 31 Jan 2023 14:49:24 +0000. Datasource DataSourceNoCloud [seed=/dev/sda2][dsmode=net].  Up 33.76 seconds

	dlcicl01 login:
	```

	</details>

## Getting help, discussing, and/or modifying

TBD

-------

## Colophon

* [Copyright and licensing](LICENSE)
* [Inspirations, information, and source material](ACKNOWLEDGEMENTS.md)
* [Notes](README-NOTES.md)
