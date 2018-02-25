# NFS (Network File System) Server
NFS server: The NFS server is the traditional way of sharing or exporting file systems in UNIX and Linux.
Often, home directories in Linux are centralized on a single server with other systems mounting the remote file systems.
<pre>
[root@target ~]# <b>yum install nfs-utils</b>
[root@target ~]# <b>systemctl enable nfs-server</b>
Created symlink from /etc/systemd/system/multi-user.target.wants/nfs-server.service to /usr/lib/systemd/system/nfs-server.service.
[root@target ~]# <b>systemctl start nfs-server</b>
[root@target ~]# <b>ss -ntl</b>
State      Recv-Q Send-Q                           Local Address:Port                                          Peer Address:Port                          
LISTEN     0      128                                          *:20048                                                    *:*                                   
LISTEN     0      64                                           *:2049                                                     *:*                  
LISTEN     0      128                                          *:57890                                                    *:*                  
LISTEN     0      64                                           *:36298                                                    *:*                  
LISTEN     0      128                                         :::20048                                                   :::*                  
LISTEN     0      128                                         :::45684                                                   :::*                  
LISTEN     0      64                                          :::40513                                                   :::*                  
LISTEN     0      64                                          :::2049                                                    :::*    
</pre>
By default, the NFS server starts just 8 processes. In a small setup this may work but not in anything larger.
As this is kernel-based a reboot is required. You can change it through `/etc/sysconfig/nfs`
<pre>
[root@target ~]# <b>pgrep nfsd</b>
1139
1149
1151
1153
1156
1159
1161
1164
1166
[root@target ~]# <b>cat /etc/sysconfig/nfs</b>
...
# Number of nfs server processes to be started.
# The default is 8. 
<b>RPCNFSDCOUNT=16</b>
...
[root@target ~]# <b>reboot</b>
[root@target ~]# <b>pgrep nfsd</b>
1117
1126
1128
1129
1130
1132
1134
1135
1136
1137
1140
1141
1142
1144
1145
1146
1148
</pre>

### Creating Exports in NFS
* The `/etc/exports` file controls which file systems are exported to remote hosts and specifies options.
<pre>
[root@target ~]# <b>showmount -e</b>
Export list for target.example.com:
[root@target ~]# <b>exportfs -v</b> # Nothing is returned so far
[root@target ~]# <b>mkdir -p /exports/{home,etc}</b>
[root@target ~]# <b>tail -n2 /etc/fstab</b> 
/home /exports/home none bind 0 0
/etc /exports/etc none bind 0 0
[root@target ~]# <b>mount -a</b>
[root@target ~]# <b>vi /etc/exports</b>
/exports *(rw,no_root_squash,crossmnt,fsid=0)
[root@target ~]# <b>exportfs -r</b>
[root@target ~]# <b>exportfs -v</b>
/exports      	<world>(rw,sync,wdelay,hide,crossmnt,no_subtree_check,fsid=0,sec=sys,secure,no_root_squash,no_all_squash)
[root@target ~]# <b>mount 127.0.0.1:/ /mnt</b>
[root@target ~]# <b>ls /mnt/</b>
etc  home
</pre>
* Host notation: We can share host names or to IP addresses. This can be specifically or via forms of wildcards.
The most specific match will be used where a host matches more than one export for a specific directory.
  * All hosts = `*`
  * All hosts in example.com = `*.example.com`
  * Subnet = `192.168.4.*`
  * Subnet mask = `192.168.4.0/255.255.255.0`
  * Subnet CIDR = `192.168.4.0/24`
  * Specific IP address = `192.168.4.16`
* Squashing users: NFS options that include the word squash relates to mapping users to an anonymous
account that can only access files that are world readable.
  * The option `root_squash` prevents root users connected remotely from having root privileges
  and assigns them the user ID for the user `nfsnobody`. This effectively *squashes* the power of
  the remote root user to the lowest local user, preventing unauthorized alteration of files on
  the remote server.  It is NOT a good idea to reverse this.
  * `all_squash`: squashes every remote user, including root.

# SAMBA
Samba is the standard Windows interoperability suite of programs for Linux and Unix.

Andrew Tridgell, the Australian developer of SAMBA wanted to use the term
SMB Server but there were naming issues with this. So using the command
`grep` and inbuilt dictionary the name was derived.

<pre>
vagrant@ac:~$ <b>grep '^s.*m.*b' /usr/share/dict/words | head -n1</b>
samba
</pre>

## Installing SAMBA
<pre>
root@samba:~# <b>apt-cache show samba</b>
Package: samba
Version: 2:4.2.14+dfsg-0+deb8u9
Installed-Size: 11417
Maintainer: Debian Samba Maintainers <pkg-samba-maint@lists.alioth.debian.org>
...
root@samba:~# <b>apt-cache show smbclient</b>
Package: smbclient
Source: samba
Version: 2:4.2.14+dfsg-0+deb8u9
...
root@samba:~# <b>apt-cache show winbind</b>
Package: winbind
Source: samba
Version: 2:4.2.14+dfsg-0+deb8u9
...
</pre>
* `smbclient` is a package that allows us to connect to Windows Shares and Printers.
* `winbind` is a package the allows a Linux system to join to a Windows domain.
<pre>
root@samba:~# <b>apt install -y samba smbclient winbind</b></pre>

### Checking for ACL support
* The file system needs to support ACLs for SAMBA. The standard Linux
mode allows for one user and one group, whereas in Windows, we have ACL
support allowing many users and groups to be listed.
<pre>
root@samba:~# <b>mount | grep -wF '/'</b>
/dev/sda1 on / type ext4 (rw,relatime,errors=remount-ro,data=ordered)
root@samba:~# <b>touch file1</b>
root@samba:~# <b>ls -l file1</b>
-rw-r--r-- 1 root root 0 Jan  1 02:15 file1
root@samba:~# <b>setfacl -m g:sudo:wr file1</b> # -m means modify
root@samba:~# <b>ls -l file1</b>
-rw-rw-r--<b>+</b> 1 root root 0 Jan  1 02:15 file1
</pre>
`+` means now ACL is set.
<pre>
root@samba:~# <b>getfacl file1</b>
# file: file1
# owner: root
# group: root
user::rw-
group::r--
group:sudo:rw-
mask::rw-
other::r--
root@samba:~# <b>rm file1</b></pre>

### SAMBA Ports
With the `smbd` service running, we have two ports open (139, 445). We can lookup
their full description from the services file. We use an extended regular
expression here (`grep -E`) to support the `grouping` and `OR` statement
in the regular expression.

<pre>
root@samba:~# <b>grep -Ew '(139|445)/tcp' /etc/services</b>
netbios-ssn	139/tcp				# NETBIOS session service
microsoft-ds	445/tcp				# Microsoft Naked CIFS
</pre>

**Samba daemons and services:**
* `smbd` service is our file and print service for Windows. In addition, it is responsible for user authentication, resource locking, and data sharing through the SMB protocol. The default ports on which the server listens for SMB traffic are TCP ports 139 and 445.
* `nmbd` understands and replies to NetBIOS name service requests such as those produced by SMB/CIFS in Windows-based systems. The default port that the server listens to for NMB traffic is UDP port 137.
* `winbindd`
* `samba`
* `samba-ad-dc` is not being commonly used
<pre>
root@samba:~# <b>systemctl status smbd nmbd winbind</b>
● smbd.service - LSB: start Samba SMB/CIFS daemon (smbd)
   Loaded: loaded (/etc/init.d/smbd)
   Active: active (running) since Mon 2018-01-01 02:06:18 GMT; 20min ago
   CGroup: /system.slice/smbd.service
           ├─5334 /usr/sbin/smbd -D
           └─5350 /usr/sbin/smbd -D

Jan 01 02:06:18 samba smbd[5325]: Starting SMB/CIFS daemon: smbd.
Jan 01 02:06:18 samba systemd[1]: Started LSB: start Samba SMB/CIFS daemon (smbd).

● nmbd.service - LSB: start Samba NetBIOS nameserver (nmbd)
   Loaded: loaded (/etc/init.d/nmbd)
   Active: active (running) since Mon 2018-01-01 02:06:19 GMT; 20min ago
   CGroup: /system.slice/nmbd.service
           └─5395 /usr/sbin/nmbd -D

Jan 01 02:06:19 samba nmbd[5384]: Starting NetBIOS name server: nmbd.
Jan 01 02:06:19 samba systemd[1]: Started LSB: start Samba NetBIOS nameserver (nmbd).

● winbind.service - LSB: start Winbind daemon
   Loaded: loaded (/etc/init.d/winbind)
   Active: active (running) since Mon 2018-01-01 02:06:19 GMT; 20min ago
   CGroup: /system.slice/winbind.service
           ├─5520 /usr/sbin/winbindd
           └─5524 /usr/sbin/winbindd

Jan 01 02:06:19 samba winbind[5509]: Starting the Winbind daemon: winbind.
Jan 01 02:06:19 samba systemd[1]: Started LSB: start Winbind daemon.
root@samba:~# <b>systemctl status samba</b>
samba-ad-dc.service  samba.service
root@samba:~# <b>systemctl status samba*</b>
● samba-ad-dc.service - LSB: start Samba daemons for the AD DC
   Loaded: loaded (/etc/init.d/samba-ad-dc)
   Active: active (exited) since Mon 2018-01-01 02:06:19 GMT; 22min ago

Jan 01 02:06:19 samba systemd[1]: Started LSB: start Samba daemons for the AD DC.

● samba.service
   Loaded: masked (/dev/null)
   Active: inactive (dead)
</pre>

### Manage services internally with SAMBA tools
* For now we just need to create an account in SAMBA for the root user.
* We can also use internal tools to SAMBA to list services.
* User accounts in Linux have a UID, in CIFS and SAMBA users have a much longer SID.
We need to create corresponding SAMBA accounts for our Linux users if they are connecting to SAMBA.
* To create a SAMBA user, there must be a corresponding Linux account for the SAMBA user we are creating:
  <pre>
  root@samba:~# <b>pdbedit --create root</b>
  new password:
  retype new password:
  Unix username:        root
  NT username:
  Account Flags:        [U          ]
  User SID:             S-1-5-21-2307813929-3417554111-1932675140-1000
  Primary Group SID:    S-1-5-21-2307813929-3417554111-1932675140-513
  Full Name:            root
  Home Directory:       \\samba\root
  HomeDir Drive:
  Logon Script:
  Profile Path:         \\samba\root\profile
  Domain:               SAMBA
  Account desc:
  Workstations:
  Munged dial:
  Logon time:           0
  Logoff time:          Wed, 06 Feb 2036 15:06:39 GMT
  Kickoff time:         Wed, 06 Feb 2036 15:06:39 GMT
  Password last set:    Mon, 01 Jan 2018 02:47:16 GMT
  Password can change:  Mon, 01 Jan 2018 02:47:16 GMT
  Password must change: never
  Last bad password   : 0
  Bad password count  : 0
  Logon hours         : FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF
  root@samba:~# <b>pdbedit -L</b>
  root:0:root
  </pre>
* The `nmblookup` program resolves NetBIOS names into IP addresses.
The program broadcasts its query on the local subnet until the target machine replies.
  <pre>
  root@samba:~# <b>nmblookup samba</b>
  10.0.2.15 samba<00>
  </pre>
* Some internal commands to see the services:
  <pre>
  root@samba:~# <b>smbstatus -V</b>
  Version 4.2.14-Debian
  </pre>
  * `net` command to how internal services:
    <pre>
    root@samba:~# <b>net rpc service list -U root</b>
    Enter root's password:
    Spooler                 "Print Spooler"
    NETLOGON                "Net Logon"
    RemoteRegistry          "Remote Registry Service"
    WINS                    "Windows Internet Name Service (WINS)"
    </pre>

## Creating, modifying, and listing users

* Password backends: SAMBA password backends store the user account details,
including passwords. The default used to be the `/etc/samba/smbpasswd` file
but in newer releases the `tdbsam` database is used. Another common backend is LDAP.

<pre>
root@samba:~# <b>grep -FB2 'passdb backend' /etc/samba/smb.conf</b>
# If you are using encrypted passwords, Samba will need to know what
# password database type you are using.
   <b>passdb backend = tdbsam</b></pre>

### pdbedit
`pdbedit` command has 5 main functions
* List users: `-L` or `-Lv` or `Lu username`
* Create users: `-a`
* Delete users: `-x`
* Modify users" `-r`
* Import users

**Lab:** Creating a user account without being prompted for the password.
<pre>
root@samba:~# <b>printf "%s\n%s\n" test test | sudo pdbedit -a -t ryan</b>
Unix username:        ryan
NT username:
Account Flags:        [U          ]
User SID:             S-1-5-21-2307813929-3417554111-1932675140-1002
Primary Group SID:    S-1-5-21-2307813929-3417554111-1932675140-513
Full Name:
Home Directory:       \\samba\ryan
HomeDir Drive:
Logon Script:
Profile Path:         \\samba\ryan\profile
Domain:               SAMBA
Account desc:
Workstations:
Munged dial:
Logon time:           0
Logoff time:          Wed, 06 Feb 2036 15:06:39 GMT
Kickoff time:         Wed, 06 Feb 2036 15:06:39 GMT
Password last set:    Mon, 01 Jan 2018 03:15:50 GMT
Password can change:  Mon, 01 Jan 2018 03:15:50 GMT
Password must change: never
Last bad password   : 0
Bad password count  : 0
Logon hours         : FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF
</pre>

**Lab:** Modifying a user account
<pre>
root@samba:~# <b>pdbedit -r -u ryan -f "Ryan Oliabak"</b>
...
root@samba:~# <b>pdbedit -L</b>
root:0:root
ryan:1001:Ryan Oliabak
</pre>
`X` Non-expiry password. `N` We can use empty passwords for `ryan`
<pre>
root@samba:~# pdbedit -r -u ryan -c "[XN]"
Unix username:        ryan
NT username:
<b>Account Flags:        [NUX        ]</b>
User SID:             S-1-5-21-2307813929-3417554111-1932675140-1002
Primary Group SID:    S-1-5-21-2307813929-3417554111-1932675140-513
Full Name:            Ryan Oliabak
Home Directory:       \\samba\ryan
HomeDir Drive:
Logon Script:
Profile Path:         \\samba\ryan\profile
Domain:               SAMBA
Account desc:
Workstations:
Munged dial:
Logon time:           0
Logoff time:          Wed, 06 Feb 2036 15:06:39 GMT
Kickoff time:         Wed, 06 Feb 2036 15:06:39 GMT
Password last set:    Mon, 01 Jan 2018 03:15:50 GMT
Password can change:  Mon, 01 Jan 2018 03:15:50 GMT
Password must change: Tue, 19 Jan 2038 03:14:07 GMT
Last bad password   : 0
Bad password count  : 0
Logon hours         : FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF
</pre>
To unset the account control settings:
<pre>
root@samba:~# <b>pdbedit -r -u ryan -c "[]"</b></pre>

#### Account Policies
We don't modify the user, we modify the policy itself.

To list the policy's names and change the defaults:
<pre>
root@samba:~# <b>pdbedit -P ""</b>
No account policy by that name!
Account policy names are:
min password length
password history
user must logon to change password
maximum password age
minimum password age
lockout duration
reset count minutes
bad lockout attempt
disconnect time
refuse machine password change
root@samba:~# <b>pdbedit -P "maximum password age"</b>
account policy "maximum password age" description: Maximum password age, in seconds (default: -1 => never expire passwords)
account policy "maximum password age" value is: 4294967295
root@samba:~# <b>pdbedit -P "maximum password age" -C 3456000</b>
account policy "maximum password age" description: Maximum password age, in seconds (default: -1 => never expire passwords)
account policy "maximum password age" value was: 4294967295
account policy "maximum password age" value is now: 3456000
root@samba:~# <b>pdbedit -Lvu ryan</b>
Unix username:        ryan
NT username:
Account Flags:        [U          ]
User SID:             S-1-5-21-2307813929-3417554111-1932675140-1002
Primary Group SID:    S-1-5-21-2307813929-3417554111-1932675140-513
Full Name:            Ryan Oliabak
Home Directory:       \\samba\ryan
HomeDir Drive:
Logon Script:
Profile Path:         \\samba\ryan\profile
Domain:               SAMBA
Account desc:
Workstations:
Munged dial:
Logon time:           0
Logoff time:          Wed, 06 Feb 2036 15:06:39 GMT
Kickoff time:         Wed, 06 Feb 2036 15:06:39 GMT
Password last set:    Mon, 01 Jan 2018 03:15:50 GMT
Password can change:  Mon, 01 Jan 2018 03:15:50 GMT
Password must change: <b>Sat, 10 Feb 2018 03:15:50 GMT</b>
Last bad password   : 0
Bad password count  : 0
Logon hours         : FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF
</pre>
To unlock an account caused by bad password count:
<pre>
root@samba:~# <b>pdbedit -ru ryan -z</b></pre>

## Managing shares

* `smb.conf` The SAMBA configuration is stored in the file `/etc/samba/smb.conf`.
Many settings are explicitly configured but even more are not and load from the defaults in SAMBA 4.
* Cleaning our configuration file:
  * `;` and `#` are used as comments
* Reading the configuration: Te command `testparm -s` can be used to test configuration
updates but also prints the configuration in various forms.

<pre>
root@samba:/etc/samba# <b>testparm -s | wc -l</b>
Load smb config files from /etc/samba/smb.conf
Processing section "[homes]"
Processing section "[printers]"
Processing section "[print$]"
Loaded services file OK.
Server role: ROLE_STANDALONE

<b>38</b>
root@samba:/etc/samba# <b>testparm -sv | wc -l</b>
Load smb config files from /etc/samba/smb.conf
Processing section "[homes]"
Processing section "[printers]"
Processing section "[print$]"
Loaded services file OK.
Server role: ROLE_STANDALONE

<b>450</b></pre>

<pre>
root@samba:/etc/samba# <b>sed '/^\s*#/d;/^\s*\;/d;/^$/d' smb.conf</b>
[global]
   workgroup = WORKGROUP
   dns proxy = no
   log file = /var/log/samba/log.%m
   max log size = 1000
   syslog = 0
   panic action = /usr/share/samba/panic-action %d
   server role = standalone server
   passdb backend = tdbsam
   obey pam restrictions = yes
   unix password sync = yes
   passwd program = /usr/bin/passwd %u
   passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .
   pam password change = yes
   map to guest = bad user
   usershare allow guests = yes
[homes]
   comment = Home Directories
   browseable = no
   read only = yes
   create mask = 0700
   directory mask = 0700
   valid users = %S
[printers]
   comment = All Printers
   browseable = no
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes
   create mask = 0700
[print$]
   comment = Printer Drivers
   path = /var/lib/samba/printers
   browseable = yes
   read only = yes
   guest ok = no
</pre>
* Ideally we do not want to be editing the production configuration file.
The services read the configuration every 60 seconds, so a misconfiguration
can cause issues without us restarting. We should copy the configuration, editing and testing
the dummy configuration (`smb.edit`) before writing to real configuration.
Then we deploy the configuration:
  * `testparm -s smb.edit > smb.conf`
* `[global]` section:
  * workgroup
  * security: defines the authentication option used for the server.
  This can be set to `AUTO | USER | DOMAIN | ADS`
  * log file: defines the path to the log files that will be used.
    * `%m` machine name
* `[shares]` section: each share has its own section `[homes]`, `[printers]`
  * `[home]` share is designed to automatically share out the home directory
  of each user on the system. In the Debian configuration this is active by default.
    * either `browsable` or `browseable` are acceptable in the `smb.conf`.
  * `[printer]` share will share all printers on the system.
  This is active on the Debian system by default.

<pre>
root@samba:/etc/samba# <b>testparm -sv --parameter-name="security"</b>
AUTO
root@samba:/etc/samba# <b>testparm -sv --parameter-name="server role"</b>
standalone server
root@samba:/etc/samba# <b>testparm -sv --show-all-parameters | grep -F 'security'</b>
security=P_ENUM,AUTO|USER|DOMAIN|ADS,FLAG_BASIC|FLAG_WIZARD|FLAG_ADVANCED
root@samba:/etc/samba# <b>testparm -sv --show-all-parameters | grep -F 'server role'</b>
server role=P_ENUM,auto|standalone server|standalone|member server|member|classic primary domain controller|classic backup domain controller|active directory domain controller|domain controller|dc,FLAG_BASIC|FLAG_ADVANCED
</pre>

<pre>
root@samba:/etc/samba# <b>grep -F 'log file' smb.conf</b>
   log file = /var/log/samba/log.%m
root@samba:/etc/samba# <b>cat /var/log/samba/log.nmbd</b>
[2018/01/01 02:06:19,  0] ../source3/nmbd/nmbd.c:908(main)
  nmbd version 4.2.14-Debian started.
  Copyright Andrew Tridgell and the Samba Team 1992-2014
[2018/01/01 02:06:19.117823,  0] ../lib/util/become_daemon.c:124(daemon_ready)
  STATUS=daemon 'nmbd' finished starting up and ready to serve connections
[2018/01/01 02:06:42.164424,  0] ../source3/nmbd/nmbd_become_lmb.c:397(become_local_master_stage2)
  *****

  Samba name server SAMBA is now a local master browser for workgroup WORKGROUP on subnet 192.168.33.101

  *****
[2018/01/01 02:06:42.164777,  0] ../source3/nmbd/nmbd_become_lmb.c:397(become_local_master_stage2)
  *****

  Samba name server SAMBA is now a local master browser for workgroup WORKGROUP on subnet 10.0.2.15

  *****
</pre>

To see the shares:
<pre>
root@samba:/etc/samba# <b>net share</b>
Enter root's password:
print$
IPC$
root
</pre>

### Creating our own shares

<pre>
root@samba:/etc/samba# <b>mkdir -m 770 /data</b>
root@samba:/etc/samba# <b>chgrp sudo /data</b>
root@samba:/etc/samba# <b>tail -n7 smb.conf
[data-share]
   comment = Admin share
   path = /data
   valid users = ryan,root,+sudo
   writeable = true
   read list = ryan
   write list = root</b>
root@samba:/etc/samba# <b>!net
net share</b>
Enter root's password:
print$
IPC$
data-share
root
</pre>

## Connecting to shares
In lunyx, the `smbclient` package ie predominately responsible for listing
and connecting to shares on SAMBA or Windows server

To list the content (files) of `smbclient`
<pre>
root@samba:~# <b>dpkg -L smbclient</b>
root@samba:~# dpkg -L smbclient
/.
/usr
/usr/bin
/usr/bin/smbget
/usr/bin/smbspool
/usr/bin/smbtar
/usr/bin/smbspool_krb5_wrapper
/usr/bin/smbclient
/usr/bin/cifsdd
/usr/bin/smbtree
/usr/bin/smbcquotas
/usr/bin/smbcacls
/usr/bin/rpcclient
...
</pre>
### Listing the share
<pre>
root@samba:~# <b>smbclient -L //localhost</b>
Enter root's password:
Domain=[WORKGROUP] OS=[Windows 6.1] Server=[Samba 4.2.14-Debian]

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	data-share      Disk      Admin share
	IPC$            IPC       IPC Service (Samba 4.2.14-Debian)
	root            Disk      Home Directories
Domain=[WORKGROUP] OS=[Windows 6.1] Server=[Samba 4.2.14-Debian]

	Server               Comment
	---------            -------
	SAMBA                Samba 4.2.14-Debian

	Workgroup            Master
	---------            -------
	WORKGROUP            SAMBA
</pre>

<pre>
root@samba:~# <b>smbclient -U ryan -L //localhost</b>
Enter ryan's password:
Domain=[WORKGROUP] OS=[Windows 6.1] Server=[Samba 4.2.14-Debian]

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	data-share      Disk      Admin share
	IPC$            IPC       IPC Service (Samba 4.2.14-Debian)
	ryan            Disk      Home Directories
Domain=[WORKGROUP] OS=[Windows 6.1] Server=[Samba 4.2.14-Debian]

	Server               Comment
	---------            -------
	SAMBA                Samba 4.2.14-Debian

	Workgroup            Master
	---------            -------
	WORKGROUP            SAMBA
</pre>
We don't want to be prompted for username and password:
<pre>
root@samba:~# <b>cat chips.fish</b>
username=ryan
password=test
root@samba:~# <b>chmod 600 chips.fish</b>
root@samba:~# <b>smbclient -A chips.fish -L //localhost</b>
Domain=[WORKGROUP] OS=[Windows 6.1] Server=[Samba 4.2.14-Debian]

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	data-share      Disk      Admin share
	IPC$            IPC       IPC Service (Samba 4.2.14-Debian)
	ryan            Disk      Home Directories
Domain=[WORKGROUP] OS=[Windows 6.1] Server=[Samba 4.2.14-Debian]

	Server               Comment
	---------            -------
	SAMBA                Samba 4.2.14-Debian

	Workgroup            Master
	---------            -------
	WORKGROUP            SAMBA
</pre>
### Using smbclient to connect to shares
<pre>
root@samba:~# <b>testparm -s --section-name=data-share</b>
Load smb config files from /etc/samba/smb.conf
Processing section "[homes]"
Processing section "[printers]"
Processing section "[print$]"
Processing section "[data-share]"
Loaded services file OK.

[data-share]
	comment = Admin share
	path = /data
	valid users = ryan root +sudo
	read list = ryan
	write list = root
	read only = No
root@samba:/data# <b>touch test</b>
root@samba:/data# <b>ls</b>
test
root@samba:~# <b>smbclient -A chips.fish //localhost/data-share</b>
Domain=[WORKGROUP] OS=[Windows 6.1] Server=[Samba 4.2.14-Debian]
smb: \> <b>ls</b>
  .                                   D        0  Mon Jan  1 20:26:22 2018
  ..                                  D        0  Mon Jan  1 04:41:34 2018
  hosts                               N      230  Mon Jan  1 20:26:22 2018
  test                                N        0  Mon Jan  1 19:17:16 2018

		9620408 blocks of size 1024. 8029744 blocks available
smb: \> <b>exit</b>
root@samba:~# <b>smbclient -A chips.fish //localhost/data-share &</b>
[1] 1704
root@samba:~# Domain=[WORKGROUP] OS=[Windows 6.1] Server=[Samba 4.2.14-Debian]


[1]+  Stopped                 smbclient -A chips.fish //localhost/data-share
root@samba:~# <b>smbstatus -b</b>

Samba version 4.2.14-Debian
PID     Username      Group         Machine            Protocol Version
------------------------------------------------------------------------------
1706      root          root          ::1          (ipv6:::1:35092) NT1
</pre>

#### Mounting Windows shares
We can mount SAMBA or Windows shares but we need to ensure that we have
the package ``cifs-utils` installed. When accessing the mount, all users
will use the permissions from the account used in the mount options.
<pre>
root@samba:~# <b>tail -n1 /etc/fstab</b>
//127.0.0.1/data-share /mnt cifs credentials=/etc/chips.fish	0	0
root@samba:~# <b>cp chips.fish /etc</b>
root@samba:~# <b>mount -a</b>
mount: wrong fs type, bad option, bad superblock on //127.0.0.1/data-share,
       missing codepage or helper program, or other error
       (for several filesystems (e.g. nfs, cifs) you might
       need a /sbin/mount.<type> helper program)

       In some cases useful info is found in syslog - try
       dmesg | tail or so.
root@samba:~# <b>apt install -y cifs-utils</b>
root@samba:~# <b>mount -a</b>
root@samba:~# <b>ls /mnt</b>
hosts
root@samba:~# <b>smbclient -U test%Password1 //192.168.33.103/Share</b>
Domain=[RYAN] OS=[Windows Server 2012 R2 Datacenter Evaluation 9600] Server=[Windows Server 2012 R2 Datacenter Evaluation 6.3]
smb: \> <b>ls</b>
  .                                   D        0  Mon Jan  1 22:47:04 2018
  ..                                  D        0  Mon Jan  1 22:47:04 2018
  I am a file on Windows Server.txt      A        0  Mon Jan  1 22:46:52 2018

		6463487 blocks of size 4096. 3460633 blocks available
</pre>

## Linux and Active Directory
* Active Directory is Microsoft implementation of LDAP to store user account
data and Kerberos for authentication. With SAMBA 4, we can create our own Active Directory
domain or join an existing AD domain.
  * We need additional softwares installed.
* Time Synchronization: Time should be synchronized to the Internet when
working with the Active Directory. Kerberos requires systems to have the
same time.
    <pre>
    root@samba:~# <b>apt get install -y ntp</b>
    root@samba:~# <b>ntpq -p</b>
         remote           refid      st t when poll reach   delay   offset  jitter
    ==============================================================================
    +0.debian.pool.n .PPS.            1 u   60   64    1   22.210   -3.997  21.991
    +1.debian.pool.n 5.56.147.93      2 u   59   64    1   30.894   -1.714   5.591
    <b>*2.debian.pool.n .PPS.            1 u   58   64    1   19.935   -5.091  19.795</b>
     3.debian.pool.n 88.159.1.196     3 u   25   64    1  200.631  -51.376  26.213
    </pre>

### Deploying Active Directory in SAMBA 4
<pre>
root@samba:~# <b>apt install -y krb5-config krb5-user libpam-winbind libnss-winbind</b>
Default Kerberos version 5 realm: <b>EXAMPLE.COM</b>
Kerberos servers for your realm: <b>example.com</b>
Administrative server for your Kerberos realm: <b>example.com</b>
root@samba:~# <b>systemctl stop samba-ad-dc smbd nmbd winbind</b>
root@samba:~# <b>systemctl disable samba-ad-dc smbd nmbd winbind</b>
root@samba:~# <b>mv /etc/samba/smb.conf /etc/samba/smb.old</b>
root@samba:~# <b>samba-tool domain provision --server-role=dc --use-rfc2307 --dns-backend=SAMBA_INTERNAL --realm=EXAMPLE.COM --domain EXAMPLE --adminpass=Password1</b>
root@samba:~# <b>mv /etc/krb5.conf /etc/krb5.orig</b>
root@samba:~# <b>ln -s /var/lib/samba/private/krb5.conf /etc</b>
root@samba:~# <b>systemctl enable samba-ad-dc</b>
Synchronizing state for samba-ad-dc.service with sysvinit using update-rc.d...
Executing /usr/sbin/update-rc.d samba-ad-dc defaults
insserv: warning: current start runlevel(s) (empty) of script `samba-ad-dc' overrides LSB defaults (2 3 4 5).
insserv: warning: current stop runlevel(s) (0 1 2 3 4 5 6) of script `samba-ad-dc' overrides LSB defaults (0 1 6).
Executing /usr/sbin/update-rc.d samba-ad-dc enable
root@samba:~# <b>systemctl start samba-ad-dc</b>
root@samba:~# <b>vim /etc/dhcp/dhclient.conf</b>
...
supersede domain-name "example.com"; # Uncommented and edited
supersede domain-search "example.com"; # Added
prepend domain-name-servers 127.0.0.1; # Uncommented
...
root@samba:~# <b>systemctl restart networking</b></pre>
Test our configuration
<pre>
root@samba:~# <b>samba-tool domain level show</b>
Domain and forest function level for domain 'DC=example,DC=com'

Forest function level: (Windows) 2008 R2
Domain function level: (Windows) 2008 R2
Lowest function level of a DC: (Windows) 2008 R2

root@samba:~# <b>cat /etc/resolv.conf</b>
nameserver 127.0.0.1
search example.com
root@samba:~# <b>host -t SRV _ldap._tcp.example.com</b>
_ldap._tcp.example.com has SRV record 0 100 389 samba.example.com.
</pre>
Kerberos ticket-granting ticket
<pre>
root@samba:~# <b>kinit administrator@EXAMPLE.COM</b>
Password for administrator@EXAMPLE.COM:
Warning: Your password will expire in 41 days on Tue 13 Feb 2018 12:23:01 AM GMT
root@samba:~# <b>klist</b>
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: administrator@EXAMPLE.COM

Valid starting       Expires              Service principal
01/02/2018 01:30:31  01/02/2018 11:30:31  krbtgt/EXAMPLE.COM@EXAMPLE.COM
	renew until 01/03/2018 01:30:26
root@samba:~# <b>smbclient -k -L //samba.example.com</b>
Domain=[EXAMPLE] OS=[Windows 6.1] Server=[Samba 4.2.14-Debian]

	Sharename       Type      Comment
	---------       ----      -------
	netlogon        Disk
	sysvol          Disk
	IPC$            IPC       IPC Service (Samba 4.2.14-Debian)
Domain=[EXAMPLE] OS=[Windows 6.1] Server=[Samba 4.2.14-Debian]

	Server               Comment
	---------            -------

	Workgroup            Master
	---------            -------
	WORKGROUP            SAMBA
</pre>

Integrating Linux into the AD
<pre>
root@samba:~# cat /etc/samba/smb.conf
# Global parameters
[global]
	workgroup = EXAMPLE
	realm = EXAMPLE.COM
	netbios name = SAMBA
	server role = active directory domain controller
	dns forwarder = 10.0.2.3
	idmap_ldb:use rfc2307 = yes
        <b>winbind enum users = yes
        winbind enum groups = yes
        template shell = /bin/bash</b> # These 3 lines added to the default configuration

[netlogon]
	path = /var/lib/samba/sysvol/example.com/scripts
	read only = No

[sysvol]
	path = /var/lib/samba/sysvol
	read only = No
root@samba:~# <b>pam-auth-update</b>
[*] Unix authentication                                                                                                            │
[*] Winbind NT/Active Directory authentication
root@samba:~# <b>cat /etc/nsswitch.conf</b>

passwd:         compat <b>winbind</b>
group:          compat <b>winbind</b>
shadow:         compat
gshadow:        files

hosts:          files dns
networks:       files

protocols:      db files
services:       db files
ethers:         db files
rpc:            db files

netgroup:       nis
root@samba:~# <b>grep use_authtok /etc/pam.d/common-password</b> # We have to remove use_authtok from this file
password	[success=1 default=ignore]	pam_winbind.so <s>use_authtok</s> try_first_pass
root@samba:~# <b>systemctl restart samba-ad-dc</b>
root@samba:~# <b>wbinfo -g</b>
enterprise read-only domain controllers
domain admins
domain users
domain guests
domain computers
domain controllers
schema admins
enterprise admins
group policy creator owners
read-only domain controllers
dnsupdateproxy
root@samba:~# <b>wbinfo -i administrator</b>
administrator:*:0:100::/home/EXAMPLE/administrator:/bin/bash
</pre>
To add a user into the AD:
<pre>
root@samba:~# <b>samba-tool user add naomh --given-name=Naomah --surname=Torin --login-shell=/bin/bash</b>
New Password:
Retype Password:
User 'naomh' created successfully
root@samba:~# <b>samba-tool user list</b>
Administrator
naomh
krbtgt
Guest
root@samba:~# <b>su - naomh</b>
naomh@samba:/$ <b>id</b>
uid=3000017(naomh) gid=100(users) groups=100(users),3000009(BUILTIN\users),3000017(naomh)
</pre>
