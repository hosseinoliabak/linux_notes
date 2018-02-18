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
  * iotop: information about I/O
  * vmstat: Virtual Memory Statistics (This is not swap)
  * sar: System Activity Reporter