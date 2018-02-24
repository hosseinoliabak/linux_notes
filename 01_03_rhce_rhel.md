# Red Hat Certified Engineer (RHCE) tutorials

## System authentication

### Red Hat Identity Management (IdM)
* This feature set is available free with Red Hat Enterprise Linux subscription.
* FreeIPA is the upstream open source project behind it (`ipa-client-install`)
* Many features are affected:
  * LDAP server
  * Kerberos Key Distribution Center
  * Certificate Server
  * Integrated DNS
  * Time server
* Windows integration is available as well, through the `realmd` service, or by using Samba 4 components.

### Using authconfig

The `authconfig` tool can configure the system to use specific services — SSSD, LDAP, NIS,
or Winbind — for its user database, along with using different forms of authentication mechanisms.

* Important: To configure Identity Management systems, Red Hat recommends using the
`ipa-client-install` utility or the `realmd` system instead of `authconfig`. The authconfig utilities
are limited and substantially less flexible.

`authconfig` writes its configuration into `/etc/sssd/sssd.conf` (SSSD, LDAP, NIS, Winbind).

To install be sure that dependencies are also installed. `yum groups install "Directory Client"`

for more information, see the 01_02_rhel_tutorials *Connecting the client to an LDAP server* section before continuing further.

<pre>
[root@client1 ~]# cat /etc/sssd/sssd.conf 
[domain/default]

autofs_provider = ldap
cache_credentials = True
<b>ldap_search_base = dc=example,dc=com</b>
id_provider = ldap
auth_provider = ldap
chpass_provider = ldap
<b>ldap_uri = ldap://serveripa.example.com</b>
ldap_id_use_start_tls = True
<b>ldap_tls_cacertdir = /etc/openldap/cacerts</b>
[sssd]
services = nss, pam, autofs

domains = default
[nss]
homedir_substring = /home

[pam]

[sudo]

[autofs]

[ssh]

[pac]

[ifp]

[secrets]
</pre>

## Secure Applications
* Using Pluggable Authentication Modules (PAM)
* Using Kerberos
* Working with certmonger
* Single Sign-On

### Kerberos
* Kerberos uses symmetric-key cryptography and a trusted third party (a key distribution center or KDC)
to authenticate users to network services which means passwords are never sent over the network.
1. User (principal) authenticates to the KDC asking for ticket to the session.
2. KDC checks for principal. KDC has two important services
  * Authentication Service (AS) 
  * Ticket-Granting Service (TGS) 
3. KDC creates TGT (ticket-granting ticket) from the authentication server and wraps it to the user's key. Then 
KDC sends the TGT (a set of credentials) specific to that session back to the user.
  * TGT usually expires after 10 to 24 hours
4. User decrypts the TGT (using login or `kinit` program using his key (password)) and stores it *ccache*.
  * The user's key is used only on the client machine and is not transmitted over the network. 
5. `ccache` can be checked by Kerberos-aware services when user asks for a specific service from
the ticket-granting server (TGS). The following types of *ccache* are supported by RHEL 7
  * KEYRING (default)
  * FILE
  * DIR
  * MEMORY
6. Ticket then decrypted and verified by service *keytab*

* Configuration in `/etc/krb5.conf`
* `klist` command shows current Kerberos credential tickets
* `klist -k` command shows current credentials from the *keytab* file
* `kinit` allows users to start a Kerberos session

**Lab**
First copy the certificate into client's `/etc/openldap/cacerts/`. You may download it using 
`wget ftp://serveripa.example.com/pub/ca.crt`
```
[root@client1 ~]# ls /etc/openldap/cacerts/
45e037a3.0  authconfig_downloaded.pem


                                      ┌────────────────┤ Authentication Configuration ├─────────────────┐
                                      │                                                                 │ 
                                      │  User Information        Authentication                         │ 
                                      │  [ ] Cache Information   [ ] Use MD5 Passwords                  │ 
                                      │  [*] Use LDAP            [*] Use Shadow Passwords               │ 
                                      │  [ ] Use NIS             [*] Use LDAP Authentication            │ 
                                      │  [ ] Use IPAv2           [*] Use Kerberos                       │ 
                                      │  [ ] Use Winbind         [*] Use Fingerprint reader             │ 
                                      │                          [ ] Use Winbind Authentication         │ 
                                      │                          [*] Local authorization is sufficient  │ 
                                      │                                                                 │ 
                                      │            ┌────────┐                      ┌──────┐             │ 
                                      │            │ Cancel │                      │ Next │             │ 
                                      │            └────────┘                      └──────┘             │ 
                                      │                                                                 │ 
                                      │                                                                 │ 
                                      └─────────────────────────────────────────────────────────────────┘ 
                                                                                                          

                                             ┌─────────────────┤ LDAP Settings ├─────────────────┐
                                             │                                                   │ 
                                             │          [*] Use TLS                              │ 
                                             │  Server: ldap://serveripa.example.com/___________ │ 
                                             │ Base DN: dc=example,dc=com_______________________ │ 
                                             │                                                   │ 
                                             │         ┌──────┐                ┌──────┐          │ 
                                             │         │ Back │                │ Next │          │ 
                                             │         └──────┘                └──────┘          │ 
                                             │                                                   │ 
                                             │                                                   │ 
                                             └───────────────────────────────────────────────────┘ 
                                                                                                   


                                           ┌─────────────────┤ Kerberos Settings ├──────────────────┐
                                           │                                                        │ 
                                           │        Realm: ________________________________________ │ 
                                           │          KDC: ________________________________________ │ 
                                           │ Admin Server: ________________________________________ │ 
                                           │               [*] Use DNS to resolve hosts to realms   │ 
                                           │               [*] Use DNS to locate KDCs for realms    │ 
                                           │                                                        │ 
                                           │          ┌──────┐                    ┌────┐            │ 
                                           │          │ Back │                    │ Ok │            │ 
                                           │          └──────┘                    └────┘            │ 
                                           │                                                        │ 
                                           │                                                        │ 
                                           └────────────────────────────────────────────────────────┘ 
                                                                                                      

[root@client1 ~]# su - ldapuser1
Last login: Wed Feb 14 22:40:45 EST 2018 on pts/0
su: warning: cannot change directory to /home/ldap/ldapuser1: No such file or directory
-sh-4.2$ pwd
/root
```

### Connecting to an IPA Server
<pre>
[root@client1 ~]# <b>yum install ipa-client</b>
[root@client1 ~]# <b>ipa-client-install</b> 
WARNING: ntpd time&date synchronization service will not be configured as
conflicting service (chronyd) is enabled
Use --force-ntpd option to disable it and force configuration of ntpd

Discovery was successful!
Client hostname: client1.example.com
Realm: EXAMPLE.COM
DNS Domain: example.com
IPA Server: serveripa.example.com
BaseDN: dc=example,dc=com

Continue to configure the system with these values? [no]: <b>yes</b>
Skipping synchronizing time with NTP server.
User authorized to enroll computers: <b>admin</b>
Password for admin@EXAMPLE.COM: <b>password</b>
Successfully retrieved CA cert
    Subject:     CN=Certificate Authority,O=EXAMPLE.COM
    Issuer:      CN=Certificate Authority,O=EXAMPLE.COM
    Valid From:  2018-02-15 01:20:39
    Valid Until: 2038-02-15 01:20:39
...
The ipa-client-install command was successful
[root@station1 ~]# <b>su - ldapuser1</b>
su: warning: cannot change directory to /home/ldap/ldapuser1: No such file or directory
-sh-4.2$
</pre>

## Configuring iSCSI
* Traditional SCSI uses a long cable and the SCSI Command Descriptor Block (CDB) to attach the to SCSI.
* iSCSI (SCSI over IP) uses the same CDB command, but encapsulated in IP packets over a network.
* This means that you can have local SCSI devices (like `/dev/sdb`) without having the storage hardware in the local computer.
* SCSI devices are emulated by using a storage backend and presenting it on the network using iSCSI targets.
* Ethernet infrastructure can be as fast as FC structure, which makes iSCSI an enterprise-ready choice for SAN.
  
  ![iscsi](https://user-images.githubusercontent.com/31813625/36345942-b1dcc754-1402-11e8-898e-7597b13d7c2b.png)
  
* The *target* on the SAN gives access to storage backends:
  * Disks
  * Partitions
  * LVMs
  * Files
* Servers connect to these using the iSCSI Initiator
* After successful connection, the connected server will see newly added storage devices
  * Check in `/proc/partitions`
  * Or use `lsscsi`
* iSCSI Terminology
  * IQN: The iSCSI qualified name
  * Initiator: the iSCSI client that is identified by an IQN
  * Target: The service on an iSCSI server that gives access to the background storage devices
  * ACL: an Access Control List is based on node IQDN's
  * Portal: The IP address and port that a target or initiator uses to establish connections; also 
  referred to as node
  * Discovery: The process where an initiator finds the targets that are configured on a portal
  * Logical Unit Numbers (LUN): Used to map to storage allocated in a SAN. Shared through the target
  * Login: Authentication that gives an initiator access to LUNs
  * TPG: Target Portal Group, the collection of IP addresses and TCP ports which a specific iSCSI
  target will listen to.
  * Fiber Channel (FC) (to move data):
    * Host Bus Adapter (HBA): you connect to the computer/server, you still have the NIC They don’t
    run Ethernet, they run Fiber Channel.
    * Computer/server adapter 
    * SFP or QSFP ports 
    * SANs are accessed through HBA 
    
    ![image](https://user-images.githubusercontent.com/31813625/36346322-8542cd4e-140a-11e8-9344-ebffaae87bf1.png)
    
<pre>
[root@target ~]# <b>lsblk</b>
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
<b>sda               8:0    0    5G  0 disk</b> 
sdb               8:16   0    6G  0 disk 
sr0              11:0    1 1024M  0 rom  
vda             252:0    0    9G  0 disk 
├─vda1          252:1    0    1G  0 part /boot
└─vda2          252:2    0    8G  0 part 
  ├─centos-root 253:0    0  7.1G  0 lvm  /
  └─centos-swap 253:1    0  924M  0 lvm  [SWAP]
</pre>
Let's go and share `sda`
<pre>
[root@target ~]# <b>yum install targetcli -y</b> # Supports a very good tab compeletion
[root@target ~]# <b>targetcli</b> 
Warning: Could not load preferences file /root/.targetcli/prefs.bin.
targetcli shell version 2.1.fb46
Copyright 2011-2013 by Datera, Inc and others.
For help on commands, type 'help'.

/> 
/> <b>ls</b>
o- / ......................................................................................................................... [...]
  o- backstores .............................................................................................................. [...]
  | o- block .................................................................................................. [Storage Objects: 0]
  | o- fileio ................................................................................................. [Storage Objects: 0]
  | o- pscsi .................................................................................................. [Storage Objects: 0]
  | o- ramdisk ................................................................................................ [Storage Objects: 0]
  o- iscsi ............................................................................................................ [Targets: 0]
  o- loopback ......................................................................................................... [Targets: 0]
/> <b>cd /backstores/block</b> 
/backstores/block> 
/backstores/block> <b>create dev=/dev/sda name=sda</b>
Created block storage object sda using /dev/sda.
/backstores> <b>cd /</b>
/> <b>cd iscsi</b> 
/iscsi> <b>create wwn=iqn.2018-02.com.example:randomnumber</b> 
Created target iqn.2018-02.com.example:randomnumber.
Created TPG 1.
Global pref auto_add_default_portal=true
Created default portal listening on all IPs (0.0.0.0), port 3260.
/iscsi> ls
o- iscsi .............................................................................................................. [Targets: 1]
  o- iqn.2018-02.com.example:randomnumber ................................................................................ [TPGs: 1]
    <b>o- tpg1 ................................................................................................. [no-gen-acls, no-auth]</b>
      o- acls ............................................................................................................ [ACLs: 0]
      o- luns ............................................................................................................ [LUNs: 0]
      o- portals ...................................................................................................... [Portals: 1]
        o- 0.0.0.0:3260 ....................................................................................................... [OK]
/iscsi> <b><b>cd iqn.2018-02.com.example:randomnumber/tpg1/luns</b></b> 
/iscsi/iqn.20...ber/tpg1/luns> create /backstores/block/sda 
Created LUN 0.
</pre>
Now, Let's copy the IQN of the initiator, we want it for our ACL configuration
<pre>
[root@target ~]# <b>ssh 192.168.4.5</b>
[root@station1 ~]# <b>yum install iscsi-initiator-utils</b>
[root@client1 ~]# <b>cd /etc/iscsi/</b>
[root@client1 iscsi]# <b>ls</b>
initiatorname.iscsi  iscsid.conf
[root@client1 iscsi]# <b>cat initiatorname.iscsi</b> 
InitiatorName=iqn.1994-05.com.redhat:55fdbbb7a09e
</pre>
Now, let's back to our target
<pre>
[root@client1 iscsi]# <b>exit</b>
logout
Connection to 192.168.4.5 closed.
[root@target ~]# targetcli 
targetcli shell version 2.1.fb46
Copyright 2011-2013 by Datera, Inc and others.
For help on commands, type 'help'.

/iscsi/iqn.20...ber/tpg1/luns> <b>cd ../acls</b> 
/iscsi/iqn.20...ber/tpg1/acls> <b>create wwn=iqn.1994-05.com.redhat:55fdbbb7a09e</b>
Created Node ACL for iqn.1994-05.com.redhat:55fdbbb7a09e
Created mapped LUN 0.
/iscsi/iqn.20...ber/tpg1/acls> <b>cd /</b>
/> <b>ls</b>
o- / ......................................................................................................................... [...]
  o- backstores .............................................................................................................. [...]
  | o- block .................................................................................................. [Storage Objects: 1]
  | | o- sda .............................................................................. [/dev/sda (5.0GiB) write-thru activated]
  | |   o- alua ................................................................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ....................................................................... [ALUA state: Active/optimized]
  | o- fileio ................................................................................................. [Storage Objects: 0]
  | o- pscsi .................................................................................................. [Storage Objects: 0]
  | o- ramdisk ................................................................................................ [Storage Objects: 0]
  o- iscsi ............................................................................................................ [Targets: 1]
  | o- iqn.2018-02.com.example:randomnumber .............................................................................. [TPGs: 1]
  |   o- tpg1 ............................................................................................... [no-gen-acls, no-auth]
  |     o- acls .......................................................................................................... [ACLs: 1]
  |     | o- iqn.1994-05.com.redhat:55fdbbb7a09e .................................................................. [Mapped LUNs: 1]
  |     |   o- mapped_lun0 ................................................................................... [lun0 block/sda (rw)]
  |     o- luns .......................................................................................................... [LUNs: 1]
  |     | o- lun0 ........................................................................ [block/sda (/dev/sda) (default_tg_pt_gp)]
  |     o- portals .................................................................................................... [Portals: 1]
  |       o- 0.0.0.0:3260 ..................................................................................................... [OK]
  o- loopback ......................................................................................................... [Targets: 0]
/> exit
Global pref auto_save_on_exit=true
Last 10 configs saved in /etc/target/backup.
Configuration saved to /etc/target/saveconfig.json
[root@target ~]# <b>systemctl enable target</b>
Created symlink from /etc/systemd/system/multi-user.target.wants/target.service to /usr/lib/systemd/system/target.service.
[root@target ~]# <b>systemctl start target</b>
[root@target ~]# <b>firewall-cmd --add-port=3260/tcp --permanent</b>
success
[root@target ~]# <b>firewall-cmd --reload</b>
success

</pre>
Now, time to configure the initiator. 
Let's start seeing what the current configuration looks like.
<pre>
[root@client1 ~]# <b>lsscsi</b>
[0:0:0:0]    cd/dvd  QEMU     QEMU DVD-ROM     2.5+  /dev/sr0 
[2:0:0:0]    disk    QEMU     QEMU HARDDISK    2.5+  /dev/sda
[root@client1 ~]# <b>lsblk</b>
NAME           MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda              8:0    0  2.1G  0 disk 
sr0             11:0    1 1024M  0 rom  
vda            252:0    0    7G  0 disk 
├─vda1         252:1    0    1G  0 part /boot
└─vda2         252:2    0    6G  0 part 
  ├─myvg1-root 253:0    0  5.3G  0 lvm  /
  └─myvg1-swap 253:1    0  716M  0 lvm  [SWAP]
</pre>
Let's continue by copying and pasting from great man page of `iscsiadm` tool
<pre>
[root@client1 ~]# <b>man iscsiadm | grep EXAMPLES -A20</b>
EXAMPLES
       Discover targets at a given IP address:

            iscsiadm --mode discoverydb --type sendtargets --portal 192.168.1.10 --discover

       Login, must use a node record id found by the discovery:

            iscsiadm --mode node --targetname iqn.2001-05.com.doe:test --portal 192.168.1.1:3260 --login

       Logout:

            iscsiadm --mode node --targetname iqn.2001-05.com.doe:test --portal 192.168.1.1:3260 --logout

       List node records:

            iscsiadm --mode node


       Display all data for a given node record:

            iscsiadm --mode node --targetname iqn.2001-05.com.doe:test --portal 192.168.1.1:3260
[root@client1 ~]# <b>iscsiadm --mode discoverydb --type sendtargets --portal 192.168.4.4 --discover</b>
192.168.4.4:3260,1 iqn.2018-02.com.example:randomnumber
[root@client1 ~]# <b>iscsiadm --mode node --targetname iqn.2018-02.com.example:randomnumber --portal 192.168.4.4:3260 --login</b>
Logging in to [iface: default, target: iqn.2018-02.com.example:randomnumber, portal: 192.168.4.4,3260] (multiple)
Login to [iface: default, target: iqn.2018-02.com.example:randomnumber, portal: 192.168.4.4,3260] successful.
[root@client1 ~]# <b>systemctl status iscsid</b>
● iscsid.service - Open-iSCSI
   Loaded: loaded (/usr/lib/systemd/system/iscsid.service; disabled; vendor preset: disabled)
   Active: active (running) since Sat 2018-02-17 19:04:19 EST; 35s ago
[root@client1 ~]# <b>lsscsi</b>
[0:0:0:0]    cd/dvd  QEMU     QEMU DVD-ROM     2.5+  /dev/sr0 
[2:0:0:0]    disk    QEMU     QEMU HARDDISK    2.5+  /dev/sda 
<b>[3:0:0:0]    disk    LIO-ORG  sda              4.0   /dev/sdb</b> 
[root@client1 ~]# <b>ls /var/lib/iscsi/nodes/</b>
iqn.2018-02.com.example:randomnumber
[root@client1 ~]# <b>cd /var/lib/iscsi/nodes/iqn.2018-02.com.example\:randomnumber/192.168.4.4\,3260\,1/</b>
[root@client1 192.168.4.4,3260,1]# <b>cat default</b> # The parameters by which you can customize your iscsi initiator
</pre>
Verification commands on the initiator:
<pre>
[root@client1 ~]# <b>lsblk</b>
NAME           MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda              8:0    0  2.1G  0 disk 
<b>sdb              8:16   0    5G  0 disk</b> 
sr0             11:0    1 1024M  0 rom  
vda            252:0    0    7G  0 disk 
├─vda1         252:1    0    1G  0 part /boot
└─vda2         252:2    0    6G  0 part 
  ├─myvg1-root 253:0    0  5.3G  0 lvm  /
  └─myvg1-swap 253:1    0  716M  0 lvm  [SWAP]
[root@client1 ~]# <b>iscsiadm -m discovery -P 0</b>
192.168.4.4:3260 via sendtargets
[root@client1 ~]# <b>iscsiadm -m discovery -P 1</b>
SENDTARGETS:
DiscoveryAddress: 192.168.4.4,3260
Target: iqn.2018-02.com.example:randomnumber
	Portal: 192.168.4.4:3260,1
		Iface Name: default
iSNS:
No targets found.
STATIC:
No targets found.
FIRMWARE:
No targets found.
[root@client1 ~]# <b>iscsiadm -m session -P 0</b>
tcp: [1] 192.168.4.4:3260,1 iqn.2018-02.com.example:randomnumber (non-flash)
[root@client1 ~]# <b>iscsiadm -m session -P 1</b>
Target: iqn.2018-02.com.example:randomnumber (non-flash)
	Current Portal: 192.168.4.4:3260,1
	Persistent Portal: 192.168.4.4:3260,1
		**********
		Interface:
		**********
		Iface Name: default
		Iface Transport: tcp
		Iface Initiatorname: iqn.1994-05.com.redhat:55fdbbb7a09e
		Iface IPaddress: 192.168.4.5
		Iface HWaddress: <empty>
		Iface Netdev: <empty>
		SID: 1
		iSCSI Connection State: LOGGED IN
		iSCSI Session State: LOGGED_IN
		Internal iscsid Session State: NO CHANGE
</pre>
In exam, if you want to delete your iSCSI, you may run this command
<pre>
[root@client1 ~]# <b>rm -rf /var/lib/iscsi/nodes/*</b></pre>

#### Mounting an LVM volume on top of the iSCSI Drive
<pre>
[root@client1 ~]# <b>pvcreate /dev/sdb</b>
  Physical volume "/dev/sdb" successfully created.
[root@client1 ~]# <b>vgcreate vg_san /dev/sdb</b>
  Volume group "vg_san" successfully created
[root@client1 ~]# <b>lvcreate vg_san -l 100%FREE -n lv_san</b>
  Logical volume "lv_san" created.
[root@client1 ~]# <b>mkfs.ext4 /dev/vg_san/lv_san</b> 
mke2fs 1.42.9 (28-Dec-2013)
...                       
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done 
[root@client1 ~]# <b>mkdir /san</b>
[root@client1 ~]# <b>tail -n1 /etc/fstab</b> 
/dev/vg_san/lv_san 	/san			ext4	_netdev		0 0 
[root@client1 ~]# <b>mount -a</b>
[root@client1 ~]# <b>reboot</b></pre>

## System Performance Reporting
* Typical performance focus area
  * Memory: Very important factor in overall server performance
  * Disk: Very important in overall server performance
  * Network: In general not the most important source of problems
  * CPU: Many tunables, but in general not very important
* Tools:
  * top: Excellent generic overview
  * iostat
    * iotop: information about I/O; better than iostat
  * vmstat: Virtual Memory Statistics (This is not swap)
  * sar: System Activity Reporter

### vmstat
<pre>
[root@client1 ~]# <b>vmstat 2 5</b>
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 3  0 112988  67752      0 273288   12  174  4655   614  315  560 10  3 86  1  0
 0  0 112988  67488      0 273288    0    0     0    32   70   89  0  1 99  0  0
 0  0 112988  67488      0 273288    0    0     0     0   49   52  0  0 100  0  0
 0  0 112988  67488      0 273288    0    0     0     0   52   54  0  0 100  0  0
 0  0 112988  67488      0 273288    0    0     0     0   64   70  0  0 100  0  0
</pre>
* `2` means every 2 seconds
* `5` means 5 times
* `s` means swap
* `b` means block
* `i` means in
* `o` means out

<pre>
[root@client1 ~]# <b>vmstat -sS M</b>
          992 M total memory
          659 M used memory
          363 M active memory
          416 M inactive memory
           64 M free memory
            0 M buffer memory
          267 M swap cache
          715 M total swap
          110 M used swap
          605 M free swap
         6573 non-nice user cpu ticks
          187 nice user cpu ticks
         1790 system cpu ticks
        85687 idle cpu ticks
          491 IO-wait cpu ticks
            0 IRQ cpu ticks
           32 softirq cpu ticks
           28 stolen cpu ticks
      3062032 pages paged in
       403812 pages paged out
         1996 pages swapped in
        28563 pages swapped out
       212828 interrupts
       374548 CPU context switches
   1518664893 boot time
         3451 forks
</pre>

<pre>
[root@client1 ~]# <b>less /proc/meminfo | grep ctive</b>
Active:           376132 kB
Inactive:         427704 kB
Active(anon):     272880 kB
Inactive(anon):   326856 kB
Active(file):     103252 kB
Inactive(file):   100848 kB
</pre>

### System Active Reporter (sar)
* `sar` comes from sysstat package and collects data every 10 minutes
* To make sorting and finding data in `sar` easy, set `LANG=C` before starting `sar`
  * `echo alias sar='LANG=C sar' >> /etc/bashrc`
* `sar` data is collected through cron jobs `/etc/cron.d/sysstat`

    <pre>
    [root@client1 ~]# <b>cat /etc/cron.d/sysstat</b>
    # Run system activity accounting tool every 10 minutes
    */10 * * * * root /usr/lib64/sa/sa1 1 1
    # Generate a daily summary of process accounting at 23:53
    53 23 * * * root /usr/lib64/sa/sa2 -A
    </pre>

* Data is written to `/var/log/sa` directory
* Default reports are kept 28 days
    <pre>
    [root@client1 ~]# <b>cat /etc/sysconfig/sysstat</b>
    # sysstat-10.1.5 configuration file.
    
    # How long to keep log files (in days).
    # If value is greater than 28, then log files are kept in
    # multiple directories, one for each month.
    HISTORY=28
    
    # Compress (using gzip or bzip2) sa and sar files older than (in days):
    COMPRESSAFTER=31
    
    # Parameters for the system activity data collector (see sadc manual page)
    # which are used for the generation of log files.
    SADC_OPTIONS="-S DISK"
    
    # Compression program to use.
    ZIP="bzip2"
    </pre>

<pre>
[root@serveripa ~]# <b>LANG=C sar | less</b>
Linux 3.10.0-693.el7.x86_64 (serveripa.example.com)     02/21/18        _x86_64_        (2 CPU)

06:18:22          LINUX RESTART

06:20:01        CPU     %user     %nice   %system   %iowait    %steal     %idle
06:30:01        all      0.07      0.00      0.03      0.02      0.04     99.84
06:40:01        all      0.04      0.00      0.03      0.01      0.04     99.88
06:50:01        all      0.03      0.00      0.03      0.01      0.04     99.89
07:00:01        all      0.03      0.00      0.02      0.01      0.04     99.90
07:10:01        all      0.04      0.00      0.03      0.01      0.04     99.88
07:20:01        all      0.03      0.00      0.02      0.01      0.03     99.90
...
[root@serveripa ~]# <b>LANG=C sar -P 0 | less</b>
Linux 3.10.0-693.el7.x86_64 (serveripa.example.com)     02/21/18        _x86_64_        (2 CPU)

06:18:22          LINUX RESTART

06:20:01        CPU     %user     %nice   %system   %iowait    %steal     %idle
06:30:01          0      0.04      0.00      0.03      0.03      0.03     99.86
06:40:01          0      0.03      0.00      0.03      0.02      0.05     99.88
06:50:01          0      0.02      0.00      0.02      0.02      0.04     99.91
07:00:01          0      0.02      0.00      0.03      0.01      0.04     99.89
07:10:01          0      0.04      0.00      0.03      0.02      0.04     99.87
07:20:01          0      0.04      0.00      0.03      0.02      0.03     99.90
...
[root@serveripa ~]# <b>LANG=C sar -n DEV</b> # Giving us network informaion
Linux 3.10.0-693.el7.x86_64 (serveripa.example.com)     02/21/18        _x86_64_        (2 CPU)

06:18:22          LINUX RESTART

06:20:01        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
06:30:01         eth0      0.61      0.21      0.03      0.02      0.00      0.00      0.00
06:30:01           lo      0.26      0.26      0.03      0.03      0.00      0.00      0.00
06:30:01    virbr0-nic      0.00      0.00      0.00      0.00      0.00      0.00      0.00
...
</pre>

## System Optimization Basics
* Through `/proc/sys`
* Or using `sysctl`
  * `/etc/sysctl.conf`
  * `/usr/lib/sysctl.d`
<pre>
[root@client1 ~]# <b>cat /proc/sys/vm/swappiness</b> 
30
[root@client1 ~]# <b>echo 80 > /proc/sys/vm/swappiness</b>
</pre>
For another example, you tune `/proc/sys/vm/nr_hugepages`

<pre>
[root@client1 ~]# <b>sysctl -a | wc -l</b>
988
[root@client1 ~]# <b>sysctl -a | tail -n18</b>
<b>vm.nr_hugepages = 0</b>
vm.nr_hugepages_mempolicy = 0
vm.nr_overcommit_hugepages = 0
vm.nr_pdflush_threads = 0
vm.numa_zonelist_order = default
vm.oom_dump_tasks = 1
vm.oom_kill_allocating_task = 0
vm.overcommit_kbytes = 0
vm.overcommit_memory = 0
vm.overcommit_ratio = 50
vm.page-cluster = 3
vm.panic_on_oom = 0
vm.percpu_pagelist_fraction = 0
vm.stat_interval = 1
<b>vm.swappiness = 80</b>
vm.user_reserve_kbytes = 29135
vm.vfs_cache_pressure = 100
vm.zone_reclaim_mode = 0
[root@client1 ~]# <b>sysctl -w vm.nr_hugepages=60</b>
vm.nr_hugepages = 60
</pre>
So far, both methods above were not persistent. Let's do it persistently
<pre>
[root@client1 ~]# <b>vim /etc/sysctl.conf</b> 
[root@client1 ~]# <b>sysctl -p</b>
vm.swappiness = 60
</pre>
**Lab**
* Disable IPv6
* Disable `icmp_echo_ignore_all`
* Disable `icmp_echo_ignore_broadcast`
* Disable routing: `ip_forward`
* How to disable CPU 7?
  * `echo 0 > /sys/devices/system/cpu/cpu7/online`
  
### tuned
* Like sdm template in Cisco switches for RHEL servers
<pre>
[root@client1 ~]# <b>systemctl status tuned</b>
● tuned.service - Dynamic System Tuning Daemon
   Loaded: loaded (/usr/lib/systemd/system/tuned.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2018-02-22 05:35:26 EST; 2h 11min ago
 Main PID: 1066 (tuned)
   CGroup: /system.slice/tuned.service
           └─1066 /usr/bin/python -Es /usr/sbin/tuned -l -P
[root@client1 ~]# <b>tuned-adm list</b>
Available profiles:
- balanced                    - General non-specialized tuned profile
- desktop                     - Optimize for the desktop use-case
- latency-performance         - Optimize for deterministic performance at the cost of increased power consumption
- network-latency             - Optimize for deterministic performance at the cost of increased power consumption, focused on low latency network performance
- network-throughput          - Optimize for streaming network throughput, generally only necessary on older CPUs or 40G+ networks
- powersave                   - Optimize for low power consumption
- throughput-performance      - Broadly applicable tuning that provides excellent performance across a variety of common server workloads
- virtual-guest               - Optimize for running inside a virtual guest
- virtual-host                - Optimize for running KVM guests
Current active profile: <b>virtual-guest</b>
[root@client1 ~]# <b>cat /usr/lib/tuned/virtual-guest/tuned.conf</b> 
#
# tuned configuration
#

[main]
summary=Optimize for running inside a virtual guest
include=throughput-performance

[sysctl]
# If a workload mostly uses anonymous memory and it hits this limit, the entire
# working set is buffered for I/O, and any more write buffering would require
# swapping, so it's time to throttle writes until I/O can catch up.  Workloads
# that mostly use file mappings may be able to use even higher values.
#
# The generator of dirty data starts writeback at this percentage (system default
# is 20%)
vm.dirty_ratio = 30

# Filesystem I/O is usually much more efficient than swapping, so try to keep
# swapping low.  It's usually safe to go even lower than this on systems with
# server-grade storage.
vm.swappiness = 30
</pre>

## Networking
* `man nmcli-examples`: see example 9

### Routing
* Static route
  * First, check if routing is enabled or not. If not, we enable it.
    <pre>
    [root@client1 ~]# <b>cat /proc/sys/net/ipv4/ip_forward</b>
    0
    [root@client1 ~]# <b>vi /etc/sysctl.conf</b>
    [root@client1 ~]# <b>sysctl -p</b>
    net.ipv4.ip_forward = 1
    [root@client1 ~]# <b>cat /proc/sys/net/ipv4/ip_forward</b>
    1
    </pre>
  * Then  
    <pre>
    [root@client1 ~]# <b>vi /etc/sysconfig/network-scripts/route-eth0</b>
    10.0.0.0/24 via 192.168.4.254 dev eth0
    [root@client1 ~]# <b>systemctl restart network</b>
    [root@client1 ~]# <b>ip r s</b>
    default via 192.168.4.2 dev eth0 proto static metric 100 
    <b>10.0.0.0/24 via 192.168.4.254 dev eth0 proto static metric 100</b> 
    192.168.4.0/24 dev eth0 proto kernel scope link src 192.168.4.5 metric 100 
    192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1
    </pre>

### Bridging

Nowadays, we use bridging in Virtualization:

![virtual bridge](https://user-images.githubusercontent.com/31813625/36599981-0295c468-187f-11e8-870d-46898294a8d5.png)

<pre>
[root@client1 ~]# <b>ip addr show virbr0</b>
3: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:ed:af:83 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
[root@client1 ~]# <b>brctl show</b>
bridge name	bridge id		STP enabled	interfaces
virbr0		8000.525400edaf83	yes		virbr0-nic
</pre>
But there is no configuration in `/etc/sysconfig/network-scripts/` for `virbr0`
<pre>
[root@client1 ~]# <b>ls /etc/sysconfig/network-scripts/</b>
ifcfg-eth0              ifdown-isdn             ifup                    ifup-plip               ifup-tunnel
ifcfg-lo                ifdown-post             ifup-aliases            ifup-plusb              ifup-wireless
ifdown                  ifdown-ppp              ifup-bnep               ifup-post               init.ipv6-global
ifdown-bnep             ifdown-routes           ifup-eth                ifup-ppp                network-functions
ifdown-eth              ifdown-sit              ifup-ib                 ifup-routes             network-functions-ipv6
ifdown-ib               ifdown-Team             ifup-ippp               ifup-sit                route-eth0
ifdown-ippp             ifdown-TeamPort         ifup-ipv6               ifup-Team               
ifdown-ipv6             ifdown-tunnel           ifup-isdn               ifup-TeamPort           
</pre>
For KVM, the configuration is in:
```bash
[root@client1 ~]# cat /etc/libvirt/qemu/networks/default.xml 
<!--
WARNING: THIS IS AN AUTO-GENERATED FILE. CHANGES TO IT ARE LIKELY TO BE
OVERWRITTEN AND LOST. Changes to this xml configuration should be made using:
  virsh net-edit default
or other application using the libvirt API.
-->

<network>
  <name>default</name>
  <uuid>fe147623-ccfb-49f1-a45f-b1d1268bb352</uuid>
  <forward mode='nat'/>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:ed:af:83'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
    </dhcp>
  </ip>
</network>
```
* Creating software bridge using NetworkManager? use `man nmcli-examples`: see example 8

### Link Aggregation
* To increase fault tolerance and bandwidth on bundled network
* We have 2 solutions
  * Bonding: legacy
  * Teaming: current
    * Faster
    * Modular and extensible
    * Small kernel driver talk to a user space daemon `teamd`
#### Link aggregation using teamd
* Runners
  * broadcast
  * roundrobin
  * activebackup
  * loadbalance
  * lacp  

use `man nmcli-examples`: see example 7
  * You will need a json file in `/etc/sysconfig/network-scripts/`. For example
  <pre>
  [root@client1 ~]# <b>cp /usr/share/doc/teamd-1.25/example_configs/lacp_1.conf /etc/sysconfig/network-scripts/</b></pre>
  
## Firewalling 
* The core of firewall in Linux is `netfilter` framework (inside the kernel)
* To manage `netfilter` we can use the following utilities
  * `iptables`
  * `firewalld`
    * zone
    * service
    * port
    * forward-port

### Firewalling with `iptables` toolset
* Not required for RHCE exam
* Traditional firewall system in the Linux distributions
* No concept of zones
* INPUT chain: Packet destined to out Linux machine
* OUTPUT chain: packets originating from the inside firewall and destined to outside
* FORWARD chain: packets that are being routed
    <pre>
    root@samba:~# <b>iptables -L</b>
    Chain <b>INPUT</b> (policy ACCEPT)
    target     prot opt source               destination

    Chain <b>FORWARD</b> (policy ACCEPT)
    target     prot opt source               destination

    Chain <b>OUTPUT</b> (policy ACCEPT)
    target     prot opt source               destination
    </pre>
* We don't have firewall being in an *off* condition. What we can do is save the configuration, then
we can go and read this back in later if we want to reset our firewall.
<pre>
root@samba:~# <b>iptables-save > fwoff</b>
root@samba:~# <b>cat fwoff</b>
# Generated by iptables-save v1.4.21 on Mon Jan  1 21:40:33 2018
*filter
:INPUT ACCEPT [4521:405288]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [2575:210924]
COMMIT
# Completed on Mon Jan  1 21:40:33 2018
</pre>
Let's add some entries and play with `iptables-save` and `iptables-restore`:
<pre>
root@samba:~# <b>iptables -A INPUT -i lo -j ACCEPT</b>
root@samba:~# <b>iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT</b>
root@samba:~# <b>iptables -A INPUT -p tcp --dport 22 -j ACCEPT</b>
root@samba:~# <b>iptables -nvL</b>
Chain INPUT (policy ACCEPT 3 packets, 234 bytes)
 pkts bytes target     prot opt in     out     source               destination
   17  1308 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0
  257 14780 ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
    0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:22

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 34 packets, 2628 bytes)
 pkts bytes target     prot opt in     out     source               destination
root@samba:~# <b>iptables-save > fwon.20180101</b>
root@samba:~# <b>cat fwon.20180101</b>
# Generated by iptables-save v1.4.21 on Mon Jan  1 21:47:39 2018
*filter
:INPUT ACCEPT [3:234]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [117:9116]
-A INPUT -i lo -j ACCEPT
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
COMMIT
# Completed on Mon Jan  1 21:47:39 2018
root@samba:~# <b>iptables-restore < fwoff</b>
root@samba:~# <b>iptables -L</b>
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
</pre>

* To deny the traffic `iptables -A INPUT -j DROP`.
  * Keep policy to ACCEPT and DROP as your last rule.
* To flush the table use `-F`, be sure your policy is ACCEPT to run `iptables -F` command.
    
### Firewalling with `firewalld`

<pre>
[vagrant@web ~]$ <b>firewall-cmd --state</b>
not running
[vagrant@web ~]$ <b>sudo systemctl start firewalld.service</b></pre>

<pre>
[vagrant@web ~]$ <b>firewall-cmd --get-default-zones</b>
public
[vagrant@web ~]$ <b>firewall-cmd --get-active-zones</b>
public
  interfaces: enp0s3
[vagrant@web ~]$ <b>firewall-cmd --get-zones</b>
block dmz drop external home internal public trusted work
</pre>

<pre>
root@web:~# <b>firewall-cmd --permanent --zone=internal --add-interface=lo</b>
success
root@web:~# <b>systemctl restart firewalld</b>
root@web:~# <b>firewall-cmd --get-active-zones</b>
internal
  interfaces: lo
</pre>

<pre>
[vagrant@web ~]$ <b>firewall-cmd --list-all</b>
public (default, active)
  interfaces: enp0s3
  sources:
  services: dhcpv6-client ssh
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:
</pre>

<pre>
[vagrant@web ~]$ <b>firewall-cmd --list-all --zone=internal</b>
internal
  interfaces:
  sources:
  services: dhcpv6-client ipp-client mdns samba-client ssh
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:
</pre>
<pre>
[root@web ~]# <b>firewall-cmd --reload</b></pre>
Add or remove services to the zones:
<pre>
[root@web ~]# <b>firewall-cmd --permanent --add-service=telnet --zone=public</b>
success
[root@web ~]# <b>firewall-cmd --reload</b>
success
[root@web ~]# <b>firewall-cmd --list-all</b>
public (default, active)
  interfaces: enp0s3
  sources:
  services: dhcpv6-client ssh <b>telnet</b>
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:
</pre>

<pre>
[root@web ~]# <b>firewall-cmd --permanent --remove-service={telnet,dhcpv6-client} --zone=public</b>
success
[root@web ~]# <b>firewall-cmd --reload</b>
success
[root@web ~]# <b>firewall-cmd --list-all</b>
public (default, active)
  interfaces: enp0s3
  sources:
  services: ssh
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:
</pre>

Creating our own services and adding in.
<pre>
[root@web ~]# <b>ls /usr/lib/firewalld/services/</b>
amanda-client.xml  ftp.xml                ipsec.xml        mdns.xml     pmcd.xml        radius.xml          telnet.xml
bacula-client.xml  high-availability.xml  kerberos.xml     mountd.xml   pmproxy.xml     RH-Satellite-6.xml  tftp-client.xml
bacula.xml         https.xml              kpasswd.xml      ms-wbt.xml   pmwebapis.xml   rpc-bind.xml        tftp.xml
dhcpv6-client.xml  http.xml               ldaps.xml        mysql.xml    pmwebapi.xml    samba-client.xml    transmission-client.xml
dhcpv6.xml         imaps.xml              ldap.xml         nfs.xml      pop3s.xml       samba.xml           vnc-server.xml
dhcp.xml           ipp-client.xml         libvirt-tls.xml  ntp.xml      postgresql.xml  smtp.xml            wbem-https.xml
dns.xml            ipp.xml                libvirt.xml      openvpn.xml  proxy-dhcp.xml  ssh.xml
</pre>

<pre>
[root@web ~]# <b>firewall-cmd --permanent --new-service="puppet"</b>
success
[root@web ~]# <b>ls /etc/firewalld/services/</b>
puppet.xml
[root@web ~]# <b>restorecon /etc/firewalld/services/puppet.xml</b>
[root@web ~]# <b>chmod 640 !$</b>
chmod 640 /etc/firewalld/services/puppet.xml
</pre>

#### Understanding Rich Rules
*Direct rules* allow admins to insert hand-coded rules
* Direct rules are processed before anything in the zones
* Not recommended to use them 

*Rich rules* use an expressive language to create custom rules that cannot be created with basic syntax
  * logging
  * port forwarding
  * masquerading
  * rate limiting
  * use `man 5 firewalld.richlanguage`
  
Rules processing order
1. port forwarding and masquerading rules
2. logging rules
3. deny rules
4. allow rules  

<pre>
[root@client1 ~]# <b>firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address=192.168.4.25 reject'</b>
success
[root@client1 ~]# <b>firewall-cmd --reload</b> 
success
[root@client1 ~]# <b>firewall-cmd --list-all</b>
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0
  sources: 
  services: ssh dhcpv6-client
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	<b>rule family="ipv4" source address="192.168.4.25" reject</b></pre>

#### Configuring Port Forwarding

To forward inbound network packets from one port to an alternative port or address, first
enable IP address masquerading for a zone (for example, external).

  * IP address masquerading in Linux is similar to the one-to-many (1:Many) NAT
  which allows internal computers with no known address outside their network, to communicate to the outside.
  * A computer can be either a router or a firewall, but not both. If you set up a computer to act as both a router and a firewall, you have defeated the purpose of your firewall!
  * Port forwarding is an application of network address translation (NAT)
  that redirects a communication request from one address and port number
  combination to another while the packets are traversing a network gateway, such as a router or firewall.

```
[root@web ~]# firewall-cmd --help | grep add-forward
  --add-forward-port=port=<portid>[-<portid>]:proto=<protocol>[:toport=<portid>[-<portid>]][:toaddr=<address>[/<mask>]]
```
<pre>
[root@web ~]# firewall-cmd --list-all --zone=home
home (default, active)
  interfaces: enp0s3 enp0s8
  sources:
  services: dhcpv6-client ipp-client mdns samba-client ssh
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:

[root@web ~]# <b>firewall-cmd --zone=home --add-masquerade --permanent</b>
success
[root@web ~]# <b>firewall-cmd --zone=home --add-forward-port=port=8999:proto=tcp:toport=80 --permanent</b>
success
[root@web ~]# <b>firewall-cmd --reload</b>
[root@web ~]# <b>firewall-cmd --list-all --zone=home</b>
home (default, active)
  interfaces: enp0s3 <b>enp0s8</b>
  sources:
  services: dhcpv6-client ipp-client mdns samba-client ssh
  ports:
  <b>masquerade: yes
  forward-ports: port=8999:proto=tcp:toport=80:toaddr=</b>
  icmp-blocks:
  rich rules:
</pre>
To test:
```
hossein@hossein ~ $ curl 192.168.33.20
curl: (7) Failed to connect to 192.168.33.20 port 80: No route to host
hossein@hossein ~ $ curl 192.168.33.20:8999
<h1>Welcome to 192.168.33.20</h1>
```

## Tunnel traffic

### Implementing SSH Tunnels

* HTTP server IP is: 192.168.33.20
* Client is the `ac` with IP address 192.168.33.10

<pre>
vagrant@ac:~$ <b>curl -I -L 192.168.33.20</b>
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
vagrant@ac:~$ <b>ssh -f -N -L 8880:localhost:80 192.168.33.20</b>
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
vagrant@ac:~$ <b>curl -I -L localhost:8880</b>
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
vagrant@ac:~$ <b>kill 1312</b>
vagrant@ac:~$ <b>curl -I -L localhost:8880</b>
curl: (7) Failed to connect to localhost port 8880: Connection refused
</pre>

## ss - socket statistics
Previously, we saw how to use the `netstat` command to get statistics on
network/socket connections. However the `netstat` command has long been
deprecated and replaced by the `ss` command from the iproute suite of tools.

<pre>
[root@web ~]# <b>ss | wc -l</b>
86
</pre>
* `l`: listening ports
* `t`: TCP
* `u`: UDP
* `n`: Does'n resolve hostname and port number
* `p`: process name and PID
* `s`: summary statistics