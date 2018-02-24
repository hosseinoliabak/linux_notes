# Linux notes:
* `ctrl+L` to clear the screen

* Commands in Linux retrieve information from virtual file systems.
Those virtual file system might include directories uch as `/proc`, `/sys`,
or `/dev`

### /sys
`/sys` directory stores and allows modification of the devices connected to the system
(mostly attached and installed hardware) in a structured way for example:
<pre>
hossein@hossein $ cat /sys<b>/module/scsi_mod/parameters/</b>max_luns
512
</pre>

Essentially `/proc` and `/sys` are the same.
`sysfs` was added in kernel 2.5 or 2.6 due to clutter in `procfs`.

The procfs was only meant to hold process information. eventually everything
started getting mixed into proc and it created a twisty maze with device data
stuck in different spots all over the place. Meanwhile, sysfs was implemented
with the objective of segmenting device data from procfs.

<pre>
hossein@hossein ~ $ <b>cat /proc/sys/kernel/version</b>
#42~16.04.1-Ubuntu SMP Tue Oct 10 16:32:20 UTC 2017
</pre>

### /proc
For example `/proc/devices` shows the drivers that are loaded
<pre>
hossein@hossein ~ $ <b>cat /proc/devices</b>
Character devices:
  1 mem
  4 /dev/vc/0
  4 tty
  4 ttyS
  5 /dev/tty
  5 /dev/console
  5 /dev/ptmx
  5 ttyprintk
  6 lp
  7 vcs
 10 misc
 13 input
 21 sg
 29 fb
 ...
</pre>

Or `/proc/cmdline` is representative of the command line arguments that will pass
through the Kernel when it was started.
<pre>
hossein@hossein ~ $ <b>cat /proc/cmdline</b>
BOOT_IMAGE=/boot/vmlinuz-4.10.0-38-generic root=UUID=6dc9ee42-ce33-4 ro quiet splash vt.handoff=7
</pre>

Or `/proc/modules` is representative of all of the Kernel modules have been loaded
<pre>
hossein@hossein ~ $ <b>cat /proc/modules</b>
xt_nat 16384 4 - Live 0x0000000000000000
veth 16384 0 - Live 0x0000000000000000
vxlan 53248 0 - Live 0x0000000000000000
ip6_udp_tunnel 16384 1 vxlan, Live 0x0000000000000000
udp_tunnel 16384 1 vxlan, Live 0x0000000000000000
xt_mark 16384 0 - Live 0x0000000000000000
nf_conntrack_netlink 36864 0 - Live 0x0000000000000000
nfnetlink 16384 2 nf_conntrack_netlink, Live 0x0000000000000000
xfrm_user 32768 1 - Live 0x0000000000000000
...
</pre>

* Hardware Device Drivers
These are typically Kernel modules that are loaded as required into the running Kernel.
They are managed with `lsmod` ans `modprobe` along with options that can be configured
in `/etc/modprobe.d/`

<pre>
hossein@hossein <b>/etc $ lsmod</b>
Module                  Size  Used by
xt_nat                 16384  4
veth                   16384  0
vxlan                  53248  0
ip6_udp_tunnel         16384  1 vxlan
udp_tunnel             16384  1 vxlan
xt_mark                16384  0
nf_conntrack_netlink    36864  0
nfnetlink              16384  2 nf_conntrack_netlink
xfrm_user              32768  1
xfrm_algo              16384  1 xfrm_user
xt_addrtype            16384  2
br_netfilter           24576  0
</pre>

* If we want to unload a driver and any dependency modules, we use `-r` option in `modprobe` command.
* You can add `-v` to see what exactly is happening. (Se the dependencies removed!) `modprobe -rv psmouse`
* To load a module `modprobe -v`
* Demo is in below example:
<pre>
hossein@hossein ~ $ <b>sudo modprobe -rv psmouse</b>
rmmod psmouse
hossein@hossein ~ $ <b>modinfo psmouse</b>
filename:       /lib/modules/4.10.0-38-generic/kernel/drivers/input/mouse/psmouse.ko
license:        GPL
description:    PS/2 mouse driver
...
vermagic:       4.10.0-38-generic SMP mod_unload
parm:           proto:Highest protocol extension to probe (bare, imps, exps, any). Useful for KVM switches. (proto_abbrev)
parm:           resolution:Resolution, in dpi. (uint)
parm:           rate:Report rate, in reports per second. (uint)
parm:           smartscroll:Logitech Smartscroll autorepeat, 1 = enabled (default), 0 = disabled. (bool)
parm:           resetafter:Reset device after so many bad packets (0 = never). (uint)
parm:           resync_time:How long can mouse stay idle before forcing resync (in seconds, 0 = never). (uint)
hossein@hossein ~ $ <b>sudo modprobe -v psmouse</b>
insmod /lib/modules/4.10.0-38-generic/kernel/drivers/input/mouse/psmouse.ko
</pre>

<pre>
hossein@hossein ~ $ <b>ls /sys/module/psmouse/parameters/</b>
proto  rate  resetafter  resolution  resync_time  smartscroll</b>
hossein@hossein ~ $ <b>cat /sys/module/psmouse/parameters/proto</b>
auto
</pre>
* To disable a hardware in `/etc/modprobe.d` we can use command `blacklist MODULE_NAME`

### /dev
`/dev` represents the device files in the system.
The device files are created during installation, and later with the /dev/MAKEDEV script.

<pre>
hossein@hossein / $ <b>ls /dev/sd*</b>
/dev/sda  /dev/sda1  /dev/sda2  /dev/sda3
</pre>

List the block devices:
<pre>
hossein@hossein ~ $ <b>lsblk</b>
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 111.8G  0 disk
├─sda2   8:2    0  95.4G  0 part /
├─sda3   8:3    0  15.9G  0 part [SWAP]
└─sda1   8:1    0   512M  0 part /boot/efi
</pre>
* MAJ: Kernel driver that is being used to access to this drive

<pre>
hossein@hossein ~ $ <b>df -hT</b>
<b>Filesystem</b>     <b>Type</b>      Size  Used Avail Use% Mounted on
udev           devtmpfs  7.8G     0  7.8G   0% /dev
tmpfs          tmpfs     1.6G  9.7M  1.6G   1% /run
/dev/sda2      ext4       94G   46G   44G  51% /
tmpfs          tmpfs     7.8G   27M  7.8G   1% /dev/shm
tmpfs          tmpfs     5.0M  4.0K  5.0M   1% /run/lock
tmpfs          tmpfs     7.8G     0  7.8G   0% /sys/fs/cgroup
/dev/sda1      vfat      511M  3.5M  508M   1% /boot/efi
cgmfs          tmpfs     100K     0  100K   0% /run/cgmanager/fs
tmpfs          tmpfs     1.6G   40K  1.6G   1% /run/user/1000
</pre>

**D-Bus** (Desktop-BUS) and **udev** are both services to detect hardware.
`dmesg` is reading itself from `/dev/kmsg` which contains Kernel messages

To clear the ring buffer:
<pre>
hossein@hossein ~ $ <b>sudo dmesg -C</b></pre>
Then when we run the `dmesg` command at this time, we won't see any output.

But if you use lower-case `c`, it prints the messages then clears all of them.

### Kernel Ring Buffer Messages
Messages can be read from the special device `/dev/kmsg` using the
command `dmesg` or from the associated log file.

<pre>hossein@hossein ~ $ <b>dmesg -w</b></pre>
to see the detection process when we connect a device for example a flash USB to the machine



### Boot Sequence
1. BIOS / UEFI: BIOS locates the bootable disk, from the bootable disk we are going to load bootloader
2. Bootloader: allows us to select the Kernel to boot to our OS. The bootloader then loads the Kernel.
3. Kernel: Going to be starting its system and service manager `systemd` PID = 1

**Bootloader Locations**

GRUB legacy: CentOS 6, SUSE 11, In MBR, first sector of partition
  * `/boot/grub/menu.lst` or `/boot/grub/grub.conf`
  * `grub-install /dev/fd0` to install grub on /dev/fd0

GRUB 2: CentOS 7, SLES 23, Debian 8, Ubuntu 9.10: MBR, GPT
* The location (GPT/MBR) is asked during the installation
* Backing-up and restoring the MBR
  * `dd if=/dev/sda of=/root/sda.mbr count=1 bs=512`
* To restore:
  * `dd if=/root/sda.mbr of=/dev/sda`
* In Debian: `/boot/grub/grub.cfg`, In Red Hat: `/boot/grub2/grub.cfg`
  * Edits are NOT made directly to `grub.cfg` but in `/etc/default/grub`
    <pre>
    GRUB_TIMEOUT=5
    GRUB_DEFAULT=saved
    GRUB_DISABLE_SUBMENU=true
    GRUB_TERMINAL_OUTPUT="console"
    GRUB_CMDLINE_LINUX="rd.lvm.lv=centos/root rd.lvm.lv=centos/swap crashkernel=auto rhgb quiet"
    GRUB_DISABLE_RECOVERY="true"
    </pre>
  * `grub2-mkconfig -o /boot/grub2/grub.cfg` to create grub.cfg for Debian `grub2-mkconfig`
    <pre>
    [vagrant@web ~]$ <b>sudo grub2-mkconfig -o /boot/grub2/grub.cfg</b>
    Generating grub configuration file ...
    Found linux image: /boot/vmlinuz-3.10.0-229.el7.x86_64
    Found initrd image: /boot/initramfs-3.10.0-229.el7.x86_64.img
    Found linux image: /boot/vmlinuz-0-rescue-02ab91023f244febb8c38be42e37cc2d
    Found initrd image: /boot/initramfs-0-rescue-02ab91023f244febb8c38be42e37cc2d.img
    done
    </pre>
* `grub2-install /dev/sda` to install grub on */dev/sda*

### Using and reading Kernel Options
Check the boot image location
<pre>
hossein@hossein ~ $ <b>cat /proc/cmdline</b>
BOOT_IMAGE=<b>/boot/vmlinuz-4.10.0-38-generic</b> root=UUID=6dc9ee42-ce33-4 <b>ro quiet</b> splash vt.handoff=7
</pre>

When booting, press `e`:

In Grub 2, with Debian-based Linux we see the Kernel as term `linux`.
In RedHat we see the term `linux16` for the Kernel line.

Then we can add at the end of line after `ro quiet` add `systemd.unit=rescue.target` for maintenance.
Then press `Ctrl+x`

### Password recovery

There are some methods, but here we review *Booting into single user mode*
1. Restart the machine and press "Shift" key continuously 
2. Boot into GRUB Stage 
3. Modify Kernel Argument (by pressing ‘a’ to modify the kernel arguments) 
4. Append 1 at the end of **rhgb quiet** and press `Enter` key to boot into single user mode 
5. Type `passwd` Command 

### systemctl
Is being used to many Linux systems.

All processes started by a service unit are tagged with the same **cgroup**. In this way when the
service shuts down, it is much easier to ensure all reloaded processes are also shutdown.

Service units can be activated on other events, such as hardware detection and not just on entering a *runlevel*.

Resource management is controlled and limited through the *cgroup* created for each service.

Service units are started in parallel. Much quicker bootup. RedHat 7 boots up much more quickly than 6
<pre>
hossein@hossein ~ $ <b>top -p 1</b>
top - 22:52:36 up  2:52,  1 user,  load average: 0.29, 0.14, 0.11
Tasks:   1 total,   0 running,   1 sleeping,   0 stopped,   0 zombie
%Cpu(s):  2.1 us,  2.3 sy,  0.0 ni, 95.5 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 16351780 total, 11428344 free,  2634256 used,  2289180 buff/cache
KiB Swap: 16698364 total, 16698364 free,        0 used. 13211276 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
    1 root      20   0  185748   6392   4000 S   0.0  0.0   0:01.59 <b>systemd</b>
</pre>

Argument: This system took hours to boot up!!! You best friend would be:

`systemd-analyze`: Analyzes system boot-up performance.

`systemd-analyze blame`: Prints a list of all running units, ordered by the time they took to initialize.

<pre>
hossein@hossein ~ $ <b>systemd-analyze</b>
Startup finished in 5.004s (firmware) + 3.821s (loader) + 3.298s (kernel) + 13.801s (userspace) = 25.927s
</pre>

<pre>
hossein@hossein ~ $ <b>systemd-analyze blame</b>
          6.925s NetworkManager-wait-online.service
          4.234s docker.service
          1.234s vmware.service
           877ms systemd-tmpfiles-setup.service
           675ms dev-sda2.device
           599ms apparmor.service
           497ms lvm2-monitor.service
           375ms accounts-daemon.service
           371ms ModemManager.service
...
</pre>

Above, We can see `NetworkManager-wait-online.service` took the longest time to startup.

<pre>
hossein@hossein ~ $ cat <b>/lib/systemd/system/</b>networking.service
<b>[Unit]</b>
Description=Raise network interfaces
Documentation=man:interfaces(5)
DefaultDependencies=no
Wants=network.target
After=local-fs.target network-pre.target apparmor.service systemd-sysctl.service systemd-modules-load.service
Before=network.target shutdown.target network-online.target
Conflicts=shutdown.target

<b>[Install]</b>
WantedBy=multi-user.target
WantedBy=network-online.target

<b>[Service]</b>
Type=oneshot
EnvironmentFile=-/etc/default/networking
ExecStartPre=-/bin/sh -c '[ "$CONFIGURE_INTERFACES" != "no" ] && [ -n "$(ifquery --read-environment --list --exclude=lo)" ] && udevadm settle'
ExecStart=/sbin/ifup -a --read-environment
ExecStop=/sbin/ifdown -a --read-environment --exclude=lo
RemainAfterExit=true
TimeoutStartSec=5min
</pre>

To see the services on the system:
<pre>
[vagrant@web ~]$ <b>systemctl list-unit-files --type=service | grep enabled</b>
auditd.service                              enabled
autovt@.service                             enabled
crond.service                               enabled
cups.service                                enabled
dbus-org.freedesktop.NetworkManager.service enabled
dbus-org.freedesktop.nm-dispatcher.service  enabled
getty@.service                              enabled
irqbalance.service                          enabled
kdump.service                               enabled
lvm2-monitor.service                        enabled
microcode.service                           enabled
NetworkManager-dispatcher.service           enabled
NetworkManager.service                      enabled
postfix.service                             enabled
rsyslog.service                             enabled
sshd.service                                enabled
systemd-readahead-collect.service           enabled
systemd-readahead-drop.service              enabled
systemd-readahead-replay.service            enabled
tuned.service                               enabled
</pre>

To list all services
<pre>[root@client1 ~]# <b>systemctl -at service</b></pre>

To list the running services
<pre>[root@client1 ~]# <b>systemctl -t service --state=active</b></pre>

### xinetd
Trivial services may be managed by the super daemon xinetd or inetd. The global
configuration `/etc/xinetd.conf` and services in `/etc/xinetd.d`

### Change Runlevels or Boot Targets and Shutdown or Reboot the system
* **Runlevels** are part of SysVinit and Upstart
  * Reserved:
    * S
    * 1
    * 0 halt
    * 6 Reboot
  * Non-reserved
    * 2 Ubuntu/Debian
    * 3, 5 Red Hat, SUSE
* Default runlevel is stored in `/etc/inittab`
  *  `inittab` is no longer used when using systemd
* `who -r` or `runlevel` to show the current runlevel
* `telinit` to change the runlevel

* **Target** is in systemd
  * `poweroff.target` (equivalent of runlevel 0)
  * `rescue.target`: 1
  * `multiuser.target`: 2, 3, 4
  * `graphical.target`: 5
  * `reboot.target`: 6
<pre>
[vagrant@web etc]$ <b>systemctl get-default</b>
multi-user.target
</pre>

<pre>
hossein@hossein ~ $ <b>systemctl get-default</b>
graphical.target
hossein@hossein ~ $ <b>sudo systemctl set-default multi-user.target</b>
[sudo] password for hossein:
Created symlink from /etc/systemd/system/default.target to /lib/systemd/system/multi-user.target.
hossein@hossein ~ $ <b>systemctl get-default</b>
multi-user.target
</pre>

<pre>
hossein@hossein ~ $ <b>systemctl status networking.service</b>
● networking.service - Raise network interfaces
   Loaded: loaded (/lib/systemd/system/networking.service; enabled; vendor preset: enabled)
  Drop-In: /run/systemd/generator/networking.service.d
           └─50-insserv.conf-$network.conf
   Active: active (exited) since Thu 2017-12-21 09:34:58 EST; 10h ago
     Docs: man:interfaces(5)
 Main PID: 1070 (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/networking.service

Dec 21 09:34:58 hossein systemd[1]: Starting Raise network interfaces...
Dec 21 09:34:58 hossein systemd[1]: Started Raise network interfaces.
</pre>

Shutdown or Reboot the system
* `shutdown`
  * `shutdown -c` to cancel the scheduled shutdown
* `reboot`
* `poweroff`

<pre>
echo "My message 10" | wall
</pre>

Let's look at some systemd directories. Here we can see the list of all services
<pre>
[vagrant@web ~]$ <b>ls /lib/systemd/system</b>
arp-ethers.service                      machines.target                                -.slice
auditd.service                          messagebus.service                             slices.target
auth-rpcgss-module.service              multi-user.target                              smartcard.target
autovt@.service                         multi-user.target.wants                        sockets.target
basic.target                            NetworkManager-dispatcher.service              sockets.target.wants
basic.target.wants                      NetworkManager.service                         sound.target
...
</pre>

Here we can see the symbolic links
<pre>
[vagrant@web ~]$ <b>ls /etc/systemd/system</b>
dbus-org.freedesktop.NetworkManager.service  dev-virtio\x2dports-org.qemu.guest_agent.0.device.wants  sockets.target.wants
dbus-org.freedesktop.nm-dispatcher.service   getty.target.wants                                       sysinit.target.wants
default.target                               multi-user.target.wants                                  system-update.target.wants
default.target.wants                         remote-fs.target.wants
</pre>


## Design hard disk layout

![mbr vs guid](https://user-images.githubusercontent.com/31813625/34284930-88aabdb4-e6a3-11e7-9e63-82bacd65eca6.png)

### MBR vs GUID
* The partition table of an **MBR** disk is stored in the first sector or 512 bytes of the disk.
If you use a BIOS-based system, it has to boot to the MBR.
* **GUID Partition Table (GPT):** Globally Unique Identifier Partition Table
  * Replacement of MBR partitioning
  * Only supported by UEFI not BIOS
* The tools that we can use to create partitions:
  * fdisk: only on MBR: `fdisk /dev/sdc`
    * nothing done unless you press `w` on prompt
  * parted: guid or mbr
  * gdisk: to work with guid partition table
    <pre>
    hossein@hossein ~ $ <b>sudo gdisk -l /dev/sda</b>
    [sudo] password for hossein:
    GPT fdisk (gdisk) version 1.0.1

    Partition table scan:
      MBR: protective
      BSD: not present
      APM: not present
      GPT: present

    Found valid GPT with protective MBR; using GPT.
    Disk /dev/sda: 234441648 sectors, 111.8 GiB
    Logical sector size: 512 bytes
    Disk identifier (GUID): 80CFBA94-37A9-4D8C-83C1-F2BFB973B000
    Partition table holds up to 128 entries
    First usable sector is 34, last usable sector is 234441614
    Partitions will be aligned on 2048-sector boundaries
    Total free space is 2925 sectors (1.4 MiB)

    Number  Start (sector)    End (sector)  Size       Code  Name
       1            2048         1050623   512.0 MiB   EF00  EFI System Partition
       2         1050624       201043967   95.4 GiB    8300
       3       201043968       234440703   15.9 GiB    8200
    </pre>

* LAB: I have added a 3.0G disk into my CentOS Linux. It is shown as `sdb`

    <pre>
    [vagrant@web ~]$ <b>lsblk</b>
    NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    sda               8:0    0   40G  0 disk
    ├─sda1            8:1    0  500M  0 part /boot
    └─sda2            8:2    0 39.5G  0 part
      ├─centos-root 253:0    0 38.5G  0 lvm  /
      └─centos-swap 253:1    0    1G  0 lvm  [SWAP]
    <b>sdb               8:16   0    3G  0 disk</b>
    </pre>

* Lets' create the partition:

    <pre>
    [vagrant@web ~]$ <b>sudo gdisk /dev/sdb</b>
    GPT fdisk (gdisk) version 0.8.6

    Partition table scan:
      MBR: not present
      BSD: not present
      APM: not present
      GPT: not present

    Creating new GPT entries.

    Command (? for help): <b>?</b>
    b	back up GPT data to a file
    c	change a partition's name
    d	delete a partition
    i	show detailed information on a partition
    l	list known partition types
    <b>n	add a new partition</b>
    o	create a new empty GUID partition table (GPT)
    p	print the partition table
    q	quit without saving changes
    r	recovery and transformation options (experts only)
    s	sort partitions
    t	change a partition's type code
    v	verify disk
    w	write table to disk and exit
    x	extra functionality (experts only)
    ?	print this menu
    </pre>

    <pre>
    [vagrant@web ~]$ <b>lsblk</b>
    NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    sda               8:0    0   40G  0 disk
    ├─sda1            8:1    0  500M  0 part /boot
    └─sda2            8:2    0 39.5G  0 part
      ├─centos-root 253:0    0 38.5G  0 lvm  /
      └─centos-swap 253:1    0    1G  0 lvm  [SWAP]
    <b>sdb               8:16   0    3G  0 disk
    └─sdb1            8:17   0    3G  0 part</b>
    </pre>

* Now, we have created the partition and it is time to format the partition:

<pre>
[vagrant@web ~]$ <b>sudo mkfs.btrfs /dev/sdb1</b>
Btrfs v3.16.2
See http://btrfs.wiki.kernel.org for more information.

Turning ON incompat feature 'extref': increased hardlink limit per file to 65536
fs created label (null) on /dev/sdb1
	nodesize 16384 leafsize 16384 sectorsize 4096 size 3.00GiB
</pre>

* Now it is time to mount the drive:
<pre>
[vagrant@web ~]$ <b>sudo blkid /dev/sdb1</b>
/dev/sdb1: <b>UUID="347f99df-5cbf-4f22-a00d-e458e0598d05"</b> UUID_SUB="3f3dab7d-1f8e-40ea-8a24-e2ca68a8158f" TYPE="btrfs" PARTLABEL="Linux filesystem" PARTUUID="02e92ce8-40bd-42cd-a080-9a4552f4bb3d"
[vagrant@web ~]$ <b>vi /etc/fstab</b>
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=57ba6944-e05b-4f02-95d4-ea9505fe584f /boot                   xfs     defaults        0 0
/dev/mapper/centos-swap swap                    swap    defaults        0 0
<b>UUID="347f99df-5cbf-4f22-a00d-e458e0598d05"     /test   btrfs   defaults        0 0</b>
[vagrant@web ~]$ <b>sudo mkdir /test</b>
[vagrant@web ~]$ <b>sudo mount -a</b>
[vagrant@web ~]$ <b>mount | grep test</b>
/dev/sdb1 on /test type btrfs (rw,relatime,seclabel,space_cache)
[vagrant@web test]$ <b>df -hT /dev/sdb1</b>
Filesystem     Type   Size  Used Avail Use% Mounted on
/dev/sdb1      btrfs  <b>3.0G</b>  320K  2.7G   1% <b>/test</b></pre>

* The file system types are:
  * ext2: 1993
  * ext3: 2001
  * ext4: 2008
  * XFS: Default in enterprise systems such as RHEL, CentOS:
    * Enterprise tools such as `xfsdump`, `xfsrestore`, ...
  * Creating EXTended file systems
    * `mkfs -t ext4 /dev/sdc`
    * `mkfs.ext4 /dev/sdc`
    * `mke2s -t ext4 /dev/sdc`
    * The command `mkfs` will run `mke2fs` when creating *ext2*, *ext3*, or *ext4* file systems

#### FHS
Filesystem Hierarchy Standard (FHS) is a document describing the Linux / Unix file hierarchy.
It is very useful to know these because it lets you easily find what you are looking for:
* `ls -l /`
* No drive letters, just mount points
  * /
  * /var: print spool files, web server content, database files, log files.
  * /boot: kernel files, boot loader files. Small read-only partition
  * SWAP: works like an extended memory.
  * /home: also called a login directory. the repository for a user's
  personal files, directories and programs.
  * /bin: Common programs, shared by the system, the system administrator and the users.
  * /sbin: Programs for use by the system and the system administrator.
  * /etc: Most important system configuration files are in /etc,
  this directory contains data similar to those in the Control Panel in Windows
  * /opt: Typically contains extra and third party software.
  * /usr filesystem is the second major section of the filesystem, containing shareable, read-only data. It can be shared between systems, although present practice does not often do this
* Logical Volume Manager: Operating system level partitioning system.
  * Physical volumes: or the disks themselves (JBOD)
  * Volume groups: combine the physical volumes together
  * Logical volumes: we format and mount into the desired mount point

### Extending a logical volume
* Mark the extra partition as physical volume: `pvcreate /dev/sdc1`
* Add the pv (physical volume) to the existing volume group (vg): `vgextend vg1 /dev/sdc1`
* Extend the space available to the logical volume (lv): `lvextend -L +1000M /dev/vg1/data`
* Resize the file system: `resize2fs /dev/vg1/data`
  * `resize2fs` if the filesystem type is **ext4** or `xfs_growfs` command if the filesystem uses **xfs** or

Before adding a physical disk:
<pre>
[root@localhost ~]# <b>lsblk</b>
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   12G  0 disk
├─sda1            8:1    0  500M  0 part /boot
└─sda2            8:2    0 11.5G  0 part
  ├─centos-root 253:0    0 10.7G  0 lvm  /
  └─centos-swap 253:1    0  820M  0 lvm  [SWAP]

[root@localhost ~]# <b>df -hT</b>
Filesystem              Type      Size  Used Avail Use% Mounted on
/dev/mapper/centos-root xfs        <b><u>11G</u></b>  7.8G  3.0G  73% /
devtmpfs                devtmpfs  986M     0  986M   0% /dev
tmpfs                   tmpfs    1001M  152K 1001M   1% /dev/shm
tmpfs                   tmpfs    1001M  8.7M  992M   1% /run
tmpfs                   tmpfs    1001M     0 1001M   0% /sys/fs/cgroup
/dev/sda1               xfs       497M  140M  358M  29% /boot
tmpfs                   tmpfs     201M   16K  201M   1% /run/user/1000
</pre>

Then I added a 2GiB physical disk

<pre>
[root@localhost ~]# <b>lsblk</b>
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   12G  0 disk
├─sda1            8:1    0  500M  0 part /boot
└─sda2            8:2    0 11.5G  0 part
  ├─centos-root 253:0    0 10.7G  0 lvm  /
  └─centos-swap 253:1    0  820M  0 lvm  [SWAP]
<b>sdb               8:16   0    2G  0 disk</b>

[root@localhost ~]# <b>pvcreate /dev/sdb</b>
  Physical volume "/dev/sdb" successfully created

[root@localhost ~]# <b>vgdisplay</b>
  --- Volume group ---
  <b>VG Name               centos</b>
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  6
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               11.51 GiB
  PE Size               4.00 MiB
...

[root@localhost ~]# <b>vgextend centos /dev/sdb</b>
  Volume group "centos" successfully extended

[root@localhost ~]# <b>lvextend -l +100%FREE /dev/centos/root</b>
  Size of logical volume centos/root changed from 10.71 GiB (2741 extents) to 12.70 GiB (3251 extents).
  Logical volume root successfully resized.

[root@localhost ~]# <b>xfs_growfs /dev/centos/root</b>
meta-data=/dev/mapper/centos-root isize=256    agcount=7, agsize=436992 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=0        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=2806784, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=0
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
<b>data blocks changed from 2806784 to 3329024</b>

[root@localhost ~]# <b>reboot</b>

[root@localhost ~]# <b>lsblk</b>
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   12G  0 disk
├─sda1            8:1    0  500M  0 part /boot
└─sda2            8:2    0 11.5G  0 part
  ├─centos-root 253:0    0 12.7G  0 lvm  /
  └─centos-swap 253:1    0  820M  0 lvm  [SWAP]
sdb               8:16   0    2G  0 disk
└─centos-root   253:0    0 12.7G  0 lvm  /

[root@localhost ~]# <b>df -HT</b>
Filesystem              Type      Size  Used Avail Use% Mounted on
/dev/mapper/centos-root xfs        <b><u>13G</u></b>  7.8G  4.9G  62% /
devtmpfs                devtmpfs  986M     0  986M   0% /dev
tmpfs                   tmpfs    1001M  152K 1001M   1% /dev/shm
tmpfs                   tmpfs    1001M  8.7M  992M   1% /run
tmpfs                   tmpfs    1001M     0 1001M   0% /sys/fs/cgroup
/dev/sda1               xfs       497M  140M  358M  29% /boot
tmpfs                   tmpfs     201M   12K  201M   1% /run/user/1000
</pre>

### Retrieving Current swap Information
`swapon -s`
<pre>
hossein@hossein ~ $ <b>swapon -s</b>
Filename				Type		Size	Used	Priority
/dev/sda3                              	partition	16698364	0	-1
</pre>
`free -m`
<pre>
hossein@hossein ~ $ <b>free -m</b>
              total        used        free      shared  buff/cache   available
Mem:          15968        5410        3478          88        7079        9999
Swap:         16306           0       16306
</pre>
`cat /proc/meminfo`
<pre>
hossein@hossein ~ $ <b>cat /proc/meminfo</b>
MemTotal:       16351780 kB
MemFree:         3561920 kB
MemAvailable:   10239656 kB
Buffers:          232016 kB
Cached:          6560472 kB
...
</pre>
* To create swap: `mkswap /dev/sdc3`
* Then we use `swapon` command to mount the partition

### Maintain the Integrity of File Systems
* The main tool for repairing file systems is `fsck`.
It is used to check and repair file systems inconsistencies. When
referencing more elaborate file systems such as XFS and BTRFS, the command
will tell you to use their native tools.
Underlying `fsck` calls other programs:
  * for EXT file systems: `e2fsck`
  * for XFC file systems: `xfs_repair`
  * for btrfs file systems: `btrfsck`
* Examples:
  * `fsck /dev/sda1` Check just the named partition
  * `fsck -A` check all file syste with fsckpass > 0 in /etc/fstab
  * `fsck -AR` as before excluding root
  * `fsck -ARM` as before excluding mounted file systems
  * `fsck -f /dev/sda1` Force a check even if marked as clean
* `debugfs`: An interactive tool for debug an EXT filesystem. It opens
the filesystem in read-only mode unless we tell it not to (with `-w` option).
It can undelete files and directories
<pre>
hossein@hossein ~ $ <b>sudo debugfs /dev/sda2</b>
debugfs 1.42.13 (17-May-2015)
debugfs:  <b>cd /etc</b/>
debugfs:  <b>pwd</b>
[pwd]   INODE: 3407873  PATH: /etc
[root]  INODE:      2  PATH: /
debugfs: <b>stat passwd</b> #shows data on one file
<b>Inode: 3430077</b>   Type: regular    Mode:  0644   Flags: 0x80000
Generation: 3542921786    Version: 0x00000000:00000001
User:     0   Group:     0   Size: 2645
File ACL: 0    Directory ACL: 0
Links: 1   Blockcount: 8
Fragment:  Address: 0    Number: 0    Size: 0
debugfs:  <b>ncheck 3430077</b>#node check an inode
Inode	Pathname
3430077	/etc/passwd
debugfs:  <b>q</b>#quit
</pre>

* `/etc/mke2fs.conf`: The defaults for creating file systems come from this file.
Consideration of the `inode_size` is worthwhile.
  * Running out of inode entries, (the number of files), can cause disk full messages
  although there may be disk space left. The `inode_ratio` (bytes/inode) controls the
  amount of available inodes
<pre>
[vagrant@web ~]$ <b>cat /etc/mke2fs.conf</b>
[defaults]
	base_features = sparse_super,filetype,resize_inode,dir_index,ext_attr
	default_mntopts = acl,user_xattr
	enable_periodic_fsck = 0
	blocksize = 4096
	<b>inode_size = 256</b> # 256 bytes
	inode_ratio = 16384

[fs_types]
	ext3 = {
		features = has_journal
	}
	ext4 = {
		features = has_journal,extent,huge_file,flex_bg,uninit_bg,dir_nlink,extra_isize,64bit
		inode_size = 256
	}
	ext4dev = {
		features = has_journal,extent,huge_file,flex_bg,uninit_bg,dir_nlink,extra_isize
		inode_size = 256
		options = test_fs=1
	}
	small = {
		blocksize = 1024
		inode_size = 128
		inode_ratio = 4096
	}
	floppy = {
		blocksize = 1024
		inode_size = 128
		inode_ratio = 8192
	}
	big = {
		inode_ratio = 32768
	}
	huge = {
		inode_ratio = 65536
	}
	news = {
		inode_ratio = 4096
	}
	largefile = {
		inode_ratio = 1048576
		blocksize = -1
	}
	largefile4 = {
		inode_ratio = 4194304
		blocksize = -1
	}
	hurd = {
	     blocksize = 4096
	     inode_size = 128
	}
</pre>

### df vs du
Available disk space can be shown using `df -H` command. The `du` command can be used to show how
much space a file or directory takes on the disk. Use `-s` to only display the total for directory
<pre>
[vagrant@web ~]$ <b>df -HT</b>
Filesystem              Type      Size  Used Avail Use% Mounted on
/dev/mapper/centos-root xfs        42G  913M   41G   3% /
devtmpfs                devtmpfs  231M     0  231M   0% /dev
tmpfs                   tmpfs     240M     0  240M   0% /dev/shm
tmpfs                   tmpfs     240M  4.5M  236M   2% /run
tmpfs                   tmpfs     240M     0  240M   0% /sys/fs/cgroup
/dev/sda1               xfs       521M  125M  397M  24% /boot
none                    vboxsf    101G   54G   47G  54% /vagrant
</pre>

<pre>
[vagrant@web ~]$ <b>du -sh /home/</b>
101M	/home/
</pre>

To create multiple directories:
<pre>
[vagrant@web ~]$ <b>for i in `seq 1 1 4`; do mkdir d$i; done</b>
[vagrant@web ~]$ ls
<b>d1  d2  d3  d4</b></pre>

### Control mounting and un-mounting of file systems
* Manually
  * `Mount`: `mount -t ext4 -o rw,noatime,usrquota /dev/sdb1 /mnt`
  * `umount`: `umount data`
* Automatically mount on boot
  * `/etc/fstab` by using UUID
    <pre>
    [vagrant@web ~]$ <b>cat /etc/fstab</b>
    # /etc/fstab
    # Created by anaconda on Tue Oct  6 21:15:29 2015
    # Accessible filesystems, by reference, are maintained under '/dev/disk'
    # See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
    /dev/mapper/centos-root /                       xfs     defaults        0 0
    UUID=57ba6944-e05b-4f02-95d4-ea9505fe584f /boot                   xfs     defaults        0 0
    /dev/mapper/centos-swap swap                    swap    defaults        0 0
    </pre>

## Create, Monitor, and Kill Processes
* Long running processes:
  * `&` if the process is long, ideally we start it in the background using `&`.
  * `Ctrl+z` to stop the process. But it keeps it running in memory. But just processing it anymore.
  * `bg` command to restart the stopped process in the background.
<pre>
[vagrant@web ~]$ <b>sleep 40&</b>
[1] 3775
[vagrant@web ~]$ <b>sleep 20</b>
<b>^Z</b>
[2]+  Stopped                 sleep 20
[vagrant@web ~]$ <b>jobs</b>
[1]-  Running                 sleep 40 &
[2]+  Stopped                 sleep 20
[vagrant@web ~]$ <b>fg 2</b>
sleep 20
[vagrant@web ~]$ <b>jobs</b>
[1]+  <b>Running</b>                 sleep 40 &
[vagrant@web ~]$ <b>jobs</b>
[1]+  <b>Done</b>                    sleep 40
</pre>

Alternatively you can use `screen` command to run a command in background.
* `screen sleep 100`
* `Ctrl+a+d` to detach from screen
<pre>
[vagrant@web ~]$ screen sleep 20
[detached from 3822.pts-0.web]
[vagrant@web ~]$
</pre>

<pre>
hossein@hossein ~ $ <b>screen gns3</b>
[detached from 24089.pts-1.hossein] # I pressed <b>Ctrl+a+d</b> to detach from screen
[1]+  Done                    gns3
hossein@hossein ~ $ <b>screen -ls</b>
There is a screen on:
	24089.pts-1.hossein	(2017-12-23 07:03:55 PM)	(Detached)
1 Socket in /var/run/screen/S-hossein.
hossein@hossein ~ $ <b>screen -r 24089.pts-1.hossein</b> #To reattach the screen
</pre>

If you have an issue you may allow another user to attach to your screen so you
can walk them through the problem.
* `# chmod u+s $(which screen)` # To share screens set SUID
* `# chmod 755 /var/run/screen` # Ensure the correct permissions
* `$ screen -S demo` # To start the screen with an easy name
* `Ctrl+a :multiuser on` # Configure settings
* `Ctrl+a :acladd ali` # Configure ACL and add user ali
* `$ screen -x hossein/demo` # User ali connects to the hossein's screen
* to stop the screen sharing, hossein types `exit` command

**nohup**

The nohup command lets you run your commands even after you logged out and writes its output to `nohup.out`:
<pre>
[vagrant@web ~]$ <b>nohup ping 8.8.8.8&</b>
nohup: ignoring input and appending output to ‘nohup.out’
[vagrant@web ~]$ <b>cat nohup.out</b>
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=63 time=30.0 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=63 time=29.4 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=63 time=29.8 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=63 time=29.6 ms
64 bytes from 8.8.8.8: icmp_seq=5 ttl=63 time=29.5 ms
64 bytes from 8.8.8.8: icmp_seq=6 ttl=63 time=29.6 ms

--- 8.8.8.8 ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 5027ms
rtt min/avg/max/mdev = 29.435/29.688/30.019/0.275 ms
[vagrant@web ~]$ <b>ps</b>
  PID TTY          TIME CMD
 3284 pts/0    00:00:00 bash
 <b>3846 pts/0    00:00:00 ping</b>
 3852 pts/0    00:00:00 ps
</pre>

**ps**
* The `ps` command shows running processes on your computer.
Without no options shows the processes within the same shell.
We can see the calc with process id 28610 is running but is not listed in `ps`
output command. Because it is running unther another shell.
<pre>
hossein@hossein ~ $ <b>ps</b>
  PID TTY          TIME CMD
28635 pts/2    00:00:00 bash
28662 pts/2    00:00:00 ps
hossein@hossein ~ $ <b>pgrep calc</b>
28610
</pre>

<pre>
[vagrant@web ~]$ <b>ps -elf | wc -l</b>
86
[vagrant@web ~]$ <b>ps -aux | wc -l</b>
86
</pre>

<pre>
[vagrant@web ~]$ <b>pgrep sleep</b>
3909
3911
3913
[vagrant@web ~]$ <b>pkill sleep</b>
[vagrant@web ~]$ <b>pgrep sleep</b>
[vagrant@web ~]$
</pre>

**kill**

We could kill processes individually by command `kill`:
* `kill 3911` is the same as `kill -15 3911` or `kill -sigterm 3911`
* `kill -9 3911` is the same as `kill -kill 3911` or `kill -sigkill 3911`
* `kill -l` to see all the possible signals
* `killall sleep`

**top**

Processes are changing and sometimes you need to check them live. `top` command will help you 
* H help 
* Q quit 
* M sort based on memory usage" 
* c show full commands 
* K kill after asking pid and signal 
* load average must be less than the total number of cores of your CPU, let’s say at max 8 (for 8 cores) 

**free**

Giv info about the system memory
<pre>
[vagrant@web ~]$ <b>free -h</b>
              total        used        free      shared  buff/cache   available
Mem:           457M         64M        237M        4.6M        155M        303M
Swap:          1.0G          0B        1.0G
</pre>


## Debian package management
* `dpkg` works with `.deb` files. Can be used to install, upgrade, or remove `.deb` files (like `msi` files in Windows);
but does not resolve dependencies or locate the package.
  * `dpkg -i package.deb` to install the software
  * `dpkg -r package` to remove the software but leaves the configuration files.
  * `dpkg -P package` to purge it, (completely remove)
  * `dpkg --get-selections` to list what's installed
    <pre>
    vagrant@ac:~$ <b>dpkg --get-selections</b>
    acl					    	install
    acpi						install
    acpi-support-base			install
    acpid						install
    adduser						install
    apt					    	install
    apt-listchanges				install
    apt-utils					install
    aptitude					install
    aptitude-common				install
    aptitude-doc-en				install
    at					    	install
    ...
    yelp						install
    yelp-xsl					install
    zenity						install
    zenity-common				install
    zip						    install
    zlib1g:amd64				install
    zlib1g-dev:amd64			install
    </pre>
  * `dpkg -l "if*"` to see all packages starting with `if`.
  * `dpkg -L vim`: to list all the files that belong to a package.
  * `dpkg-reconfigure`: Debian-based systems often will configure a service during the install.
  Later should you need to reconfigure it, `dpkg-reconfigure` will re-run that stage of installation.
* Repositories
  * Debian-based systems can use repositories configured in `/etc/apt/sources.list`
    <pre>
    vagrant@ac:~$ <b>cat /etc/apt/sources.list</b>
    deb http://httpredir.debian.org/debian jessie main
    deb-src http://httpredir.debian.org/debian jessie main

    deb http://security.debian.org/ jessie/updates main
    deb-src http://security.debian.org/ jessie/updates main
    </pre>
      * Format is: `Type    Repository URL    Distro    Component`
  * Using `aptitude` or `apt-get` or `apt-cache` packages and their dependencies
  can be located and installed, removed, or upgraded.
  * `apt-get update`
  * `apt-cache search zsh`
  * `apt-get install zsh`
  * `apt-get purge zsh`
  * `apt-get autoremove`
  * `apt-get dist-upgrade`: upgrade to next release
  * Using `aptitude` is more reliable. Try `aptitude` with no options. Then `ctrl+t` to open up the menu.

## Use RPM and YUM package management
* RPM
  * Red Hat based system, SUSE.
  * we have command `rpm` and `.rpm` files.
  * `rpm` command is used to install and manage `.rpm` files.
  * `rpm -qa` to list all packages installed
    <pre>
    [vagrant@web ~]$ <b>rpm -qa</b>
    alsa-firmware-1.0.28-2.el7.noarch
    centos-release-7-1.1503.el7.centos.2.8.x86_64
    ebtables-2.0.10-13.el7.x86_64
    filesystem-3.2-18.el7.x86_64
    libpciaccess-0.13.1-4.1.el7.x86_64
    kbd-misc-1.15.5-11.el7.noarch
    kpartx-0.4.9-77.el7.x86_64
    linux-firmware-20140911-0.1.git365e80c.el7.noarch
    ...
    </pre>
  * 302 packaged were installed:
    <pre>
    [vagrant@web ~]$ <b>rpm -qa | wc -l</b>
    302
    </pre>
  * `rpm -ivh package.rpm` to get the information package. `v` for verbose. `h` for progressbar.
  * `rpm -U package.rpm`: updates the package.
  * `rpm -e package`: remove the package.
  * `rpm -qpR`: show dependencies.
* Repositories:
  * YUM repositories.
  * `yum install nmap`
  * `yum list installed`
  * `yum upgrade ‘cal*’ `
  * `yum info nmap`
  * `yum reinstall nmap`
  * `yum remove nmap`
  * YUM is configured in `/etc/yum.conf`.
    <pre>
    [vagrant@web ~]$ <b>cat /etc/yum.conf</b>
    [main]
    cacvar/log/yum.loghedir=/var/cache/yum/$basearch/$releasever
    keepcache=0
    debuglevel=2
    logfile=/
    exactarch=1
    obsoletes=1
    gpgcheck=1
    plugins=1
    installonly_limit=5
    bugtracker_url=http://bugs.centos.org/set_project.php?project_id=23&ref=http://bugs.centos.org/bug_report_page.php?category=yum
    distroverpkg=centos-release
    </pre>
  * Repository locations can be configured within `/etc/yum.repos.d/`.
    <pre>
    [vagrant@web ~]$ <b>ls /etc/yum.repos.d/</b>
    CentOS-Base.repo  CentOS-CR.repo  CentOS-Debuginfo.repo  CentOS-fasttrack.repo  CentOS-Sources.repo  CentOS-Vault.repo
    </pre>

    <pre>
    [vagrant@web yum.repos.d]$ <b>cat CentOS-Base.repo</b></pre>
  * `yumdownloader`: Download RPMs without installing them
    * from yum-utils package
    * If you need to download all the dependencies too, use the `--resolve` switch.

**rpm2cpio**
The *cpio* is kind of an archive, just like *zip*, *rar* or *tar*. the
`rpm2cpio` can convert rpm files to *cpio* archives so you can open them
using `cpio` command:
  * `rpm2cpio bzr-2.6.0-2.fc20.x86_64.rpm | cpio -idv`

## Work on the Command Line

### Bash
Bash is the most common Linux shell (CLI).

Linux commands can be broadly classified into **Internal** and **External** commands.
* Most external commands are stored in the form of binaries in `/bin` directory.
  <pre>
  [vagrant@web ~]$ <b>ll /bin</b>
  lrwxrwxrwx. 1 root root 7 Oct  6  2015 <b>/bin -> usr/bin</b>
  [vagrant@web ~]$ <b>cd /bin</b>
  [vagrant@web bin]$ <b>pwd</b>
  <b>/bin</b>
  [vagrant@web bin]$ <b>./pwd</b>
  <b>/usr/bin</b>
  </pre>
* To execute external command shell check `$PATH` variable . If command present in the
location mentioned in `$PATH` variable shell will execute it , otherwise it will give error
  <pre>
  [vagrant@web ~]$ <b>echo $PATH</b>
  /usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/vagrant/.local/bin:/home/vagrant/bin
  </pre>

### Paths

External commands are just files on disks. So where does bash knows where to find commands?
* `echo $PATH`
* `which ping`
* `whereis ping`
* `type ping`
* `type for`

If we need to add to the PATH variable, we generally will append to it.

In the current shell:
* `export PATH=$PATH:$HOME/bin`

* Every time the user logs in
<pre>
vi ~/.bashrc
export PATH=$PATH:/tmp/mypath # for example
export PATH=$PATH:$PWD
</pre>
<pre>
echo $PATH
</pre>

**running other commands which are not in the path**
* Give the full path of the command: `tmp/mypath/test.sh`
* Give the relative path: `.././mypath/test.sh`
* To add into the `PATH: PATH=$PATH:/tmp/mypath`
  * `vi ~/.bashrc`
  * `export PATH=$PATH:/tmp/mypath` or `PATH=$PATH:/tmp/mypath; export PATH`

### History

* The bash, saves the commands you issue in a file defined in HISTFILE ev.
the history shows the full command history (normally 500-1000 commands,
but can be changes via HISTSIZE)
* `history 20` shows last 20 commands
* `!!` executes last command
* `cd !$`: `!$` represents the last argument
* `!v`: rerun the last command in our history starting with character `v`
* `!string` most recent command that starts with string
* when logout, all these are saved in `.bash_history`
* `Ctrl+r` multiple times is very helpful in action. reverse searches for your input
  <pre>
  hossein@hossein ~/ansible_notes/server_oriented/lab_environment $ !vagrant
  vagrant ssh web
  Last login: Sat Dec 23 03:43:46 2017 from 10.0.2.2
  [vagrant@web ~]$
  </pre>

### Login Shell vs Non-login Shell
* **Login shell:** the log in script that we are going to run is in this order:
  *  `/etc/profile` Then
    * `~/.bash_profile` if it does'nt exist:
    * `~/.bash_login`
    * `~/.profile`
  * When we log out if there is `~/.bash_logout` we execute that.
* **Non-login shell**
  * `~/.bashrc`

## Process Text Streams Using Filters
  * `tac` to reverse the output of a file.
  * To number the lines use the command `nl`: `nl /etc/services`.
  To only see the number of lines in a file: `wc -l /etc/services`
  * `tail -f /var/log/messages` or `tail -f /var/log/syslog` To stop following: `ctrl + c`.
  * less used for paging large files. `/` start searching forwards. `?` start searching backwards. press `n` for the next match.
  * `uniq` removes duplicate entries from its input; before using `uniq`,
  we have to `sort` the input to `uniq`: `sort whatIHave.txt | uniq`
<pre>
hossein@hossein ~ $ <b>cat /etc/hosts</b>
127.0.0.1	localhost
127.0.1.1	hossein

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
</pre>
</pre>
To see the structure of a file: for example `^I` referes to tab, and `$` means enf of the line:
<pre>
hossein@hossein ~ $ <b>cat -vet !$</b>
127.0.0.1<b>^I</b>localhost<b>$</b>
127.0.1.1<b>^I</b>hossein<b>$</b>
$
# The following lines are desirable for IPv6 capable hosts$
::1     ip6-localhost ip6-loopback$
fe00::0 ip6-localnet$
ff00::0 ip6-mcastprefix$
ff02::1 ip6-allnodes$
ff02::2 ip6-allrouters$
</pre>

<pre>
hossein@hossein ~ $ <b>tac !$</b>
tac /etc/hosts
ff02::2 ip6-allrouters
ff02::1 ip6-allnodes
ff00::0 ip6-mcastprefix
fe00::0 ip6-localnet
::1     ip6-localhost ip6-loopback
# The following lines are desirable for IPv6 capable hosts

127.0.1.1	hossein
127.0.0.1	localhost
</pre>

<pre>
[vagrant@web ~]$ <b>tabs 8</b>
[vagrant@web ~]$ <b>echo -e "File\twith\ttabs" > ft</b>
[vagrant@web ~]$ <b>cat ft</b>
File	with	tabs
[vagrant@web ~]$ <b>cat -vet ft</b>
File^Iwith^Itabs$
[vagrant@web ~]$ <b>expand ft > fs</b>
[vagrant@web ~]$ <b>cat fs</b>
File    with    tabs
[vagrant@web ~]$ <b>cat -vet fs</b>
File    with    tabs$
[vagrant@web ~]$ <b>unexpand -a fs > ft</b>
[vagrant@web ~]$ <b></b>od -a ft
0000000   F   i   l   e  ht   w   i   t   h  ht   t   a   b   s  nl
0000017
[vagrant@web ~]$ <b>od -a fs</b>
0000000   F   i   l   e  sp  sp  sp  sp   w   i   t   h  sp  sp  sp  sp
0000020   t   a   b   s  nl
0000025
</pre>
To copy something to the current PWD:
<pre>
[vagrant@web ~]$ <b>cp /etc/hosts .</b>
[vagrant@web ~]$ <b>pwd</b>
/home/vagrant
[vagrant@web ~]$ <b>ls</b>
hosts
</pre>


### find
The `find` command helps us to find files based on many criteria. Look at these: 
* `find .`
* `find . -iname "*my*"` #not case sensitive
* `find . -empty` #finds empty files 


<pre>
[vagrant@web ~]$ <b>sudo find / -name "apache*"</b>
/etc/selinux/targeted/modules/active/modules/apache.pp
/usr/sbin/apachectl
/usr/share/man/man8/apachectl.8.gz
/usr/share/httpd/icons/apache_pb.gif
/usr/share/httpd/icons/apache_pb.png
/usr/share/httpd/icons/apache_pb.svg
/usr/share/httpd/icons/apache_pb2.gif
/usr/share/httpd/icons/apache_pb2.png
/usr/share/httpd/noindex/images/apache_pb.gif
</pre>
To look for directories use `-type d`. For file it is `-type f`
<pre>
[vagrant@web ~]$ <b>sudo find / -type d -name "httpd*"</b>
/run/httpd
/etc/httpd
/var/log/httpd
/var/cache/httpd
/usr/lib64/httpd
/usr/share/doc/httpd-tools-2.4.6
/usr/share/doc/httpd-2.4.6
/usr/share/httpd
/usr/libexec/initscripts/legacy-actions/httpd
</pre>

These are the most common file types: 
* `-type f` will search for a regular file 
* `-type d` will search for a directory 
* `-type l` will search for a symbolic link 
* `find /var -iname “*tmp*” -size +1M -size -100M` # finds all files ending in tmp with size between 1M and 100M in /var/ directory 

To search based on time: 
* `-atime -6` file was last accessed less than 6*24 hours ago 
* `-ctime +6` file was changed more than 6*24 hours ago  
* `-mtime -6` file content modification less than 6*24 hours ago 
* We also have   `-amin`, `-cmin`, and `-mmin` you guess! 

Acting on files 
* Switch   `-ls`: will run `ls -dils` on each file 
* Switch `-print`: will print the full name of the files on each line 
* Switch `-exec`: to run commands on found files; You can point to the
file with `{}` and finish your command with `\;` 
* `find . -empty -exec rm '{}' \;` 
* `find . -name "*.htm" -exec mv '{}' '{}l' \; `
* `find /etc -maxdepth 1 -type f -exec wc -l {} \; | sort -n`
* The `-maxdepth` tells the find how deep it should go into the directories 

Other options:
* The `-user` and `-group` specifies a specific user & group.
* Or even find the files not belonging to any user or group with `-nouser` and `-nogroup`
* Or even find the files not belonging to any user / group with `-nouser` and `-nogroup` 
* Add a `!` just before any phrase to negate it. So, this will find files not belonging to *hossein*: 
`find . ! -user hossein` 

### locate & updatedb
find is too slowwww. It searches the file system on each run but let’s see the fastest command:
`locate kernel`

If you create a new file and search it by locate command, you will not be given any
result unless you apply `sudo updatedb` command or wait for **1 day**.
You can see this database by `locate -S` command.

If you don't have `updatedb`, you have to install required package(s):
<pre>
[vagrant@web ~]$ <b>sudo yum install mlocate</b></pre>

<pre>
[vagrant@web ~]$ <b>sudo cat /etc/cron.daily/mlocate</b>
#!/bin/sh
nodevs=$(awk '$1 == "nodev" && $2 != "rootfs" && $2 != "zfs" { print $2 }' < /proc/filesystems)

renice +19 -p $$ >/dev/null 2>&1
ionice -c2 -n7 -p $$ >/dev/null 2>&1
/usr/bin/updatedb -f "$nodevs"
</pre>

<pre>
hossein@hossein ~ $ <b>locate -S</b>
Database <b>/var/lib/mlocate/mlocate.db</b>:
        16,834 directories
        116,206 files
        6,836,328 bytes in file names
        2,848,662 bytes used to store database
</pre>

### Search and replace with tr and sed

**tr**

* To convert everything in a text file to uppercase:
<pre>
[vagrant@web ~]$ <b>tr [a-z] [A-Z] < /etc/hosts</b>
127.0.0.1   WEB LOCALHOST LOCALHOST.LOCALDOMAIN LOCALHOST4 LOCALHOST4.LOCALDOMAIN4
::1         LOCALHOST LOCALHOST.LOCALDOMAIN LOCALHOST6 LOCALHOST6.LOCALDOMAIN6
</pre>

**sed**

`sed` is a powerful tool and stands for *stream editor* which allows you to filter and transform text.
It uses regular expressions and is a great tool for replacing text.
If you need to replace A with B only once in each line in a stream, you need to say `sed 's/A/B/g'`

<pre>
hossein@hossein ~ $ <b>sed 's/line/l/g' tac.py</b>
for l in reversed(list(open("test"))):
    print(l.rstrip())
</pre>

<pre>
[vagrant@web ~]$ <b>sed -n 's/bash/sh/p' /etc/passwd</b>
root:x:0:0:root:/root:/bin/<b>sh</b>
vagrant:x:1000:1000:vagrant:/home/vagrant:/bin/<b>sh</b>
</pre>

To delete comments on a text file then print it:
<pre>
hossein@hossein ~ $ <b>sed '/^#/d' /etc/ntp.conf</b>

driftfile /var/lib/ntp/ntp.drift


statistics loopstats peerstats clockstats
filegen loopstats file loopstats type day enable
filegen peerstats file peerstats type day enable
filegen clockstats file clockstats type day enable


pool 0.ubuntu.pool.ntp.org iburst
pool 1.ubuntu.pool.ntp.org iburst
pool 2.ubuntu.pool.ntp.org iburst
pool 3.ubuntu.pool.ntp.org iburst

pool ntp.ubuntu.com


restrict -4 default kod notrap nomodify nopeer noquery limited
restrict -6 default kod notrap nomodify nopeer noquery limited

restrict 127.0.0.1
restrict ::1

restrict source notrap nomodify noquery
</pre>

Using `sed`, we can edit the file directly. In this example we want to remove all lines from `/etc/ntp.conf`
and write the content into it.
<pre>
hossein@hossein ~ $ <b>sudo scp /etc/ntp.conf vagrant@192.168.33.20:</b>
vagrant@192.168.33.20's password:
ntp.conf                                                                                                  100% 2424     2.4KB/s   00:00
[vagrant@web ~]$ <b>ls</b>
ntp.conf
[vagrant@web ~]$ <b>sed -i.bak '/^\s*#/d;/^$/d' ntp.conf</b>
[vagrant@web ~]$ <b>cat ntp.conf</b>
driftfile /var/lib/ntp/ntp.drift
statistics loopstats peerstats clockstats
filegen loopstats file loopstats type day enable
filegen peerstats file peerstats type day enable
filegen clockstats file clockstats type day enable
pool 0.ubuntu.pool.ntp.org iburst
pool 1.ubuntu.pool.ntp.org iburst
pool 2.ubuntu.pool.ntp.org iburst
pool 3.ubuntu.pool.ntp.org iburst
pool ntp.ubuntu.com
restrict -4 default kod notrap nomodify nopeer noquery limited
restrict -6 default kod notrap nomodify nopeer noquery limited
restrict 127.0.0.1
restrict ::1
restrict source notrap nomodify noquery
[vagrant@web ~]$ <b>cat ntp.conf.bak</b>
# /etc/ntp.conf, configuration for ntpd; see ntp.conf(5) for help

driftfile /var/lib/ntp/ntp.drift

# Enable this if you want statistics to be logged.
#statsdir /var/log/ntpstats/

statistics loopstats peerstats clockstats
...
</pre>


### grep

<pre>
hossein@hossein ~ $ <b>grep hossein /etc/group</b>
...
cdrom:x:24:hossein
...
hossein:x:1000:
...
wireshark:x:134:hossein
</pre>

<pre>
hossein@hossein ~ $ <b>grep 'sd[abc]' /proc/mounts</b>
/dev/sda2 / ext4 rw,relatime,errors=remount-ro,data=ordered 0 0
/dev/sda1 /boot/efi vfat rw,relatime,fmask=0077,dmask=0077,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro 0 0
/dev/sda2 /var/lib/docker/plugins ext4 rw,relatime,errors=remount-ro,data=ordered 0 0
/dev/sda2 /var/lib/docker/aufs ext4 rw,relatime,errors=remount-ro,data=ordered 0 0
</pre>

Exercise: How many words in English contain 4 vowel in-row?

<pre>
hossein@hossein ~ $ <b>egrep '[aeiou]{4}' /usr/share/dict/words | wc -l</b>
36
</pre>

### Use streams, pipes and redirects
On a Linux system most shells use streams for input and output (a list of characters).
These streams can be from (and toward) various things including keyboard,
block devices (hards, usb stick,...), window of a program, files,...
* `stdin` is the standard input stream, which provides input to commands (file descriptor 0)
* `stdout` is the standard output stream, which displays output from commands (file descriptor 1)
* `stderr` is the standard error stream, which displays error output from commands (file descriptor 2)

**What these file descriptions (0, 1, and 2) are?**

They are used to control the output. If you need to control where your output to go,
you can add `n>` or `n>>` (n=1 or 2 or 3)
* `n>` redirects file description `n` to a file or device. If the file
already exists, it is overwritten and if it does not exist, it will be created.
* `n>>` redirects file description `n` to a file or device. If the file already exists the stream will be appended to the end of it and if it does not exist, it will be created
if the n is not given, the default is standard output.
  * `ls 1> listfiles` is the same as `ls > listfiles`
  * `ls 1>> listfiles` is the same as `ls >> listfiles`
* The user who runs the command should have write access to the file.

<pre>
[vagrant@web ~]$ <b>tar -czvf etc.tar.gz /etc 2> /dev/null</b>
/etc/
/etc/fstab
/etc/mtab
/etc/resolv.conf
/etc/pki/
...
[vagrant@web ~]$ <b>tar -xf etc.tar.gz</b>
[vagrant@web ~]$ <b>cd etc</b>
[vagrant@web etc]$ <b>ls</b>
adjtime                  dnsmasq.conf   init.d                    motd               profile.d         shells
aliases                  dnsmasq.d      inittab                   mtab               protocols         skel
aliases.db               dracut.conf    inputrc                   my.cnf             rc0.d             ssh
...
</pre>
Sometimes (say during automated tasks) we prefer to send both standard
output and standard error to same place:
<pre>
[vagrant@web ~]$ <b>tar -czvf etc.tar.gz /etc 1> backup.log 2>&1</b>
[vagrant@web ~]$ <b>ls</b>
backup.log  etc.tar.gz
</pre>
Which is the same as:
<pre>
[vagrant@web ~]$ <b>tar -czvf etc.tar.gz /etc &> backup.log</b>
[vagrant@web ~]$ <b>ls</b>
backup.log  etc.tar.gz
</pre>

Another example
<pre>
[vagrant@web ~]$ <b>strace ls -l 2>&1 | grep passwd</b>
open("/etc/passwd", O_RDONLY|O_CLOEXEC) = 4
</pre>

#### Pipes:

<pre>
[vagrant@web ~]$ <b>cd /lib/modules/$(uname -r)</b>
[vagrant@web 3.10.0-229.el7.x86_64]$ pwd
/lib/modules/3.10.0-229.el7.x86_64
</pre>

<pre>
[vagrant@web ~]$ <b>for i in $(ls); do stat $i; done</b>
  File: ‘hosts’
  Size: 28        	Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d	Inode: 135272598   Links: 1
Access: (0664/-rw-rw-r--)  Uid: ( 1000/ vagrant)   Gid: ( 1000/ vagrant)
Context: unconfined_u:object_r:user_home_t:s0
Access: 2017-12-23 23:09:41.127202016 +0000
Modify: 2017-12-23 23:09:36.533202016 +0000
Change: 2017-12-23 23:09:36.533202016 +0000
 Birth: -
</pre>
<pre>
[vagrant@web ~]$ <b>ls</b>
file1   file11  file13  file15  file17  file19  file20  file4  file6  file8  hosts
file10  file12  file14  file16  file18  file2   file3   file5  file7  file9  servers
[vagrant@web ~]$ <b>ls | xargs grep 13526329*</b>
file1 file10 file11 file12 file13 file14 file15 file16 file17 file18 file19 file2 file20 file3 file4 file5 file6 file7 file8 file9 hosts servers
</pre>

**tee**

What if you need to see the output on screen and save it to a file?

<pre>
[vagrant@web ~]$ <b>ls | tee file1</b>
<b>file1</b>
hosts
servers
[vagrant@web ~]$ <b>cat file1</b>
file1
hosts
servers
</pre>

if you want to prevent overwriting files, use the -a switch to append to files if exists.

**Some examples of `stdin`:**

The `<` operand redirects the input.

Example:
<pre>
hossein@hossein ~ $ <b>tr ' ' ',' < /etc/hosts</b> #to substitute each space by comma
127.0.0.1	localhost
127.0.1.1	hossein

#,The,following,lines,are,desirable,for,IPv6,capable,hosts
::1,,,,,ip6-localhost,ip6-loopback
</pre>
Another example:
<pre>
[vagrant@web ~]$ <b>while read i; do ping -c2 $i; done < hosts</b>
PING 192.168.33.10 (192.168.33.10) 56(84) bytes of data.
64 bytes from 192.168.33.10: icmp_seq=1 ttl=64 time=0.532 ms
64 bytes from 192.168.33.10: icmp_seq=2 ttl=64 time=0.622 ms

--- 192.168.33.10 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.532/0.577/0.622/0.045 ms
PING 192.168.33.30 (192.168.33.30) 56(84) bytes of data.
64 bytes from 192.168.33.30: icmp_seq=1 ttl=64 time=0.449 ms
64 bytes from 192.168.33.30: icmp_seq=2 ttl=64 time=0.613 ms

--- 192.168.33.30 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.449/0.531/0.613/0.082 ms
</pre>

## Perform some file management
<pre>
hossein@hossein ~ $ <b>ls /dev/sd[abc]?</b>
/dev/sda1  /dev/sda2  /dev/sda3
[vagrant@web ~]$ <b>ls</b> #Nothing is in home directory
[vagrant@web ~]$ <b>mkdir -p parent/child</b>
[vagrant@web ~]$ <b>tree</b>
.
└── parent
    └── child

2 directories, 0 files
[vagrant@web ~]$ <b>mkdir dir</b>
[vagrant@web ~]$ <b>tree</b>
.
├── dir
└── parent
    └── child

3 directories, 0 files
[vagrant@web ~]$ <b>cp -R parent/ dir/</b>
[vagrant@web ~]$ <b>tree</b>
.
├── dir
│   └── parent
│       └── child
└── parent
    └── child

</pre>

**dd**

The `dd` command copies data from one location to another. This data
comes from files. You can use it just like copy:

<pre>
[vagrant@web ~]$ <b>dd if=/etc/hosts of=./hosts</b>
0+1 records in
0+1 records out
162 bytes (162 B) copied, 0.000245596 s, 660 kB/s
[vagrant@web ~]$ <b>ls</b>
dir  <b>hosts</b>  parent
[vagrant@web ~]$ <b>file hosts</b> # To identify a file
hosts: ASCII text
</pre>
Identifying a file:
<pre>
[vagrant@web ~]$ <b>file $(which ping)</b>
/usr/bin/ping: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.32, BuildID[sha1]=0xa00a6dcfec2962f8ce71a476395da21c434e23af, stripped
</pre>
But it is used in many other cases specially writing directly to block
devices such as `/dev/sdb` or changing data to upper/lower case
* `dd if=/dev/sda of=backup.dd bs=4096` # to backup whole hard to a file

To restore the backup:
* `sudo dd of=/dev/sda if=backup.dd bs=4096`

To make a bootable flash memory from an iso file:
* `sudo dd if=ubuntu.iso of=/dev/sdb bs=4096`

Another common usage is creating files of specific size:
* `dd if=/dev/zero of=1g.test bs=1G count=1`

<pre>
[vagrant@web ~]$ <b>for i in `seq 1 1 4`; do dd if=/dev/zero of=file$i bs=100M count=1; done</b>
1+0 records in
1+0 records out
104857600 bytes (105 MB) copied, 0.108891 s, 963 MB/s
1+0 records in
1+0 records out
104857600 bytes (105 MB) copied, 0.0893152 s, 1.2 GB/s
1+0 records in
1+0 records out
104857600 bytes (105 MB) copied, 0.110539 s, 949 MB/s
1+0 records in
1+0 records out
104857600 bytes (105 MB) copied, 0.107373 s, 977 MB/s
[vagrant@web ~]$ <b>ls -ltrh</b>
total 400M
-rw-rw-r--. 1 vagrant vagrant 100M Dec 23 22:13 file1
-rw-rw-r--. 1 vagrant vagrant 100M Dec 23 22:13 file2
-rw-rw-r--. 1 vagrant vagrant 100M Dec 23 22:13 file3
-rw-rw-r--. 1 vagrant vagrant 100M Dec 23 22:13 file4

</pre>

**cut**
`cut` command will cut a column of one file. Good for separating fields 

Example:
<pre>
[vagrant@web ~]$ <b>grep vagrant /etc/passwd | <u>cut -f1 -d':'</u> | tr [:lower:] [:upper:]</b>
VAGRANT
</pre>

### Archiving files

**tar**

to archive all files in the current directory:
* `tar -cf myarchive.tar *`

then we can zip this archive:
* `gzip myarchive.tar.gz`

or we can archive and zip files altogether (z switch)
* `tar -cf myarchive.tar.gz -z *`

to extract:
* `tar -xf myarchive.tar.gz`

to append files to an archive:
* `tar -rf myarchive.tar newww`

<pre>
[vagrant@web ~]$ <b>tar -cf myarchive.tar *</b>
[vagrant@web ~]$ <b>ls</b>
file1  file2  file3  file4  <b>myarchive.tar</b>
</pre>

<pre>
[vagrant@web ~]$ <b>tar -cf myarchive.tar.gz -z *</b>
[vagrant@web ~]$ <b>ls</b>
file1  file2  file3  file4  <b>myarchive.tar.gz</b></pre>

**cpio**

`cpio` Gets a list of files and creates archive (one file) which can be
opened later; `-o` makes `cpio` to create an output from its input:
<pre>
[vagrant@web ~]$ <b>sudo find /var/log -type f -mtime +1 | cpio -o > log.cpio</b></pre>

To extract:
<pre>
[vagrant@web ~]$ <b>mkdir extract</b>
[vagrant@web ~]$ <b>mv log.cpio extract/</b>
[vagrant@web ~]$ <b>cd extract/</b>
[vagrant@web extract]$ <b>cpio -id --no-absolute-filenames < log.cpio</b>
cpio: Removing leading `/' from member names
413 blocks
[vagrant@web <b>extract</b>]$ <b>tree</b>
.
├── log.cpio
└── <b>var</b>
    └── <b>log</b>
        ├── vboxadd-install.log
        ├── vboxadd-install-x11.log
        └── VBoxGuestAdditions.log

2 directories, 4 files
</pre>

## Manage Disk Quotas
* Quotas are applied to a file system as a *mount option*.
* Either per user or per group.
* Migrating user home directories:
  * First use the single user mode to ensure users can't access their data during the move
  * `# mount /dev/sdXN /mnt`: Mount the partition to a temporary directory
  * `apt install quota rsync` or `yum install quota rsync`
  * `# resync -av /home/ /mnt`: Synchronize the content of home through to mount. The idea of rsync to copy user data is that we can resume if we need to pause it.
  * `rm -rf /home/*`
  * Add entry to `/etc/fstab`: Using `mount -a` will read the `/etc/fstab` file and mount all partitions
  that are not currently mounted (at least that have the auto option which is the default)
    * `UUID=xxxx-xxxx /home ext4 usrquota,auto 0 2`
    * `# mount -a`
  * Create our quota database: The quota database will be created at the root of the partition and called `aquota.user`
    * `# quotacheck -cmu /dev/sdXN`
    * `# quotaon /dev/sdXN`
    * `# setquota -u vagrant 100000 150000 0 0 /dev/sdXN`
    * To see the quota usage: `repquota -au /dev/sdXN`
* `XFS` as better quota management
  * `xfs_quota` will be used
* I tested that `btrfs` doesn't support user quota.

* **Lab**

Currently the home directory is 39 GiB. I want to change it to 3.0 GiB.

<pre>
[vagrant@web /]$ df -hT /home
Filesystem              Type  <b>Size</b>  Used Avail Use% Mounted on
/dev/mapper/centos-root xfs    <b>39G</b>  945M   38G   3% /
</pre>

<pre>
The rest of the lab will be provided later ...
</pre>

## Manage file permissions and ownership

### File ownership & permissions

<pre>-rwxrw-r--</pre>
* Dash (-) is for ordinary files, 'l' is for links & 'd' is for directory
* user's access (read, write, execute)
* group's access (read, write)
* others (read)
<pre>
[vagrant@web ~]$ touch file1
[vagrant@web ~]$ ls -lh file1
-rw-rw-<b>r--</b>. 1 vagrant vagrant 0 Dec 24 23:13 file1
[vagrant@web ~]$ umask
0002
</pre>

`umask` tells the system what permissions should not be given to new files
(in this example, when we create a file, write permission (2) is denied for others)

<pre>
[vagrant@web ~]$ umask 0
[vagrant@web ~]$ touch file2
[vagrant@web ~]$ ls -lh file2
-rw-rw-<b>rw-</b>. 1 vagrant vagrant 0 Dec 24 23:16 file2
</pre>

For example `umask 077` makes the new files totally private.

**Ownership**

When a new file is created the user owner is the current **UID** and, so long as no
special permissions are set on the directory, the **GID** of the current user becomes the group owner.

Users have a primary group which is listed in the `/etc/passwd` file against the user.
This becomes their default **GID**. Secondary groups can be used to gain permissions to a resource.

* Changing ownership and groups
  * To change your **GID**, use the `/usr/bin/newgrp` command.
  * File ownership needs to be manipulated as root.
  * `/bin/chown` can change user and group ownership.
    * `chown user1 hello.sh`
    * `chown user1:users hello.sh`

It is possible to change the permissions on files & directories using the `chmod` command.
There are two ways to tell this command what you want to do:
* using octal codes (r=4, w=2, x=1)
  * e.g if you want to give `rwx` to owner, `rx` to group and only `x` to others, you must use `751`
    * `chmod 751 myfile`
* using short codes
  * You can use `+x` to give execute permission, `+r` to give read permission and `+w` to give read permission.
  Removing these permissions will be like `-r`
    * `chmod u-x myfile`
    * `chmod +x myfile`
    * `chmod uo+xr myfile`
* One very common switch on `chmod` is `-R` for recursive chmoding on files.
Below example will give read permission of all files inside `/tmp/` to any user
  * `chmod -R o+r /tmp`
  * `chmod -R hossein:group1 /net/10.120.1.10/hossein`
  * <pre>
    [vagrant@web ~]$ umask
    0002
    [vagrant@web ~]$ ./hello.sh
    -bash: ./hello: Permission denied
    [vagrant@web ~]$ chmod u+x hello
    [vagrant@web ~]$ ./hello.sh
    Hello World!
    </pre>

**Lab:**
<pre>
[vagrant@web ~]$ ll newfile
-rw-r--r--. 1 <b>root cdrom</b> 0 Dec 24 23:36 newfile
[vagrant@web ~]$ <b>sudo chgrp vagrant newfile</b>
[vagrant@web ~]$ ll newfile
-rw-r--r--. 1 <b>root vagrant</b> 0 Dec 24 23:36 newfile
[vagrant@web ~]$ <b>sudo chown vagrant newfile</b>
[vagrant@web ~]$ ll newfile
-rw-r--r--. 1 <b>vagrant vagrant</b> 0 Dec 24 23:36 newfile
</pre>

#### Special permissions
* Set sticky bit (`t`): when set on a directory, you can only delete files you own, such as in `/tmp`
  * `chmod o+t /shared`
  * `chmod 1777 /shared`
  * `-T` means noexec
* Set group ID bit (`s`): When set on an **executable** file, the file runs with GID of the group owner of the file
  * Enabled by default on `/usr/bin/wall`
  * `chmod g+s /shared`
  * `chmod 2777 /shared`
  * `chmod 3777 /shared` we often want to set the sticky bit as well.
* Set user ID bit (`s`): **Executable** files with the SUID mode set run with the UID of the user owner of the file.
  * This is enabled by default on `/usr/bin/passwd`
  * `chmod u+s /usr/bin/passwd`
  * `chmod 4755 /usr/bin/passwd`


<pre>
[vagrant@web ~]$ <b>ls -ld /tmp</b>
drwxrwxrw<b>t</b>. 7 root root 88 Dec 24 23:35 /tmp
[vagrant@web ~]$ <b>ll /usr/bin/wall</b>
-r-xr-<b>s</b>r-x. 1 root tty 15344 Jun  9  2014 /usr/bin/wall
[vagrant@web ~]$ <b>ll /usr/bin/passwd</b>
-rw<b>s</b>r-xr-x. 1 root root 27832 Jun 10  2014 /usr/bin/passwd
</pre>

<pre>
[vagrant@web ~]$ <b>find /usr/bin -perm -u+s</b>
/usr/bin/mount
/usr/bin/su
/usr/bin/chage
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/chsh
/usr/bin/umount
/usr/bin/pkexec
/usr/bin/crontab
/usr/bin/passwd
/usr/bin/sudo
</pre>

## Create and change hard & symbolic links
On a storage device, a file or directory is contained in a collection of blocks.
Information about a file is contained in an inode which records information such as the owner;
when the file was last accessed, how large it is, whether it is a directory or not,
and who can read from or write to it
* `ls -i` #to see the inode

#### hard links
Mainly hard links will be used for directories and just the default of `.` and `..` directories.
A hard linked file is a file with more than one name.

The first thing to note is that this type of file does not show as a link. Here we see `d` for directory and
even if it were a regular file, it would NOT be of a link type.
That hard link count being greater than 1, identifies this as being hard linked.

Directories always have a hard count > 1
<pre>
[vagrant@web ~]$ <b>ls -ld /usr/share</b>
<b>d</b>rwxr-xr-x. <b>72</b> root root 4096 Oct  6  2015 /usr/share
</pre>
`72` here means that you have 70 subdirectories in `/usr/share`
<pre>
[vagrant@web ~]$ <b>ll /usr/share | grep ^d | wc -l</b>
70
</pre>

**Hard links on files**

<pre>
[vagrant@web ~]$ echo hello > file1
[vagrant@web ~]$ ln file1 file2
[vagrant@web ~]$ echo world >> file2
[vagrant@web ~]$ cat file1
hello
world
[vagrant@web ~]$ ls -li
<b>135272594</b> -rw-rw-r--. <b>2</b> vagrant vagrant 12 Dec 25 00:47 file1
<b>135272594</b> -rw-rw-r--. <b>2</b> vagrant vagrant 12 Dec 25 00:47 file2
</pre>
* Hard links are links ti inodes.
* `ls -l` shows the same size for both files
* If we remove one, we still have the other one
* Restriction of hard links:
  * Must be in the same file system. So you cannot hard link from /home to /root.
  * We cannot hard link to directories only system can.

#### soft links or symbolic links, like shortcut concept in Windows
* has file type of `l`.
* Can cross file system boundaries.
* If the target is deleted, the link itself is broken.
* It is like shortcut in windows.
* It is not link to the inode but the file path and name itself.

<pre>
[vagrant@web ~]$ ls -li
135272594 -rw-rw-r--. 2 vagrant vagrant 12 Dec 25 00:47 file1
<b>135272594</b> -rw-rw-r--. 2 vagrant vagrant 12 Dec 25 00:47 file2
<b>135272593</b> <b>l</b>rwxrwxrwx. 1 vagrant vagrant  5 Dec 25 00:54 <b>file3 -> file2</b>
</pre>

* Symbolic Links Administration
<pre>
[vagrant@web ~]$ <b>ls -l /lib/systemd/system/multi-user.target.wants/</b>
total 0
lrwxrwxrwx. 1 root root 16 Oct  6  2015 brandbot.path -> ../brandbot.path
lrwxrwxrwx. 1 root root 15 Oct  6  2015 dbus.service -> ../dbus.service
lrwxrwxrwx. 1 root root 15 Oct  6  2015 getty.target -> ../getty.target
lrwxrwxrwx. 1 root root 24 Oct  6  2015 plymouth-quit.service -> ../plymouth-quit.service
lrwxrwxrwx. 1 root root 29 Oct  6  2015 plymouth-quit-wait.service -> ../plymouth-quit-wait.service
lrwxrwxrwx. 1 root root 33 Oct  6  2015 systemd-ask-password-wall.path -> ../systemd-ask-password-wall.path
lrwxrwxrwx. 1 root root 25 Oct  6  2015 systemd-logind.service -> ../systemd-logind.service
lrwxrwxrwx. 1 root root 32 Oct  6  2015 systemd-user-sessions.service -> ../systemd-user-sessions.service
</pre>

Let's find symbolic links in /etc/httpd
<pre>
[vagrant@web ~]$ <b>sudo find /etc/httpd -type l -name "*"</b>
/etc/httpd/logs
/etc/httpd/modules
/etc/httpd/run
</pre>

To create a new symlink (will fail if symlink exists already):
* `ln -s /path/to/file /path/to/symlink`
To create or update a symlink:
* `ln -sf /path/to/file /path/to/symlink`

## Customize and use the shell environment
* Local variables
  * `VARNAME=varvalue`
* Environment Variable
  * `export VARNAME=varvalue`
* `env` displays the current environment variables
* `env NAME=val command`
    <pre>
    [vagrant@web ~]$ <b>date</b>
    Mon Dec 25 01:51:02 <b>UTC</b> 2017
    [vagrant@web ~]$ <b>env TZ='America/Toronto' date</b> # Runs the date command using a set time zone
    Sun Dec 24 20:51:12 <b>EST</b> 2017
    </pre>

    <pre>
    [vagrant@web ~]$ <b>PS1="[\d \t \u@\H:\w ] "</b>
    [<b>Mon Dec 25 04:42:40</b> vagrant@web:~ ] <b>sudo cp -f /usr/share/zoneinfo/Canada/Eastern /etc/localtime</b>
    [<b>Sun Dec 24 23:42:57</b> vagrant@web:~ ] <b>date</b>
    Sun Dec 24 23:43:00 EST 2017
    </pre>

* `echo $SHELL`
* Aliases
  * `alias` show us the current aliases
  * `\ls` forces `ls` command as not in aliases

## Manage shared libraries
**Shared Libraries:** Most applications will make use of dynamically linked
shared libraries, compiled code that can be used by many applications.

**Display dependencies for a given application:**
<pre>
hossein@hossein ~ $ <b>ldd /bin/ping</b>
	linux-vdso.so.1 =>  (0x00007ffd3f7a8000)
	libcap.so.2 => /lib/x86_64-linux-gnu/libcap.so.2 (0x00007fd821381000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fd820fb7000)
	/lib64/ld-linux-x86-64.so.2 (0x0000562947df9000)
</pre>

<pre>
hossein@hossein ~ $ <b>ldd $(which ping)</b>
	linux-vdso.so.1 =>  (0x00007ffdca1b2000)
	libcap.so.2 => /lib/x86_64-linux-gnu/libcap.so.2 (0x00007f6c19ba7000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f6c197dd000)
	/lib64/ld-linux-x86-64.so.2 (0x0000561e3d4c7000)
</pre>

You should know that libraries are installed in `/lib` and `/lib64` (for 32bit and 64bit libraries)
* `ls /lib`
* `ls /lib64`
* `ls /usr/lib/`
* `ls /usr/lib64/`

**Dynamic library config:**

As most of other Linux tools, dynamic linking is also configured using
a text config file. It is located at `/etc/ld.so.conf`. On an Ubuntu
system it is just points to other config files in `/etc/ld.so.conf.d/`
but all those lines can be included in the main file too

`$ cat /etc/ld.so.conf`

`$ ls /etc/ld.so.conf.d/`
We need to be fast; we cannot iterate through all conf files.
`ldconfig` command processes all these files and creates `ld.so.cache` in `/etc`

<pre># ldconfig</pre>
<pre># cat /etc/ld.so.cache</pre>

* **LD_LIBRARY_PATH** is the environment variable path in Linux; you can
see all environment variables by `export` command

<pre>
[root@web ~]# <b>export</b>
declare -x HISTCONTROL="ignoredups"
declare -x HISTSIZE="1000"
declare -x HOME="/root"
declare -x HOSTNAME="web"
declare -x LANG="en_CA.UTF-8"
declare -x LESSOPEN="||/usr/bin/lesspipe.sh %s"
declare -x LOGNAME="root"
declare -x LS_COLORS="rs=0:di=38;5;27:ln=38;5;51:mh=44;38;5;15:pi=40;38;5;11:so=38;5;13:do=38;5;5:bd=48;5;232;38;5;11:cd=48;5;232;38;5;3:or=48;5;232;38;5;9:mi=05;48;5;232;38;5;15:su=48;5;196;38;5;15:sg=48;5;11;38;5;16:ca=48;5;196;38;5;226:tw=48;5;10;38;5;16:ow=48;5;10;38;5;21:st=48;5;21;38;5;15:ex=38;5;34:*.tar=38;5;9:*.tgz=38;5;9:*.arc=38;5;9:*.arj=38;5;9:*.taz=38;5;9:*.lha=38;5;9:*.lz4=38;5;9:*.lzh=38;5;9:*.lzma=38;5;9:*.tlz=38;5;9:*.txz=38;5;9:*.tzo=38;5;9:*.t7z=38;5;9:*.zip=38;5;9:*.z=38;5;9:*.Z=38;5;9:*.dz=38;5;9:*.gz=38;5;9:*.lrz=38;5;9:*.lz=38;5;9:*.lzo=38;5;9:*.xz=38;5;9:*.bz2=38;5;9:*.bz=38;5;9:*.tbz=38;5;9:*.tbz2=38;5;9:*.tz=38;5;9:*.deb=38;5;9:*.rpm=38;5;9:*.jar=38;5;9:*.war=38;5;9:*.ear=38;5;9:*.sar=38;5;9:*.rar=38;5;9:*.alz=38;5;9:*.ace=38;5;9:*.zoo=38;5;9:*.cpio=38;5;9:*.7z=38;5;9:*.rz=38;5;9:*.cab=38;5;9:*.jpg=38;5;13:*.jpeg=38;5;13:*.gif=38;5;13:*.bmp=38;5;13:*.pbm=38;5;13:*.pgm=38;5;13:*.ppm=38;5;13:*.tga=38;5;13:*.xbm=38;5;13:*.xpm=38;5;13:*.tif=38;5;13:*.tiff=38;5;13:*.png=38;5;13:*.svg=38;5;13:*.svgz=38;5;13:*.mng=38;5;13:*.pcx=38;5;13:*.mov=38;5;13:*.mpg=38;5;13:*.mpeg=38;5;13:*.m2v=38;5;13:*.mkv=38;5;13:*.webm=38;5;13:*.ogm=38;5;13:*.mp4=38;5;13:*.m4v=38;5;13:*.mp4v=38;5;13:*.vob=38;5;13:*.qt=38;5;13:*.nuv=38;5;13:*.wmv=38;5;13:*.asf=38;5;13:*.rm=38;5;13:*.rmvb=38;5;13:*.flc=38;5;13:*.avi=38;5;13:*.fli=38;5;13:*.flv=38;5;13:*.gl=38;5;13:*.dl=38;5;13:*.xcf=38;5;13:*.xwd=38;5;13:*.yuv=38;5;13:*.cgm=38;5;13:*.emf=38;5;13:*.axv=38;5;13:*.anx=38;5;13:*.ogv=38;5;13:*.ogx=38;5;13:*.aac=38;5;45:*.au=38;5;45:*.flac=38;5;45:*.mid=38;5;45:*.midi=38;5;45:*.mka=38;5;45:*.mp3=38;5;45:*.mpc=38;5;45:*.ogg=38;5;45:*.ra=38;5;45:*.wav=38;5;45:*.axa=38;5;45:*.oga=38;5;45:*.spx=38;5;45:*.xspf=38;5;45:"
declare -x MAIL="/var/spool/mail/root"
declare -x OLDPWD
declare -x PATH="/usr/local/sbin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin"
declare -x PWD="/root"
declare -x SHELL="/bin/bash"
declare -x SHLVL="1"
declare -x SUDO_COMMAND="/bin/bash"
declare -x SUDO_GID="1000"
declare -x SUDO_UID="1000"
declare -x SUDO_USER="vagrant"
declare -x TERM="xterm-256color"
declare -x USER="root"
declare -x USERNAME="root"
</pre>

if you give this command:
`export LD_LIBRARY_PATH=/usr/lib/myoldlibs:/home/hossein/libs/`
and then run any command, the system will search `/usr/lib/myoldlibs`
and then `/home/hossein/libs/` before going to the main system libraries
(defined in `ld.so.cache`)

## Manage user and group accounts and related system files
* Databases:
  * `/etc/passwd`: The main user account database that can but usually does not contain the user password.
  * `/etc/shadow`": This is the file normally will contain user password and other shadow data such as password last changed date.
  * `/etc/group`: stores group accounts.
* `getent`: This command can be used to list all entries particular database. For example `getent passwd`

### Create users
<pre>
[root@localhost ~]# <b>useradd -D</b>
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes
[root@localhost ~]# vim /etc/default/useradd # useradd -D shows the default values when adding a new user. We can change the defaults in /etc/default/useradd
[root@localhost ~]# <b>tail -n1 /etc/passwd</b>
torben:x:1001:1001::/home/torben:/bin/bash
[root@localhost ~]# <b>useradd -m soren</b>
[root@localhost ~]# <b>!ta</b>
tail -n1 /etc/passwd
<b>soren</b>:x:1002:1002::/home/soren:/bin/sh
[root@localhost ~]# <b>useradd -m -g 100 uwe</b>
[root@localhost ~]# !ta
tail -n1 /etc/passwd
uwe:x:1003:<b>100</b>::/home/uwe:/bin/sh
</pre>
`-m` creates home directory which comes from `/etc/skel` directory.

### Give the user a password
<pre>
[root@localhost ~]# <b>grep uwe /etc/shadow</b>
uwe:<b>!!</b>:17526:0:99999:7:::
[root@localhost ~]# <b>passwd -S uwe</b>
uwe LK 2017-12-25 0 99999 7 -1 (Password locked.)
[root@localhost ~]# <b>chage -l uwe</b>
Last password change					: Dec 26, 2017
Password expires					: never
Password inactive					: never
Account expires						: never
Minimum number of days between password change		: 0
Maximum number of days between password change		: 99999
Number of days of warning before password expires	: 7
[root@localhost ~]# <b>passwd uwe</b>
Changing password for user uwe.
New password:
Retype new password:
[root@localhost ~]# <b>!gr</b>
grep uwe /etc/shadow
uwe:<b>$6$NII.1Rsu$uoCTGpPGghKbgQuUxGHhsTVLZSmLOgJFDqhQ7FMlpgJVD7FpgByffk3kaw3td8zKhQUu3q.AtaymOc559Bwk.0</b>:17526:0:99999:7:::
</pre>
* `17526` last time the password changed. Number of days after 01/01/1970
* `!` Password is locked or there is no password
* `0` User cannot change the password 0 days after each change
* `99999` Every 99999, the user has to change his password
* `7` The user will be informed 7 days before the expiration to change his password
* Now let's change some of these values
<pre>
[root@localhost ~]# <b>passwd -x 90 uwe</b>
Adjusting aging data for user uwe.
passwd: Success
[root@localhost ~]# <b>passwd -S uwe</b>
uwe PS 2017-12-25 0 90 7 -1 (Password set, SHA512 crypt.)
</pre>

### Group accounts
groups are located in `/etc/group`
  * `adm` administrative group in Debian
  * `wheel` administrative group in Red Hat

We want to see what users are in the `wheel` group:
<pre>
[root@localhost ~]# grep wheel /etc/group
wheel:x:10:letmein # We can see user letmein is in the admin group
</pre>

we want to see the group memberships that *uwe* belongs to:
<pre>
[root@localhost ~]# id uwe
uid=1003(uwe) gid=100(users) groups=100(users)
</pre>

Now we want to add uwe to the *wheel* group. We modify the user account by: `usermod` with switch `-aG` to add to more groups
<pre>
[root@localhost ~]# <b>usermod -a -G wheel uwe</b>
[root@localhost ~]# <b>id uwe</b>
uid=1003(uwe) gid=100(users) groups=100(users),10(wheel)
[root@localhost ~]# <b>grep wheel /etc/group</b>
wheel:x:10:letmein,uwe
[root@localhost ~]# <b>sed '/^#/d' /etc/sudoers</b> # Let's see if the user uwe is part of sudoers
root	ALL=(ALL) 	ALL
%wheel	ALL=(ALL)	ALL # Yes he is!
</pre>

<pre>
[vagrant@web ~]$ <b>lastlog</b>
Username         Port     From             Latest
root                                       **Never logged in**
bin                                        **Never logged in**
daemon                                     **Never logged in**
adm                                        **Never logged in**
lp                                         **Never logged in**
sync                                       **Never logged in**
shutdown                                   **Never logged in**
halt                                       **Never logged in**
mail                                       **Never logged in**
operator                                   **Never logged in**
games                                      **Never logged in**
ftp                                        **Never logged in**
nobody                                     **Never logged in**
avahi-autoipd                              **Never logged in**
dbus                                       **Never logged in**
polkitd                                    **Never logged in**
rpc                                        **Never logged in**
tss                                        **Never logged in**
rpcuser                                    **Never logged in**
nfsnobody                                  **Never logged in**
postfix                                    **Never logged in**
sshd                                       **Never logged in**
vagrant          pts/0    10.0.2.2         Wed Dec 27 15:07:59 -0500 2017
vboxadd                                    **Never logged in**
apache                                     **Never logged in**
systemd-network                            **Never logged in**
</pre>

* To limit user, this is managed through PAM: `/etc/pam.d`
* `/etc/nologin`. To allow root only logins to the system create the `/etc/nologin` file. This is
automatically generated with the shutdown command when less than 5 minutes is left remaining.

<pre>
[root@web ~]# <b>echo "Server down." > /etc/nologin</b>
</pre>


## Automate System Administration tasks by scheduling jobs

Cron is a Linux service.
We use `cron`, `anacron`, `at`, and `batch` for scheduling tasks
* Recurring tasks:
  * `cron`: down to the minute level
    * Minutes, Hour, Day of Month, Month, Day of week
    * misses jobs if the system is down!
    * Never use `echo` in crons because it doesn't have STDOUT
    * **system crons** are in `/etc/crontab` which we don't edit; but we can use `/etc/cron.d` directory.
      * We also have extension directories for:
        * Cron files: `/etc/cron.d`, `/etc/cron.hourly`,
        * Scripts: `/etc/cron.daily`, `/etc/cron.weekly`, `/etc/cron.monthly`
      * Format is like: `Minute Hour DayOfMonth Month DayOfWeek user command`
    * **User crons** are like system crons, but we exclude the `user` from the format
      * Format is like: `Minute Hour DayOfMonth Month DayOfWeek command`
      * To see your crons you can use `crontab -l` (list)
      * To edit it you can use `crontab -e` (edit) which will open the cron files with a special editor and will load your inserted crons (if they are correct).
      * The files will be saved at `/var/spool/cron/tabs/` or `/var/spool/crontabs`
      * You should never edit this file; but use `crontab -e` instead
  * `anacron`:
    * `/etc/anacrontab`
    * Can run jobs once a day
    * Doesn't miss jobs if the system is down
    * Designed to run jobs after a certain amount of time after system startup.
    * Format: 4 fields
      * `1 15 daily.bk /usr/sbin/backup.sh`
        * `1`: period in days or macro: daily
        * `15`: delay (minutes after system startup for job to run)
        * `daily.bk`: Job identifier used to name timestamp file in `/var/spool/anacron` indicating when job was last run.
        * `/usr/sbin/backup.sh`: command
* Ad-Hoc tasks (One-off type tasks)
  * `at`: runs at specified time and date
    * `at 11 am june 26`
    * to see the jobs on the queue: `atq`
    * To remove entry: `atrm 10`. 10 is the job number from `at -l` or `atq`
  * `batch` job: is going to run when the load average dips below 0.8

**Lab: system crons**

<pre>
[uwe@localhost ~]$ <b>sudo vi /etc/cron.d/list</b>
*/5 * * * 1-5 root ls /etc > /tmp/lsList # */5 means every 5 minutes
[uwe@localhost ~]$ <b>watch ls /tmp</b>
</pre>

**Lab: user crons**

Note that we don't edit this file itself, but use `crontab -e` instead:
<pre>
[uwe@localhost ~]$ <b>cat /etc/crontab</b>
# We can reas these environment settings in when we run our jobs.
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  *  command to be executed
0 5 * * 1 tar -czf /tmp/backup $HOME # Weekly backup on monday morning
</pre>

**Lab: anacron**

You may add your entry to this file
<pre>
[uwe@localhost ~]$ <b>sudo sed '/^#/d' /etc/anacrontab</b>
SHELL=/bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
RANDOM_DELAY=45
START_HOURS_RANGE=3-22

1	5	cron.daily		nice run-parts /etc/cron.daily
7	25	cron.weekly		nice run-parts /etc/cron.weekly
@monthly 45	cron.monthly		nice run-parts /etc/cron.monthly
1	15	daily.bk		tar -czf backup.tar.gz /home
[uwe@localhost ~]$ <b>ls /var/spool/anacron/</b>
cron.daily    cron.monthly  cron.weekly
[uwe@localhost ~]$ <b>sudo anacron</b>
[uwe@localhost ~]$ <b>ls /var/spool/anacron/</b>
cron.daily    cron.monthly  cron.weekly   <b>daily.bk</b>
</pre>

**Lab: at**
<pre>
[uwe@localhost ~]$ at 7:20 pm
at> cp /etc/hosts /tmp
at> ls $(PWD)
at> <EOT> # Ctrl + D
job 1 at Tue Dec 26 19:20:00 2017
</pre>

## Install and Configure X11

![x11](https://user-images.githubusercontent.com/31813625/34367109-6bacfaf4-ea74-11e7-97cb-10b86fe6fa01.png)

<pre>
[uwe@localhost ~]$ <b>xwininfo</b>

xwininfo: Please select the window about which you
          would like information by clicking the
          mouse in that window.

xwininfo: Window id: 0x2400007 "root@localhost:/home/uwe"

  Absolute upper-left X:  315
  Absolute upper-left Y:  100
  Relative upper-left X:  10
  Relative upper-left Y:  46
  Width: 1032
  Height: 511
  Depth: 32
  Visual: 0x4f
  Visual Class: TrueColor
  Border width: 0
  Class: InputOutput
  Colormap: 0x2400006 (not installed)
  Bit Gravity State: NorthWestGravity
  Window Gravity State: NorthWestGravity
  Backing Store State: NotUseful
  Save Under State: no
  Map State: IsViewable
  Override Redirect State: no
  Corners:  +315+100  -19+100  -19-53  +315-53
  -geometry 113x27-9-43
</pre>

<pre>
[uwe@localhost ~]$ <b>xdpyinfo | less</b>
[uwe@localhost ~]$ <b>xhost +</b>
access control <b>disabled</b>, clients can connect from any host
[uwe@localhost ~]$ <b>xhost -</b>
access control <b>enabled</b>, only authorized clients can connect
[uwe@localhost ~]$ <b>xhost</b>
access control enabled, only authorized clients can connect
SI:localuser:uwe
SI:localuser:gnome-initial-setup
SI:localuser:gdm
SI:localuser:root
</pre>

How to disable touchpad?
<pre>
root@localhost:~# <b>man xorg.conf</b>
root@localhost:~# <b>vi /usr/share/X11/xorg.conf.d/70-synaptics.conf</b>
Section "InputClass"
        Identifier "touchpad catchall"
        Driver "synaptics"
        MatchIsTouchpad "on"
      MatchDevicePath "/dev/input/event*"
        <b>Option "Ignore" "on"</b>
EndSection
...
</pre>
### Adding screen resolution

Currently I don't have 1280x720 resolution

![image](https://user-images.githubusercontent.com/31813625/34367281-a0c8e444-ea76-11e7-8ec1-d2dd9f4fd68c.png)

I want to add this resolution:

This adds a transient resolution but needs to be added to `/usr/share/X11/xorg.conf.d` to persist reboots.
1. Calculate our video timings: `cvt 1280 720 60`
2. Create a new mode: `xrandr --newmode <modeline value from cvt>`
3. Query our monitor name: `xrandr -q`
4. Add to that monitor the resolution: `xrandr --addmode <monitor name> 1280x720_60.00`

<pre>
[uwe@localhost ~]$ <b>cvt 1280 720 60</b>
# 1280x720 59.86 Hz (CVT 0.92M9) hsync: 44.77 kHz; pclk: 74.50 MHz
Modeline "1280x720_60.00"   74.50  1280 1344 1472 1664  720 723 728 748 -hsync +vsync
[uwe@localhost ~]$ <b>xrandr --newmode "1280x720_60.00"   74.50  1280 1344 1472 1664  720 723 728 748 -hsync +vsync</b>
[uwe@localhost ~]$ <b>xrandr -q</b>
Screen 0: minimum 64 x 64, current 1366 x 664, maximum 32766 x 32766
VGA-0 connected primary 1366x664+0+0 0mm x 0mm
   1366x664      60.00*+
   2560x1600     60.00
   2560x1440     60.00
   2048x1536     60.00
   1920x1600     60.00
   1920x1080     60.00
   1600x1200     60.00
   1680x1050     60.00
   1400x1050     60.00
   1280x1024     60.00
   1024x768      60.00
   800x600       60.00
   640x480       60.00
  1280x720_60.00 (0x1a5) 74.500MHz -HSync +VSync
        h: width  1280 start 1344 end 1472 total 1664 skew    0 clock  44.77KHz
        v: height  720 start  723 end  728 total  748           clock  59.86Hz
[uwe@localhost ~]$ <b>xrandr --addmode VGA-0 1280x720_60.00</b>
</pre>

Now we have the resolution:

![image](https://user-images.githubusercontent.com/31813625/34367357-c226d35c-ea77-11e7-8f99-16022617045e.png)

### Adding Fonts:
* Download your cont from `font.ubuntu.com`
* System: `/usr/share/fonts`
* Personal: `~/.fonts`
* Then run this command: `fc-cache -fv`

### Set up a display manager

#### LightDM
* was introduced in Debian 7 (Wheezy) and provides a lightweight alternative to GDM.
* `apt install lightdm`
* `dpkg-reconfigure lightdm` (optional)
* `vi /etc/X11/default-display-manager` (alternatively)
* `/etc/lightdm/lightdm.conf`
* `/etc/lightdm/lightdm-gtk-greeter.conf`
* Place your avatar at your home directory with the name of `.face`
* Controlling DMs
  * The lightdm works as a service. You can start, stop, restart,
  or even use `systemctl disable lightdm` to disable it on next boots.

## Localization and Internationalization (i18n)
### Locale
Locale is a set of language and cultural rules
<pre>
[uwe@localhost ~]$ <b>locale</b> # Displays current settings
LANG=en_US.UTF-8
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=
</pre>
* `export LC_ALL='en_GB.UTF-8'`
* `locale -a` lists available locales
* Configurations:
  * Red Hat: `/etc/locale.conf`
      <pre>
      [uwe@localhost ~]$ cat /etc/locale.conf
      LANG="en_US.UTF-8"      </pre>
  * Debian: `etc/default/locale`
    * `dpkg-reconfigure locales`

### iconv
If you needed to convert coding to each other, the command is `iconv`.
The `-l` switch will show you all the available coding:
* `iconv -f WINDOWS-1258 -t UTF-8 /tmp/myfile.txt`

### Working with Timezones
<pre>
[uwe@localhost ~]$ tzselect
Please identify a location so that time zone rules can be set correctly.
Please select a continent or ocean.
 1) Africa
 2) Americas
 3) Antarctica
 4) Arctic Ocean
 5) Asia
 6) Atlantic Ocean
 7) Australia
 8) Europe
 9) Indian Ocean
10) Pacific Ocean
11) none - I want to specify the time zone using the Posix TZ format.

You can make this change permanent for yourself by appending the line
        <b>TZ='America/Toronto'; export TZ</b>
to the file '.profile' in your home directory; then log out and log in again.

Here is that TZ value again, this time on standard output so that you
can use the /usr/bin/tzselect command in shell scripts:
America/Toronto
</pre>

* How to change time zone for all the system?
<pre>
[vagrant@web ~]$ <b>PS1="[\d \t \u@\H:\w ] "</b> # Optional
[<b>Mon Dec 25 04:42:40</b> vagrant@web:~ ] <b>sudo cp -f /usr/share/zoneinfo/Canada/Eastern /etc/localtime</b>
[<b>Sun Dec 24 23:42:57</b> vagrant@web:~ ] <b>date</b>
Sun Dec 24 23:43:00 EST 2017
</pre>

* With systemd we can simplify our management using `timedatectl` and `localectl`
<pre>
[root@web ~]# <b>localectl</b>
   System Locale: LANG=en_US.UTF-8
       VC Keymap: us
      X11 Layout: us
[root@web ~]# <b>localectl set-keymap ca</b>
[root@web ~]# <b>cat /etc/vconsole.conf</b>
KEYMAP=ca
FONT=latarcyrheb-sun16
[root@web ~]# <b>localectl set-locale LANG=en_US.UTF-8</b>
[root@web ~]# <b>cat /etc/locale.conf</b>
LANG="en_US.UTF-8"
</pre>

<pre>
[uwe@localhost ~]$ <b>timedatectl</b>
      Local time: Tue 2017-12-26 20:51:25 EST
  Universal time: Wed 2017-12-27 01:51:25 UTC
        RTC time: Wed 2017-12-27 01:51:24
       Time zone: America/New_York (EST, -0500)
     NTP enabled: yes
NTP synchronized: no
 RTC in local TZ: no
      DST active: no
 Last DST change: DST ended at
                  Sun 2017-11-05 01:59:59 EDT
                  Sun 2017-11-05 01:00:00 EST
 Next DST change: DST begins (the clock jumps one hour forward) at
                  Sun 2018-03-11 01:59:59 EST
                  Sun 2018-03-11 03:00:00 EDT
</pre>

## Maintain System Time
Albert Einstein once said: "The only reaon for time is so everything does not happen at once."

<pre>
[uwe@localhost network-scripts]$ <b>sudo hwclock</b>
Tue 26 Dec 2017 09:32:18 PM EST  -0.181567 seconds
[uwe@localhost network-scripts]$ <b>date</b>
Tue Dec 26 21:33:46 EST 2017
[uwe@localhost network-scripts]$ <b>date +%F</b>
2017-12-26
[uwe@localhost network-scripts]$ <b>date -d "90 days"</b>
Mon Mar 26 22:34:36 EDT 2018
</pre>
* With systemctl: `timedatectl set- <Double Tab>`
### NTP
* `sed -i.$(date +%F) '/^#/d;/^$/d' /etc/ntp.conf`
  * `-i` in place edit creating backup with date extension
  * `'/^#/d;/^$/d'` Deletes commented and blank lines
  * `/etc/ntp.conf` the file
<pre>
hossein@hossein ~ $ <b>sudo sed -i.$(date +%F) '/^#/d;/^$/d' /etc/ntp.conf</b>
hossein@hossein ~ $ <b>cat /etc/ntp.conf</b>
driftfile /var/lib/ntp/ntp.drift
statistics loopstats peerstats clockstats
filegen loopstats file loopstats type day enable
filegen peerstats file peerstats type day enable
filegen clockstats file clockstats type day enable
pool 0.ubuntu.pool.ntp.org iburst
pool 1.ubuntu.pool.ntp.org iburst
pool 2.ubuntu.pool.ntp.org iburst
pool 3.ubuntu.pool.ntp.org iburst
pool ntp.ubuntu.com
restrict -4 default kod notrap nomodify nopeer noquery limited
restrict -6 default kod notrap nomodify nopeer noquery limited
restrict 127.0.0.1
restrict ::1
restrict source notrap nomodify noquery
hossein@hossein ~ $ <b>ll /etc/ntp.conf</b>
ntp.conf             <b>ntp.conf.2017-12-26</b>
</pre>
* `driftfile`: is used to store the frequency offset between the system clock anf the NTP source, or simply, the difference
in the ticks in the local clock to the NTP clock.
* `statistics` Enable statistics collections for named elements
* `filegen` Create files as required for collected statistics, on Debian the directory defaults to `/var/log/ntpstats`
* `restrict` Access Control restriction, by default 127.0.0.1 and ::1 are unrestricted
  * `-4`: IPv4 clients
  * `-6`: IPv6 clients
* `server` Entry for server to synchronize with
* `iburst` make the synchronization a little bit quicker
* `systemctl status ntp.service`

#### NTP tools
<pre>
apt install ntpdate
ntpdate pool.ntp.org
apt install ntp
</pre>
* Insane time: 1000s by defaults
* `ntpdate`: once off adjustment no matter the sanity of the time
* `ntpq`: `ntpd -p` shows peers
* `ntpstat`: shows status but not on Debian. Try `ntpd -c sysinfo`

## System Logging

If a tree falls in a forest and no one is around to hear it, does it make a sound? - George Berkeley

### rsyslog
* `/etc/rsyslog.conf` 2004
  * It is written like this: `facility.priority /pathtofile`
    * facilities: Where from? what is generating the log? auth, user, kern, cron, daemon, mail, user, local1, local2, ...; When a program generated a log, it tags or labels that log with a facility
    * priorities: How bad? Unlike facilities, which have no relationship to each other, priorities are hierarchical. Possible priorities in Linux are (in increasing order of urgency): debug, info, notice, warning, err, crit, alert and emerg. Note that the urgency of a given message is determined by the programmer who wrote it
* Log files will be located in `/var/log/`
  * messages: nearly everything is always a great place to start
  * secure: su and sudo events amongst the other
  * dmesg: kernel ring buffer messages
* Replaced by syslogd (`/etc/syslog.conf`) and syslog-ng
* Version: `rsyslogd -v`
* `systemctl restart rsyslog`

### journalctl
* Part of systemd
  * Responsible of viewing and log management
* Member of adm or wheel to read
  * `usermod -aG wheel username`
* `journalctl -n 50 -p err`: Displays last 50 entries with error priority
* By default only in memory: `/run/log/journal`
* To make it persistent:
  * `mkdir /var/log/journal`
  * `systemctl restart systemd-journald`

### Rotating log files
* Makes us sure that files aren't going too large
* `logrotate /etc/logrotate.conf`
* Usually will be added to `/etc/cron.daily`
* Files are in `/etc/logrotate.d`

**Lab**
1. Open a shell where you log in as user `ryan`. From ryan's shell, type `su` and enter a wrong password
2. Display the messages that were logged due to this event in **journald** as well as in **syslog**
<pre>
[root@rhel ~]# <b>tail -f /var/log/secure</b>
Jan 11 22:40:07 rhel unix_chkpwd[1438]: password check failed for user (ryan)
Jan 11 22:40:07 rhel sudo: pam_unix(sudo-i:auth): authentication failure; logname=root uid=1000 euid=0 tty=/dev/pts/1 ruser=ryan rhost=  user=ryan
[root@rhel ~]# <b>journalctl -f</b>
Jan 11 22:40:07 rhel unix_chkpwd[1438]: password check failed for user (ryan)
Jan 11 22:40:07 rhel sudo[1436]: pam_unix(sudo-i:auth): authentication failure; logname=root uid=1000 euid=0 tty=/dev/pts/1 ruser=ryan rhost
</pre>
3. Configure **logrotate** to keep log files for a month, while keeping the last 6 log files
<pre>
[root@rhel ~]# head -n6 <b>/etc/logrotate.conf</b>
# see "man logrotate" for details
# rotate log files weekly
<b>monthly</b>

# keep 4 weeks worth of backlogs
<b>rotate 6</b></pre>
  * No need to restart anything because it is part of cron job
4. Make sure that the systemd journal is stored persistently
<pre>
[root@rhel ~]# <b>mkdir /var/log/journal</b>
[root@rhel ~]# <b>systemctl restart systemd-journald</b>
[root@rhel ~]# <b>ls /var/log/journal/</b>
06cc202f948c4b3eb22b82cd97b063c3
</pre>

## Mail transfer agent
* MTA: Mail Transfer Agent is the outgoing mail server.
* MTA uses the SMTP which is listening to TCP port 25.
  * `netstat -ltn`
    * `l` Listening
    * `t` TCP
    * `n` Port number rather than service name
* Wen you want to send an email to `uwe@mail.com`:
  1. Send it the local mail server (SMTP)
  2. We define the service-to-port resolution within the `/etc/services` file
  3. We look the MX or mail exchange record for mail.com
  4. We deliver it to the correct email server at mail.com (SMTP)
* TCP port 25 (SMTP):
  * Postfix: More so, we are now movig towards Postfix or in Debian Exim is used
    * Configuration `/etc/postfix/main.cf`
  * Sendmail: Unix traditional mail system
    * `/etc/mail/sendmail.cf`: We don't edit it directly
    * `/etc/mail/sendmail.mc`: We edit this macro file
  * Exim
    * `dpkg-reconfigure exim4-config`

## Manage printers and printing
* Common Unix Printig Service (CUPS)
  * TCP 631
  * Internet Printing Protocol (IPP)
  * Apple Inc.
  * October 1999
  * www.cups.org
  * We can manage through web interface
  * By default only accessible from localhost
  * we can change the configuration `/etc/cups/cupsd.conf`
  * `systemctl restart cups.service`
* Command line tools: `lp`, 'lpr', `lpq`, `lprm`

## Securing data with encryption
* OpenSSH
  * Server configuration `/etc/ssh/sshd_config`
    * Listen address
    * LoginGraceTime
    * PermitRootLogin
    * SysLogFacility
    * MaxSessions
    * ClientAliveInterval
    * ClientAliveMaxCount
  * Server Authentication
    * The public key of the server can be used to authenticate to the client
    * `/etc/ssh/ssh_host_rsa_key.pub`
    * It is down to the client to check the key: StrictHostChecking
    * Server public keys are stored in `/etc/ssh/ssh_known_hosts` or `~/.ssh/known_hosts`
  * Client Configuration and Authentication
    * `/etc/ssh/ssh_config`
    * StrictHostKeyChecking
    * ssh_keygen
    * ssh-copy-id
    * ssh-agent: to cache our password for our private key
    * Client public keys are stored in `~/.ssh/authorized_keys`

**SSH Server Config**
<pre>
[root@web ~]# <b>sed -i.$(date +%F) '/^#/d;/^$/d' /etc/ssh/sshd_config</b>
[root@web ~]# <b>cat /etc/ssh/sshd_config</b>
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key
SyslogFacility AUTHPRIV
AuthorizedKeysFile	.ssh/authorized_keys
PasswordAuthentication yes
ChallengeResponseAuthentication no
GSSAPIAuthentication yes
GSSAPICleanupCredentials no
UsePAM yes
X11Forwarding yes
UsePrivilegeSeparation sandbox		# Default for new installations.
AcceptEnv LANG LC_CTYPE LC_NUMERIC LC_TIME LC_COLLATE LC_MONETARY LC_MESSAGES
AcceptEnv LC_PAPER LC_NAME LC_ADDRESS LC_TELEPHONE LC_MEASUREMENT
AcceptEnv LC_IDENTIFICATION LC_ALL LANGUAGE
AcceptEnv XMODIFIERS
Subsystem	sftp	/usr/libexec/openssh/sftp-server
UseDNS no
GSSAPIAuthentication no
</pre>

**SSH Troubleshooting scenario**
How SSH works:
1. Client is issuing command `ssh server_address`
2. It will connects to sshd service on the server
3. SSH service sends back it's public key (`/etc/ssh/ssh_host.pub`) to the client
4. Client asks you if you want to continue because you are going to connect to the server for the first time
5. If you answer yes then the public key of the server is written to `~/.ssh/known_hosts` of the client
6. If the user doesn't want to enter the password each time connecting to the server, he/she can generate his/her key pair using `ssh-keygen` command and copy his/her public key `~/.ssh/id_rsa.pub` to the his/her account on the server as an authorized key `~/.ssh/authorized_keys`

In troubleshooting, `ssh-keygen` and `~/.ssh/id_rsa.pub` on the client
and copying the public key from the client to `ls ~/.ssh/authorized_keys` on the server
came handy for me.

For troubleshooting if your VMs cloned, they have the same ssh_host_* keys.
* `sudo rm /etc/ssh/ssh_host_*` OR.
* `sudo dpkg-reconfigure openssh-server` If you are using Debian-based systems.
* `ssh-keygen -R 192.168.33.x` On the SSH client machine.

### Implementing SSH Tunnels

* HTTP server IP is: 192.168.33.20
* Client is the `ac` with IP address 192.168.33.10

<pre>
<b>vagrant@ac:~$ curl -I -L 192.168.33.20</b>
HTTP/1.1 403 Forbidden
Date: Thu, 28 Dec 2017 00:33:36 GMT
Server: Apache/2.4.6 (CentOS)
Last-Modified: Thu, 16 Oct 2014 13:20:58 GMT
ETag: "1321-5058a1e728280"
Accept-Ranges: bytes
Content-Length: 4897
Content-Type: text/html; charset=UTF-8
</pre>

Now, let's run the SSH tunnel as HTTP is not secure.
<pre>
<b>vagrant@ac:~$ ssh -f -N -L 8880:localhost:80 192.168.33.20</b>
vagrant@192.168.33.20's password:
vagrant@ac:~$ netstat -ltn
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN
<b>tcp        0      0 127.0.0.1:8880          0.0.0.0:*               LISTEN</b>
tcp        0      0 0.0.0.0:41424           0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN
tcp6       0      0 :::36744                :::*                    LISTEN
tcp6       0      0 :::111                  :::*                    LISTEN
<b>tcp6       0      0 ::1:8880                :::*                    LISTEN</b>
tcp6       0      0 :::22                   :::*                    LISTEN
tcp6       0      0 ::1:25                  :::*                    LISTEN
vagrant@ac:~$ <b>ps -aux | grep "ssh -f"</b>
vagrant   <b>1312</b>  0.0  0.5  46504  2840 ?        Ss   00:26   0:00 ssh -f -N -L 8880:localhost:80 192.168.33.20
</pre>
Now, let's test:
<pre>
<b>vagrant@ac:~$ curl -I -L localhost:8880</b>
HTTP/1.1 403 Forbidden
Date: Thu, 28 Dec 2017 00:36:38 GMT
Server: Apache/2.4.6 (CentOS)
Last-Modified: Thu, 16 Oct 2014 13:20:58 GMT
ETag: "1321-5058a1e728280"
Accept-Ranges: bytes
Content-Length: 4897
Content-Type: text/html; charset=UTF-8
</pre>
Let's bring down the tunnel:
<pre>
<b>vagrant@ac:~$ kill 1312</b>
<b>vagrant@ac:~$ curl -I -L localhost:8880</b>
curl: (7) Failed to connect to localhost port 8880: Connection refused
</pre>