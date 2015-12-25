# Linux Networking

## ip vs ifconfig

### ifconfig, up and down
Ethernet networks are called `ethx` (old fashion) or things like `enp0s25` (nowadays)
* `ifconfig -a`
* `ifconfig eth0 192.168.42.42 netmask 255.255.255.0`
* `ifconfig enp0s25 down`
* `ifdown enp0s25`

### ip
The `ip` command is the new tool for configuring the networking interfaces.
You can do many things using it. the `ip a` show will show you the current interfaces and their configurations

<pre>
[vagrant@web ~]$ <b>ip a</b>
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:f6:b0:07 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 77065sec preferred_lft 77065sec
    inet6 fe80::a00:27ff:fef6:b007/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:e2:6c:75 brd ff:ff:ff:ff:ff:ff
    inet 192.168.33.20/24 brd 192.168.33.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fee2:6c75/64 scope link
       valid_lft forever preferred_lft forever
[vagrant@web ~]$ <b>ip r</b> # To see the routing table
default via 10.0.2.2 dev enp0s3  proto static  metric 100
10.0.2.0/24 dev enp0s3  proto kernel  scope link  src 10.0.2.15  metric 100
169.254.0.0/16 dev enp0s8  scope link  metric 1003
192.168.33.0/24 dev enp0s8  proto kernel  scope link  src 192.168.33.20
[vagrant@web ~]$ <b>ip n</b> # show the neigbors
10.0.2.3 dev enp0s3 lladdr 52:54:00:12:35:03 STALE
10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:02 REACHABLE
192.168.33.10 dev enp0s8 lladdr 08:00:27:f0:24:fc STALE
</pre>
Creating network namespaces: NEtwork namespaces resides within the kernel memory
<pre>
[vagrant@web ~]$ <b>sudo ip netns add develpment</b>
[vagrant@web ~]$ <b>ip netns</b>
develpment
</pre>

## Configuring hostnames and client-side DNS
Hostname
<pre>
[vagrant@web ~]$ <b>hostnamectl</b>
   <b>Static hostname: web</b>
         Icon name: computer-vm
           Chassis: vm
        <b>Machine ID: 02ab91023f244febb8c38be42e37cc2d</b>
           Boot ID: 1fabaf6c2ad44197b503d5def2df0026
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-229.el7.x86_64
      Architecture: x86-64
[vagrant@web ~]$ <b>uname -n</b>
web
[vagrant@web ~]$ <b>cat /etc/hostname</b>
web
[vagrant@web ~]$ <b>sudo hostnamectl set-hostname httpd</b>
[vagrant@web ~]$ <b>bash</b>
[vagrant@httpd ~]$
[vagrant@httpd ~]$ <b>cat /etc/hostname</b>
httpd
[vagrant@httpd ~]$ <b>cat /etc/machine-id</b>
02ab91023f244febb8c38be42e37cc2d
</pre>

`/etc/hosts`: When you don't have access to a DNS server

* This is the format: `IP FQDN ALIAS1 [ALIAS2 ...]`
<pre>
[vagrant@web ~]$ <b>cat /etc/hosts</b>
127.0.0.1   web localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.33.10 ac
192.168.33.30 db
[vagrant@web ~]$ <b>ping -c2 ac</b>
PING ac (192.168.33.10) 56(84) bytes of data.
64 bytes from ac (192.168.33.10): icmp_seq=1 ttl=64 time=0.817 ms
64 bytes from ac (192.168.33.10): icmp_seq=2 ttl=64 time=0.660 ms

--- ac ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1004ms
rtt min/avg/max/mdev = 0.660/0.738/0.817/0.083 ms
</pre>

<pre>
[vagrant@web ~]$ <b>getent hosts</b>
127.0.0.1       web localhost localhost.localdomain localhost4 localhost4.localdomain4
127.0.0.1       localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.33.10   ac
192.168.33.30   db
</pre>

<pre>
[vagrant@web ~]$ <b>yum info avahi</b>
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: centos.mirror.iweb.ca
 * extras: mirror.gpmidi.net
 * updates: centos.mirror.iweb.ca
Available Packages
Name        : avahi
Arch        : i686
Version     : 0.6.31
Release     : 17.el7
Size        : 262 k
Repo        : base/7/x86_64
Summary     : Local network service discovery
URL         : http://avahi.org
License     : LGPLv2+
Description : Avahi is a system which facilitates service discovery on
            : a local network -- this means that you can plug your laptop or
            : computer into a network and instantly be able to view other people who
            : you can chat with, find printers to print to or find files being
            : shared. This kind of technology is already found in MacOS X (branded
            : 'Rendezvous', 'Bonjour' and sometimes 'ZeroConf') and is very
            : convenient.
</pre>
To check the order of discovery:
<pre>
[vagrant@web ~]$ <b>grep hosts /etc/nsswitch.conf</b>
#hosts:     db files nisplus nis dns
hosts:      <b>files dns</b>
</pre>

`/etc/resolv.conf`
  * `nameserver`: Domain name servers to lookup
  * `domain`: the default domain to lookup
  * `search`: to search the domain for names that we enter
<pre>
[vagrant@web ~]$ <b>cat /etc/resolv.conf</b>
# Generated by NetworkManager
nameserver 10.0.2.3
</pre>

<pre>
[vagrant@web ~]$ <b>sudo yum install -y bind-utils</b>
[vagrant@web ~]$ <b>dig www.github.com</b>

;;  DiG 9.9.4-RedHat-9.9.4-51.el7_4.1  www.github.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 40294
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 8, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.github.com.			IN	A

;; ANSWER SECTION:
www.github.com.		3600	IN	CNAME	github.com.
github.com.		60	IN	A	192.30.253.113
github.com.		60	IN	A	192.30.253.112

;; AUTHORITY SECTION:
github.com.		119383	IN	NS	ns-1283.awsdns-32.org.
github.com.		119383	IN	NS	ns1.p16.dynect.net.
github.com.		119383	IN	NS	ns-520.awsdns-01.net.
github.com.		119383	IN	NS	ns-421.awsdns-52.com.
github.com.		119383	IN	NS	ns4.p16.dynect.net.
github.com.		119383	IN	NS	ns3.p16.dynect.net.
github.com.		119383	IN	NS	ns2.p16.dynect.net.
github.com.		119383	IN	NS	ns-1707.awsdns-21.co.uk.

;; ADDITIONAL SECTION:
ns-421.awsdns-52.com.	119383	IN	A	205.251.193.165
<b>
;; Query time: 82 msec
;; SERVER: 10.0.2.3#53(10.0.2.3)
;; WHEN: Wed Dec 27 20:40:33 EST 2017
;; MSG SIZE  rcvd: 325</b>
[vagrant@web ~]$ <b>dig +short www.github.com@8.8.8.8</b>
github.com.
192.30.253.113
192.30.253.112
[vagrant@web ~]$ <b>dig +short github.com MX</b>
5 alt1.aspmx.l.google.com.
10 alt4.aspmx.l.google.com.
5 alt2.aspmx.l.google.com.
10 alt3.aspmx.l.google.com.
1 aspmx.l.google.com.
</pre>

## Configuring NTP
Old school:

<pre>
[vagrant@web ~]$ sudo hwclock -w # to sysnc the harware clock with the system clock
[vagrant@web ~]$ sudo hwclock -s # to sync the system clock with the hardware clock
</pre>

### timedatectl
by which we can change both hardware and system time
<pre>
[vagrant@web ~]$ <b>timedatectl</b>
      Local time: Wed 2017-12-27 21:08:08 EST
  Universal time: Thu 2017-12-28 02:08:08 UTC
        RTC time: Thu 2017-12-28 02:08:07
       Time zone: n/a (EST, -0500)
     NTP enabled: n/a
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
To change the time:
<pre>
[vagrant@web ~]$ <b>sudo timedatectl set-ntp false</b>
[vagrant@web ~]$ <b>sudo timedatectl set-time "2000-01-01 23:00:00"</b>
[vagrant@web ~]$ <b>timedatectl</b>
      Local time: Sat 2000-01-01 23:00:02 EST
  Universal time: Sun 2000-01-02 04:00:02 UTC
        RTC time: Sun 2000-01-02 04:00:02
       Time zone: n/a (EST, -0500)
     NTP enabled: n/a
NTP synchronized: no
 RTC in local TZ: no
      DST active: no
 Last DST change: DST ended at
                  Sun 1999-10-31 01:59:59 EDT
                  Sun 1999-10-31 01:00:00 EST
 Next DST change: DST begins (the clock jumps one hour forward) at
                  Sun 2000-04-02 01:59:59 EST
                  Sun 2000-04-02 03:00:00 EDT
</pre>
### chronyd
new time server, time client to NTP. Defualt in RHEL 7
<pre>
[vagrant@web ~]$ <b>sed '/^#/d;/^$/d' /etc/chrony.conf</b>
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
logdir /var/log/chrony
</pre>
<pre>
[vagrant@web ~]$ <b>sudo systemctl start chronyd</b></pre>
<pre>
[vagrant@web ~]$ <b>chronyc tracking</b>
Reference ID    : 2D4FBB0A (tock.no-such-agency.net)
Stratum         : 3
Ref time (UTC)  : Thu Dec 28 02:28:34 2017
System time     : 0.000001194 seconds fast of NTP time
Last offset     : -0.000991792 seconds
RMS offset      : 0.000991792 seconds
Frequency       : 106.176 ppm fast
Residual freq   : +0.000 ppm
Skew            : 483.736 ppm
Root delay      : 0.025486534 seconds
Root dispersion : 0.023743885 seconds
Update interval : 2.1 seconds
Leap status     : Normal
</pre>
<pre>
[vagrant@web ~]$ <b>chronyc sources</b>
210 Number of sources = 4
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^+ white.web-ster.com            2   6    77    58  +1209us[+1454ns] +/-   84ms
^* tock.no-such-agency.net       2   6    77    59  +2085us[ +876us] +/-   30ms
^+ resolver2.skyfiberintern>     2   6    77    60  -3571us[-4776us] +/-   85ms
^+ resolver1.skyfiberintern>     2   6    77    60  -4306us[-5512us] +/-   78ms
</pre>

### NTP Daemon

the traditional NTP server/client
  * `ntpdate`: once off adjustment no matter the sanity of the time
  * `ntpq`: `ntpd -p` shows peers
  * `ntpstat`: shows status but not on Debian. Try `ntpd -c sysinfo`

## Network Services
<pre>
[root@web network-scripts]# systemctl status network
network.service - LSB: Bring up/down networking
   Loaded: loaded (/etc/rc.d/init.d/network)
   Active: active (exited) since Thu 2017-12-28 19:21:39 UTC; 19min ago
  Process: 622 ExecStart=/etc/rc.d/init.d/network start (code=exited, status=0/SUCCESS)

Dec 28 19:21:38 localhost.localdomain network[622]: Bringing up loopback interface:  Could not load file '/etc/sysconfig/network-sc...fg-lo'
Dec 28 19:21:39 localhost.localdomain network[622]: Could not load file '<b>/etc/sysconfig/network-scripts/</b>ifcfg-lo'
Dec 28 19:21:39 localhost.localdomain network[622]: Could not load file '/etc/sysconfig/network-scripts/ifcfg-lo'
Dec 28 19:21:39 localhost.localdomain network[622]: Could not load file '/etc/sysconfig/network-scripts/ifcfg-lo'
Dec 28 19:21:39 localhost.localdomain network[622]: [  OK  ]
Dec 28 19:21:39 localhost.localdomain network[622]: Bringing up interface enp0s3:  [  OK  ]
Dec 28 19:21:39 localhost.localdomain systemd[1]: Started LSB: Bring up/down networking.
Hint: Some lines were ellipsized, use -l to show in full.
</pre>
### Network configuration files
* Redhat-based systems (/etc/sysconfig/network-scripts/)
<pre>
<b>cat /etc/sysconfig/network-scripts/ifcfg-eth0</b>
DEVICE=eth0
ONBOOT=yes
TYPE=Ethernet
IPADDR=192.168.1.10
NETMASK=255.255.255.0
DNS1=4.2.2.4
</pre>
If you want to assign the default GW Add these 2 lines to the file:
<pre>
DEFROUTE="yes"
GATEWAY="192.168.1.1"
</pre>

* Debian-based systems (`/etc/network/`)
<pre>
<b>cat /etc/network/interfaces</b>
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
        address 172.133.76.33
        netmask 255.255.255.0
        network 172.133.76.0
        broadcast 172.133.76.255
        gateway 172.133.76.1
        dns-nameservers 172.133.79.11 172.133.78.11
</pre>

## Routing with CentOS 7
To check if the routing is turned off or on:
<pre>
[vagrant@web ~]$ <b>cat /proc/sys/net/ipv4/ip_forward</b>
0
</pre>
It is off; let's turn the ipv4 routing on:
<pre>
[vagrant@web ~]$ <b>sudo vi /etc/sysctl.conf</b>
<b>net.ipv4.ip_forward=1</b>
[vagrant@web ~]$ <b>sudo sysctl -p</b>
net.ipv4.ip_forward = 1
[vagrant@web ~]$ <b>cat /proc/sys/net/ipv4/ip_forward</b>
1
</pre>

### Enabling NAT

<pre>
[vagrant@web ~]$ <b>sudo iptables -L | wc -l</b>
104
[vagrant@web ~]$ <b>sudo systemctl stop firewalld</b>
[vagrant@web ~]$ <b>sudo iptables -L</b>
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
[vagrant@web ~]$ <b>sudo iptables -t nat -L</b>
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
</pre>
Then be sure your routing is on in `/etc/sysctl.conf`
<pre>
[vagrant@web ~]$ <b>sudo iptables -t nat -A POSTROUTING -o enp0s8 -j MASQUERADE</b>
[vagrant@web ~]$ <b>sudo iptables -t nat -L</b>
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
<b>MASQUERADE  all  --  anywhere             anywhere</b></pre>

## Firewalling with firewalld

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

## Firewalling with iptables toolset
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

## Tunnel traffic

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

## openVPN Server

Installing openVPN
<pre>
[vagrant@web ~]$ <b>sudo yum install epel-release</b>
[vagrant@web ~]$ <b>sudo yum install openvpn easy-rsa -y</b>
</pre>
Configure VPN server
<pre>
[vagrant@web ~]$ <b>sudo cp /usr/share/doc/openvpn-2.4.4/sample/sample-config-files/server.conf /etc/openvpn/</b>
[vagrant@web ~]$ <b>sudo mkdir -p /etc/openvpn/easy-rsa/keys</b>
[vagrant@web ~]$ <b>sudo cp -rf /usr/share/easy-rsa/2.0/* /etc/openvpn/easy-rsa/</b>
[vagrant@web ~]$ <b>vim /etc/openvpn/easy-rsa/vars</b></pre>
Then change these fields:
* `export KEY_NAME="EasyRSA"`
* `export KEY_CN="CommonName"`
<pre>
[root@web ~]# <b>cd /etc/openvpn/easy-rsa/</b>
[root@web easy-rsa]# <b>source ./vars</b>
NOTE: If you run ./clean-all, I will be doing a rm -rf on /etc/openvpn/easy-rsa/keys
[root@web easy-rsa]# <b>source ./clean-all</b>
</pre>

Then create your Certificate of Authority and keys
<pre>
[root@web easy-rsa]# <b>./build-ca</b> # Build a root certificate
[root@web easy-rsa]# <b>./build-key-server web</b> # Make a certificate/private key pair using a locally generated root certificate.
[root@web easy-rsa]# <b>./build-dh</b> # Build Diffie-Hellman parameters for the server side of the SSL/TLS connection.
[root@web easy-rsa]# ls keys/ # To see the different keys that we have created
01.pem  ca.key      EasyRSA.key  index.txt       index.txt.old  serial.old  web.csr
ca.crt  dh2048.pem  EasyRS.key   index.txt.attr  serial         web.crt     web.key
[root@web easy-rsa]# <b>cd keys/</b>
[root@web keys]# cp dh2048.pem ca.crt web.crt web.key /etc/openvpn/
</pre>
No time to generate keys for clients
<pre>
[root@web ~]# <b>cd /etc/openvpn/easy-rsa/</b>
[root@web easy-rsa]# <b>./build-key client</b>
[root@web easy-rsa]# <b>cd keys/</b>
[root@web keys]# ls
01.pem  ca.crt  client.crt  client.key  EasyRSA.key  index.txt       index.txt.attr.old  serial      web.crt  web.key
02.pem  ca.key  client.csr  dh2048.pem  EasyRS.key   index.txt.attr  index.txt.old       serial.old  web.csr
[root@web keys]# <b>cp client.key client.crt /etc/openvpn/</b>
[root@web keys]# <b>systemctl enable openvpn@web</b>
Created symlink from /etc/systemd/system/multi-user.target.wants/openvpn@web.service to /usr/lib/systemd/system/openvpn@.service.
[root@web keys]# <b>systemctl start openvpn@web</b>
</pre>
The on the client:
<pre>
vagrant@ac:~$ <b>mkdir certs</b>
vagrant@ac:~$ <b>cd certs/</b>
vagrant@ac:~/certs$ <b>scp root@192.168.33.20:/etc/openvpn/ca.crt .</b>
ca.crt                                                                                                    100% 1655     1.6KB/s   00:00
vagrant@ac:~/certs$ <b>scp root@192.168.33.20:/etc/openvpn/client.crt .</b>
client.crt                                                                                                100% 5319     5.2KB/s   00:00
vagrant@ac:~/certs$ <b>scp root@192.168.33.20:/etc/openvpn/client.key .</b>
client.key                                                                                                100% 1704     1.7KB/s   00:00
vagrant@ac:~/certs$ <b>ls</b>
ca.crt  client.crt  client.key
vagrant@ac:~/certs$ <b>sudo apt install openvpn</b>
vagrant@ac:~/certs$ <b>cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf .</b>
vagrant@ac:~/certs$ <b>vi client.conf</b>
</pre>
Change the fields to point to your server.

## Monitoring the network

### tracepath
<pre>
[root@web ~]# <b>tracepath 192.168.1.1</b>
 1?: [LOCALHOST]                                         pmtu 1500
 1:  10.0.2.2                                              0.290ms
 1:  10.0.2.2                                              0.342ms
 2:  dlinkrouter                                           2.910ms asymm 64
 3:  192.168.1.1                                           4.812ms reached
     Resume: pmtu 1500 hops 3 back 2
</pre>

### netstat
* `-l`: listening
* `-n`: port number
* `-t`: tcp
* `-u`: udp
<pre>
[root@web ~]# <b>netstat -lun</b>
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
udp        0      0 0.0.0.0:32063           0.0.0.0:*
udp        0      0 0.0.0.0:68              0.0.0.0:*
udp6       0      0 :::51354                :::*
</pre>

* `-i`: interfaces
<pre>
[root@web ~]# <b>netstat -i</b>
Kernel Interface table
Iface      MTU    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
enp0s3    1500    35693      0      0 0         17600      0      0      0 BMRU
enp0s8    1500      248      0      0 0           247      0      0      0 BMRU
enp0s9    1500        0      0      0 0            19      0      0      0 BMRU
lo       65536        8      0      0 0             8      0      0      0 LRU
tun0     1500       411      0      0 0           502      0      0      0 MOPRU
</pre>

### nmap
<pre>
[root@web ~]# <b>nmap scanme.nmap.org</b>

Starting Nmap 6.40 ( http://nmap.org ) at 2017-12-29 03:35 UTC
Nmap scan report for scanme.nmap.org (45.33.32.156)
Host is up (0.090s latency).
Not shown: 996 closed ports
PORT     STATE    SERVICE
<b>22/tcp   open     ssh
25/tcp   filtered smtp
80/tcp   open     http
9929/tcp open     nping-echo</b>

Nmap done: 1 IP address (1 host up) scanned in 11.02 seconds
</pre>
To document your system in a quick and easy manner
<pre>
[root@web ~]# <b>nmap --iflist</b>

Starting Nmap 6.40 ( http://nmap.org ) at 2017-12-29 03:36 UTC
************************INTERFACES************************
DEV    (SHORT)  IP/MASK                     TYPE     UP MTU   MAC
enp0s3 (enp0s3) 10.0.2.15/24                ethernet up 1500  08:00:27:F6:B0:07
enp0s3 (enp0s3) fe80::a00:27ff:fef6:b007/64 ethernet up 1500  08:00:27:F6:B0:07
enp0s8 (enp0s8) 192.168.33.20/24            ethernet up 1500  08:00:27:E2:0B:FA
enp0s8 (enp0s8) fe80::a00:27ff:fee2:bfa/64  ethernet up 1500  08:00:27:E2:0B:FA
enp0s9 (enp0s9) 172.16.0.20/24              ethernet up 1500  08:00:27:6C:53:4C
enp0s9 (enp0s9) fe80::a00:27ff:fe6c:534c/64 ethernet up 1500  08:00:27:6C:53:4C
lo     (lo)     127.0.0.1/8                 loopback up 65536
lo     (lo)     ::1/128                     loopback up 65536

**************************ROUTES**************************
DST/MASK                     DEV    METRIC GATEWAY
172.16.0.0/24                enp0s9 0
192.168.33.0/24              enp0s8 0
10.0.2.0/24                  enp0s3 100
169.254.0.0/16               enp0s8 1003
169.254.0.0/16               enp0s9 1004
0.0.0.0/0                    enp0s3 100    10.0.2.2
::1/128                      lo     0
fe80::a00:27ff:fe6c:534c/128 lo     0
fe80::a00:27ff:fee2:bfa/128  lo     0
fe80::a00:27ff:fef6:b007/128 lo     0
::1/128                      lo     256
fe80::/64                    enp0s3 256
fe80::/64                    enp0s8 256
fe80::/64                    enp0s9 256
ff00::/8                     enp0s3 256
ff00::/8                     enp0s8 256
ff00::/8                     enp0s9 256
</pre>

### ss - socket statistics
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