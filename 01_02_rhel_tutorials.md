# Red Hat Certified System Administrator
* If you are not having previous knowledge about Linux, better to start
from `01_tutorials.md`

* To download Red Hat: https://developers.redhat.com (Minimum RedHat 7.2 needed)
* Alternatively you can use CentOS 7.2 or later

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
  * Use `yum` to install .rpm packages. Because it also checks the dependencies.
    * I installed Google Chrome this way: `yum install '/home/test/Downloads/google-chrome-stable_current_x86_64.rpm'`
  * `rpm -qa bash-completion`
  * `rpm -ivh package.rpm` to get the information package. `v` for verbose. `h` for progressbar.
  * `rpm -U package.rpm`: updates the package.
  * `rpm -e package`: remove the package.
  * `rpm -qpR`: show dependencies.
  * `rpm -qf filename`: very useful to query files. for example `rpm -qf /etc/idmapd.conf`.
  * Downside: you have to know all dependencies of your software. That is why we are lucky having **yum**
  * Nowadays, don't use rpm unless for queries for installed packages
* Repositories:
  * YUM repositories installs rpms. See `yum repolist`. Repositories are packages that are not yet installed.
    * To query repositories: `repoquery -i httpd`, `repoquery -l httpd`
  * `yum search nmap`
  * `yum install nmap`
  * `yum provides */sepolicy`: to look for the content of the packages based on keyword provided.
  * `yum list installed`: to see the list of installed packages.
  * `yum upgrade ‘cal*’ `
  * `yum update`
  * `yum update kernel`
  * `yum info nmap`
  * `yum groups help`: to manage software in more convenient way
  * `yum groups list hidden`
  * `yum reinstall nmap`
  * `yum remove nmap`
  * `yumdownloader httpd`: to download the package but not to install it. you need to install it using `yum install yum-utils`
    * Then run `rpm -qp --scripts httpd-2.4.6-40.el7.rpm` against the package to see what scripts are inside the package.
    Do this whenever you get an RPM from unknown source.
  * YUM is configured in `/etc/yum.conf`.
  * `man yum.conf`

* To add a local repository (e.g DVD ROM Repository)
<pre>
[vagrant@web ~]$ <b>lsblk</b>
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   40G  0 disk
├─sda1            8:1    0  500M  0 part /boot
└─sda2            8:2    0 39.5G  0 part
  ├─centos-root 253:0    0 38.5G  0 lvm  /
  └─centos-swap 253:1    0    1G  0 lvm  [SWAP]
<b>sr0              11:0    1  4.2G  0 rom</b>
[root@web ~]# <b>mount /dev/sr0 /media/</b>
mount: /dev/sr0 is write-protected, mounting read-only
[root@web ~]# <b>df -h /media</b>
Filesystem      Size  Used Avail Use% Mounted on
/dev/sr0        4.3G  4.3G     0 100% /media
[root@web ~]# <b>vi /etc/yum.repos.d/example.repo</b>
[RHELDVD]
baseurl=file:///media
enable=1
gpgcheck=0
[root@web ~]# <b>yum clean all</b>
Loaded plugins: fastestmirror
Repository 'RHELDVD' is missing name in configuration, using id
Cleaning repos: RHELDVD base extras updates
Cleaning up everything
[root@web ~]# <b>yum repolist</b>
Loaded plugins: fastestmirror
Repository 'RHELDVD' is missing name in configuration, using id
<b>RHELDVD</b>                                                                                                              | 3.6 kB  00:00:00
base                                                                                                                 | 3.6 kB  00:00:00
extras                                                                                                               | 3.4 kB  00:00:00
updates                                                                                                              | 3.4 kB  00:00:00
(1/6): RHELDVD/group_gz                                                                                              | 156 kB  00:00:00
(2/6): RHELDVD/primary_db                                                                                            | 3.1 MB  00:00:00
(3/6): base/7/x86_64/group_gz                                                                                        | 156 kB  00:00:00
(4/6): extras/7/x86_64/primary_db                                                                                    | 145 kB  00:00:00
(5/6): base/7/x86_64/primary_db                                                                                      | 5.7 MB  00:00:01
(6/6): updates/7/x86_64/primary_db                                                                                   | 5.2 MB  00:00:01
Determining fastest mirrors
 * base: centos.mirror.vexxhost.com
 * extras: mirror.calgah.com
 * updates: centos.mirror.ca.planethoster.net
repo id                                                           repo name                                                           status
RHELDVD                                                           RHELDVD                                                             3,894
base/7/x86_64                                                     CentOS-7 - Base                                                     9,591
extras/7/x86_64                                                   CentOS-7 - Extras                                                     327
updates/7/x86_64                                                  CentOS-7 - Updates                                                  1,642
repolist: 15,454
</pre>

You can create repositories with `createrepo /repo` command.

**Lab**
1. Create a directory `/downloads` and use `cd /downloads` to make it your current directory
2. Use `yumdownloader c*` to download some packages
3. Configure your server to use the `/downloads` directory as a repository. To start with, use `createrepo /downloads` to make the indexes.
4. Use the appropriate command to list all of the yum groups on your server
5. Find the rpm package that owns the file `/etc/passwd`
6. Use the appropriate command to find out which documentation is installed in the RPM packages containing the `/etc/passwd` file
7. Use the appropriate command to find which package needs to be installed to get the seinfo utility installed on your server

<pre>
[root@web downloads]# <b>yum install yum-utils</b>
[root@web downloads]# <b>yumdownloader c*</b>
[root@web downloads]# <b>yum install createrepo</b>
[root@web downloads]# <b>createrepo /downloads</b>
[root@web downloads]# <b>vi /etc/yum.repos.d/downloaded.repo</b>
[DOWNLOADED]
baseurl=file:///downloads
gpgcheck=0
[root@web downloads]# <b>yum group list hidden</b>
[root@web downloads]# <b>rpm -qf /etc/passwd</b>
setup-2.8.71-5.el7.noarch
[root@web downloads]# <b>rpm -qd setup-2.8.71-5.el7.noarch</b>
/usr/share/doc/setup-2.8.71/COPYING
/usr/share/doc/setup-2.8.71/uidgid
[root@web downloads]# <b>yum provides */seinfo</b></pre>

## Finding the appropriate help
* through `man` pages: There are 9 sections. Important sections are:
  * `1` for user commands. e.g `man -1 intro`
  * `5` for configuration files . e.g `man -5 intro`
  * `7` for different topicse. e.g `man -7 intro`
  * `8` for sysadmin e.g `man -8 intro`
  * If you don't now what command you have to look for, use `man -k` or `apropos`
    * `[hossein@localhost ~]$ man -k lvm`

* adding `--help` after the command
* `/usr/share/doc`

## Processes and jobs
* Process # 1 is `systemd`.
* All other processes are below `systemd`.
* Daemons are processes running in the background.
* Processes ran from bash are called jobs. Users can manage their jobs.
<pre>
[hossein@localhost ~]$ <b>ps aux | head -n5</b>
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.2  0.3 129544  6936 ?        Ss   21:16   0:03 /usr/lib/systemd/systemd --switched-root --system --deserialize 21
root         2  0.0  0.0      0     0 ?        S    21:16   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S    21:16   0:00 [ksoftirqd/0]
root         5  0.0  0.0      0     0 ?        S<   21:16   0:00 [kworker/0:0H]
</pre>
* `USER` the user account is running the process.
* `systemd` has always `PID`=1.
* `?` in `TTY` means background process.
* `S` in `STAT` mean sleeping process.
* `START` indicates when it started.
* `COMMAND`s within `[]` are part of kernel like kernel thread `kthreadd`.
* Switches `aux` after `ps` command doesn't need `-` to respect UNIX-like commands.
<pre>
[hossein@localhost ~]$ <b>top -p 1</b>
top - 21:53:14 up 37 min,  3 users,  load average: 0.00, 0.01, 0.11
Tasks:   1 total,   0 running,   1 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1883560 total,   432076 free,   654836 used,   796648 buff/cache
KiB Swap:  2097148 total,  2097148 free,        0 used.   958420 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
    <b>1 root      20   0  129544   6936   2740 S  0.0  0.4   0:03.77 systemd</b></pre>
Let's generate some load on the system
<pre>
[hossein@localhost ~]$ <b>dd if=/dev/zero of=/dev/null &</b>
[4] 7177
[hossein@localhost ~]$ <b>dd if=/dev/zero of=/dev/null &</b>
[5] 7178
[hossein@localhost ~]$ <b>dd if=/dev/zero of=/dev/null &</b>
[6] 7179
[hossein@localhost ~]$ <b>dd if=/dev/zero of=/dev/null &</b>
[7] 7180
[hossein@localhost ~]$ <b>uptime</b>
 22:09:26 up 53 min,  3 users,  load average: 5.54, 2.43, 0.95
[hossein@localhost ~]$ <b>uptime</b>
 22:10:57 up 54 min,  3 users,  <b>load average: 6.05, 3.41, 1.44</b>
[hossein@localhost ~]$ top | head -n15
top - 22:17:36 up  1:01,  3 users,  <b>load average: 6.05, 5.38, 3.06</b>
Tasks: 179 total,   8 running, 170 sleeping,   1 stopped,   0 zombie
%Cpu(s):  <b>0.0 us,100.0 sy</b>,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1883560 total,   429212 free,   657252 used,   797096 buff/cache
KiB Swap:  2097148 total,  2097148 free,        0 used.   955940 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
 7176 hossein   20   0  107948    608    516 R 25.0  0.0   1:49.31 dd
 7177 hossein   20   0  107948    612    516 R 25.0  0.0   1:47.18 dd
 7174 hossein   20   0  107948    612    516 R 12.5  0.0   1:59.12 dd
 7179 hossein   20   0  107948    608    516 R 12.5  0.0   1:46.47 dd
 7180 hossein   20   0  107948    612    516 R 12.5  0.0   1:46.19 dd
 7178 hossein   20   0  107948    608    516 R  6.2  0.0   1:46.65 dd
    1 root      20   0  129544   6936   2740 S  0.0  0.4   0:03.80 systemd
    2 root      20   0       0      0      0 S  0.0  0.0   0:00.00 kthreadd
</pre>

* `zombie` processes that lost communication from parent process.
* `us` processes by users.
* `sy` processes by the system. here we can see 100 of CPU is occupied by the system.
* By pressing `f`, you can manage the `top` display fields and layout. To quit of this menu press `q`.
to save the layout, use `W` then the configuration will be saved into `~/.toprc`.
* Sending signals to processes:
```
       Signal     Value     Action   Comment
       ──────────────────────────────────────────────────────────────────────
       SIGHUP        1       Term    Hangup detected on controlling terminal
                                     or death of controlling process
       SIGINT        2       Term    Interrupt from keyboard
       SIGQUIT       3       Core    Quit from keyboard
       SIGILL        4       Core    Illegal Instruction
       SIGABRT       6       Core    Abort signal from abort(3)
       SIGFPE        8       Core    Floating point exception
       SIGKILL       9       Term    Kill signal
       SIGSEGV      11       Core    Invalid memory reference
       SIGPIPE      13       Term    Broken pipe: write to pipe with no
                                     readers
       SIGALRM      14       Term    Timer signal from alarm(2)
       SIGTERM      15       Term    Termination signal
       SIGUSR1   30,10,16    Term    User-defined signal 1
       SIGUSR2   31,12,17    Term    User-defined signal 2
       SIGCHLD   20,17,18    Ign     Child stopped or terminated
       SIGCONT   19,18,25    Cont    Continue if stopped
       SIGSTOP   17,19,23    Stop    Stop process
       SIGTSTP   18,20,24    Stop    Stop typed at terminal
       SIGTTIN   21,21,26    Stop    Terminal input for background process
       SIGTTOU   22,22,27    Stop    Terminal output for background process
```
The most famous one are 9 and 15
* `9` to kill the process.
* `15` to terminate the process gracefully.
* However, you can use `kill` command to the other 30 different signals.
  * `kill-9 7179`.
  * Always first try signal 15 before trying signal 9 to kill processes.
  * Alternatively, you can use `killall dd` or `pidof dd` for example.
* You can also send signals from `top` command, by pressing `k`.

### Priorities and niceness
Processes can be run into real time or normal
* Real time processes are for kernel and some applications.
* Priority is by default = 20
* Nice can vary from -20 to +19 and it affects the priority.
* The lower the priority the nicer. For example if you give a process a
nice = -10, the priority decreases to 10 which is better that when you give a process
a priority = 30.
* You can see the nice value in `ps -l` or in `top` command the `NI` field.
* you can run commands with `nice` command and `-n` switch (for nice):
  * `nice -n -20 echo "I am running!"`
  * `nice ping 4.2.2.4`
  * `nice -n 19 tar -cf myarchive.tar *`
* The `renice` command can change the niceness of running processes:
<pre>
[hossein@localhost ~]$ <b>!dd
dd if=/dev/zero of=/dev/null &</b>
[1] 6238
[hossein@localhost ~]$ <b>dd if=/dev/zero of=/dev/null &</b>
[2] 6239
[hossein@localhost ~]$ <b>dd if=/dev/zero of=/dev/null &</b>
[3] 6240
[hossein@localhost ~]$ <b>dd if=/dev/zero of=/dev/null &</b>
[4] 6241
[hossein@localhost ~]$ <b>top | head -n15</b>
top - 22:48:45 up  1:32,  3 users,  load average: 3.39, 1.25, 0.95
Tasks: 175 total,   5 running, 170 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,100.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1883560 total,   357068 free,   675644 used,   850848 buff/cache
KiB Swap:  2097148 total,  2097148 free,        0 used.   936024 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                  GROUP      USED
 6241 hossein   20   0  107948    608    516 R 37.5  0.0   0:24.87 dd                                                       hossein     608
 6239 hossein   20   0  107948    612    516 R 25.0  0.0   0:26.55 dd                                                       hossein     612
 6238 hossein   20   0  107948    612    516 R 18.8  0.0   0:35.27 dd                                                       hossein     612
 6240 hossein   20   0  107948    612    516 R 18.8  0.0   0:26.09 dd                                                       hossein     612
 6262 hossein   20   0  157716   2208   1516 R  6.2  0.1   0:00.01 top                                                      hossein    2208
    1 root      20   0  129544   6936   2740 S  0.0  0.4   0:03.83 systemd                                                  root       6936
    2 root      20   0       0      0      0 S  0.0  0.0   0:00.00 kthreadd                                                 root          0
    3 root      20   0       0      0      0 S  0.0  0.0   0:00.42 ksoftirqd/0                                              root          0
[hossein@localhost ~]$ <b>sudo renice -n  -20 6239</b>
[sudo] password for hossein:
6239 (process ID) old priority 0, new priority -20
[hossein@localhost ~]$ <b>top | head -n15</b>
top - 22:52:21 up  1:36,  3 users,  load average: 4.54, 2.88, 1.66
Tasks: 175 total,   5 running, 170 sleeping,   0 stopped,   0 zombie
%Cpu(s): 20.0 us, 80.0 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1883560 total,   356944 free,   675788 used,   850828 buff/cache
KiB Swap:  2097148 total,  2097148 free,        0 used.   935904 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                  GROUP      USED
 6239 hossein    0 -20  107948    612    516 R 41.7  0.0   1:30.37 dd                                                       hossein     612
 6240 hossein   20   0  107948    612    516 R  0.7  0.0   1:16.41 dd                                                       hossein     612
 6241 hossein   20   0  107948    608    516 R  0.7  0.0   1:15.16 dd                                                       hossein     608
 6316 hossein   20   0  157716   2208   1516 R  0.7  0.1   0:00.02 top                                                      hossein    2208
    1 root      20   0  129544   6936   2740 S  0.0  0.4   0:03.84 systemd                                                  root       6936
    2 root      20   0       0      0      0 S  0.0  0.0   0:00.00 kthreadd                                                 root          0
    3 root      20   0       0      0      0 S  0.0  0.0   0:00.42 ksoftirqd/0                                              root          0
    5 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 kworker/0:0H                                             root          0
</pre>
* In reality, these tools (nice and renice) are not a good practice.

## Checking CPU, RAM, OS version
### CPU
* `cat /proc/cpuinfo`
* `lscpu`
* `getconf LONG_BIT`: Output: 64
* `uname -i`

### Memory
* `cat /proc/meminfo`
* `free -m`: m for Megabytes

## Basic commands
* `cal`
* `cal 2018`
* `date`
  * `date +%R`: 02:36
  * `date +%r`: 02:36:16 AM
  * `date +%F`: 2018-01-04
* `alias`
  * `[root@localhost ~]# alias ls`
  * to skip alias use `\` for example `\ls`
* `find`: important switches are `-name`, `-user`, and `-size`:
    <pre>
    [root@localhost ~]# <b>find / -size +100M -exec ls -lh {} \; 2>/dev/null</b>
    -r--------. 1 root root 128T Jan  6 17:52 /proc/kcore
    -rw-r--r--. 1 root root 102M Jan  5 21:08 /usr/lib/locale/locale-archive
    </pre>

#### hostnamectl
<pre>
[vagrant@web ~]$ <b>hostnamectl</b>
   Static hostname: web
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 02ab91023f244febb8c38be42e37cc2d
           Boot ID: ba222e4f1c8248c397a188931cf4d885
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-229.el7.x86_64
      Architecture: x86_64
[vagrant@web ~]$ <b>hostnamectl set-hostname web1.redhathelp.com</b>
[vagrant@web ~]$ <b>hostname</b>
web1.redhathelp.com
</pre>

#### timedatectl
<pre>
[vagrant@web ~]$ <b>timedatectl</b>
      Local time: Thu 2018-01-04 02:40:17 UTC
  Universal time: Thu 2018-01-04 02:40:17 UTC
        Timezone: UTC (UTC, +0000)
     NTP enabled: n/a
NTP synchronized: no
 RTC in local TZ: no
      DST active: n/a
[vagrant@web ~]$ <b>timedatectl set-timezone EST</b>
[vagrant@web ~]$ <b>date</b>
Wed Jan  3 21:42:14 EST 2018
</pre>
To get time from NTP use `timedatectl set-ntp yes`.
This command starts `chronyd.service` which is a replacement of ntpd in new RHEL servers.
The configuration is in `/etc/chrony.conf`

To synchronise hardware clock according to the system time use `hwclock --systohc`
#### uptime
<pre>
[vagrant@web ~]$ <b>uptime</b>
 21:43:11 up 26 min,  1 user,  load average: 0.00, 0.01, 0.04
 </pre>

### Creating a file
* `touch`:
The touch command with no option will update the modification date of a
file to the current time (will create a file if it doesn’t exist)
  * `-t`: `touch -t 200908121510.59 f3`
  * `-d`: `touch -d "2 days ago 12:00" f3`
  * `-r`: `touch -r f1 f3` #changes timestamp of f3 the same as f1
* `fallocate`:
  * `fallocate -l 1g test`: to create a 1-gig-size file
* or with `dd`: `dd if=/dev/zero of=file bs=10M count=100`

### tar

to archive all files in the current directory:
* `tar cvf myarchive.tar *`
  * Use `tar tvf myarchive.tar` to show what is inside

then we can zip this archive:
* `gzip myarchive.tar.gz`. To unzip, use `gunzip`.
* or
  * With `bzip2` which is more effective.  To unzip, use `bunzip2`.
* or we could archive and zip files altogether (z switch)
  * `tar cvf myarchive.tar.gz -z *` to zip with `gzip`.
  * `tar cvf bzipfiles.tar.bz -j *` to zip with `bzip2`.
to extract:
* `tar xvf myarchive.tar.gz`

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

#### grep
* `man 7 grep`
* `-v`: ! (selects non-matching lines)
* `-e` allows you to llok for multiple expressions.
* `-A 3`: print the line matched and 3 lines after
* `-B 3`: similar to `-A`, but print 3 lines before
<pre>
[root@localhost ~]# <b>grep -v -e '^#' -e '^$' /etc/sysconfig/sshd -B3 -A1</b>
# Do not change this option unless you have hardware random
# generator and you REALLY know what you are doing

<b>SSH_USE_STRONG_RNG=0</b>
# SSH_USE_STRONG_RNG=1
</pre>

#### awk
* By default, `awk` prints all the lines that match pattern.
* You can instruct `awk` to print some columns or fields.

<pre>
[root@localhost ~]# <b>awk -F : '{print $1 }' /etc/passwd | sort</b>
abrt
adm
avahi
bin
chrony
colord
daemon
dbus
...
</pre>

#### sed
I have so many examples in `01_tutorials.md`.

**Lab**

1. Use `head` and `tail` to display the fifth line of the file `/etc/passwd`
2. Use `sed` to display the fifth line of the file `/etc/passwd`
3. Use `awk` in a pipe to filter the first column out of the results of the command `ps aux`
4. Use `grep` to show the names of all files in `/etc` that have lines starting with the text `root`
5. Use `grep` to show all lines from all files in `/etc` that contain exactly 3 characters
6. Use `grep` to find all files that contain the string *alex*, in `/`

<pre>
[root@localhost ~]# <b>head -n5 /etc/passwd | tail -n1</b>
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
[root@localhost ~]# <b>sed -n 5p /etc/passwd</b>
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
[root@localhost ~]# <b>ps aux | awk '{print $1}'</b>
[root@localhost ~]# <b>grep -l '^root*' /etc/* 2>/dev/null</b> # -l to show the file names not all the matching lines
[root@localhost ~]# <b>grep -l '^...$' /etc/* 2>/dev/null</b>
[root@localhost ~]# grep -R 'alex' / # -R to recursive
</pre>


## File system hierarchy standard (FHS) and file management
* `/bin`, `/sbin`, `/lib`, and `/lib64` are linked to the `/usr` directory (like Program Files in Windows).
* information is written by kernel into `/proc`.
* See `man hier`.

## User account administration
In Linux, there are 2 kinds of users
* Services (1-999)
* People (>1000)
  * root (0)
  * login: `/bin/bash`
    * Home directory comes from `/etc/skel`
  * `/sbin/nologin`
<pre>
[vagrant@web ~]$ <b>sudo grep vagrant /etc/shadow</b>
vagrant:$6$RltuSxMxR8tL0So8$GGrz5uPH6a90PGPaJu5sE7bkkHHhOLJmbf2860RC1QedQ3AtoAqW.QU09fncmRnpIM6dX1cDtWN3RGg3FPgYD/:16714:0:99999:7:::
[vagrant@web ~]$ <b>chage -l vagrant</b>
Last password change					: Oct 06, 2015
Password expires					: never
Password inactive					: never
Account expires						: never
Minimum number of days between password change		: 0
Maximum number of days between password change		: 99999
Number of days of warning before password expires	: 7
</pre>

<pre>
[vagrant@web ~]$ id vagrant
uid=1000(vagrant) gid=1000(vagrant) groups=1000(vagrant)
</pre>

To add a user, use `useradd`. To modify a user account use `usermod`:
<pre>
[vagrant@web ~]$ <b>sudo useradd ryan</b>
[vagrant@web ~]$ <b>tail -n1 /etc/passwd</b>
ryan:x:1002:1002::/home/ryan:/bin/bash
[vagrant@web ~]$ <b>sudo usermod -c adminuser ryan</b>
[vagrant@web ~]$ <b>!ta</b>
tail -n1 /etc/passwd
ryan:x:1002:1002:adminuser:/home/ryan:/bin/bash
[vagrant@web1 ~]$ <b>sudo passwd ryan</b>
Changing password for user ryan.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
[vagrant@web1 ~]$ <b>su - ryan</b> # With - when you log in as ryan you will be placed into the home directory for ryan.
Password:
Last login: Wed Jan  3 22:59:06 EST 2018 on pts/0
[ryan@web1 ~]$ <b>pwd</b>
/home/ryan
</pre>
Some other important arguments for `usermod`
* `-L` to lock an account
* To unlock an account: `-U`
* To append to other group(s): `-aG`
* `-s`: by default when you create an account, the default shell would be `/bin/bash`.
But you may change the `/sbing/nologin` for system users for example.
* some of these options are available when you use `useradd`.

`passwd` is to update the user's password (token). Mostly I use it to change a user's
password. But 2 options I like are  `--stdin` to determine the user's password by a script.
`-S` to report password status.

For all other options I prefer to use `chage` command.
<pre>
[root@rhel ~]# <b>echo Password1 | passwd bran --stdin</b>
Changing password for user bran.
passwd: all authentication tokens updated successfully.
[root@rhel ~]# <b>passwd -S bran</b>
bran PS 2018-01-06 0 99999 7 -1 (Password set, SHA512 crypt.)
[root@rhel ~]# <b>chage -l bran</b>
Last password change					: Jan 07, 2018
Password expires					: never
Password inactive					: never
Account expires						: never
Minimum number of days between password change		: 0
Maximum number of days between password change		: 99999
Number of days of warning before password expires	: 7
</pre>

#### Configuration file for user management
* `/etc/default/useradd`: when adding a user, default value come from this file.
    <pre>
    # useradd defaults file
    GROUP=100
    HOME=/home
    INACTIVE=-1
    EXPIRE=
    SHELL=/bin/bash
    SKEL=/etc/skel
    CREATE_MAIL_SPOOL=yes
    </pre>
* `/etc/login.defs`: If there is any conflicts with `/etc/default/useradd`, this file (`/etc/login.defs`) wins
    <pre>
    MAIL_DIR	/var/spool/mail
    PASS_MAX_DAYS	99999
    PASS_MIN_DAYS	0
    PASS_MIN_LEN	5
    PASS_WARN_AGE	7
    UID_MIN                  1000
    UID_MAX                 60000
    SYS_UID_MIN               201
    SYS_UID_MAX               999
    GID_MIN                  1000
    GID_MAX                 60000
    SYS_GID_MIN               201
    SYS_GID_MAX               999
    CREATE_HOME	yes
    UMASK           077
    USERGROUPS_ENAB yes
    ENCRYPT_METHOD SHA512
    </pre>
* `/etc/skel/`: copied to the home directory of the user
* `/etc/passwd`
* `/etc/shadow`: help is in `man 5 shadow`
* `/etc/group`

#### Create a home directory if user account exists but with no home directory:

<pre>
[root@rhel ~]# <b>cp -a /etc/skel/ /home/blanid</b>
[root@rhel ~]# <b>sudo chown blanid:blanid /home/blanid/</b>
[root@rhel ~]# <b>ls -ld /home/blanid/</b>
drwxr-xr-x. 4 <b>blanid blanid</b> 90 Jan  7 17:41 /home/blanid/
</pre>

#### Delete a user account:
<pre>
[vagrant@web1 ~]$ <b>sudo userdel -rf uwe</b></pre>

### Groups
* Primary group
  * Mandatory
  * Private: use himself can grant access to his group to someone else
  * Ownership
* Secondary group
  * Additional permissions
  * Special functions
<pre>
[vagrant@web1 ~]$ <b>groupadd ops</b>
[vagrant@web1 ~]$ <b>sudo usermod -G ops ryan</b>
[vagrant@web1 ~]$ <b>grep ryan /etc/group</b>
ryan:x:1002:
ops:x:1003:ryan
</pre>


#### Remove a user from a group
<pre>
[vagrant@web1 ~]$ <b>sudo gpasswd -d user group</b></pre> 

#### Manage users and groups through GUI
<pre>
[root@client1 ~]# <b>yum install system-config-users</b>
[root@client1 ~]# <b>system-config-users</b></pre>

![users](https://user-images.githubusercontent.com/31813625/35785467-9539f5ae-09ee-11e8-9c43-f9c7da7f172d.png)

You can also use manage what you can do with `chage` and `passwd` command with GUI.

These commands are the same:
<pre>
[root@client1 ~]# <b>chage -d 0 user1</b>
[root@client1 ~]# <b>passwd -e user1</b>
Expiring password for user user1.
passwd: Success
[root@client1 ~]# chage -l user1
<b>Last password change					: password must be changed
Password expires					: password must be changed
Password inactive					: password must be changed</b>
Account expires						: never
Minimum number of days between password change		: 0
Maximum number of days between password change		: 99999
Number of days of warning before password expires	: 7

</pre>
## File and folder permission
* `r` on directory means `ls`
* `w` on directories means create or delete
* `x` on directories means opening the directory (`cd`)
* How to set ownership lab:
    <pre>
    [root@rhel blanid]# <b>cd /home/blanid/</b>
    [root@rhel blanid]# <b>mkdir sales/</b>
    [root@rhel blanid]# <b>ls -l</b>
    total 0
    drwxr-xr-x. 2 <b>root root</b> 6 Jan  7 19:21 sales
    [root@rhel blanid]# <b>groupadd sale_group</b>
    [root@rhel blanid]# <b>chown -R blanid:sale_group sales/</b> # -R for recursive. For files we don't use -R
    [root@rhel blanid]# <b>ls -l</b>
    total 0
    drwxr-xr-x. 2 <b>blanid sale_group</b> 6 Jan  7 19:21 sales
    </pre>
* How to change the permission?
    <pre>
    [vagrant@web1 ~]$ <b>chmod u=rwx,g=rw,o=- test</b>
    [vagrant@web1 ~]$ <b>ls -lh</b>
    total 1.0G
    -<b>rwxrw----</b>. 1 vagrant vagrant 1.0G Jan  3 22:38 test
    [vagrant@web1 ~]$ <b>chmod o+r test</b>
    [vagrant@web1 ~]$ <b>ls -lh</b>
    total 1.0G
    -rwxrw-<b>r</b>--. 1 vagrant vagrant 1.0G Jan  3 22:38 test
    [vagrant@web1 ~]$ <b>chmod 661 test</b>
    [vagrant@web1 ~]$ <b>ls -lh</b>
    total 1.0G
    -rw-rw---x. 1 vagrant vagrant 1.0G Jan  3 22:38 test
    </pre>

#### umask
* For root is `0022`
* For ordinary users is `0002`
<pre>
[root@rhel ~]# <b>umask</b>
<b>0022</b>
[root@rhel ~]# <b>su - blanid</b>
Last login: Sun Jan  7 17:37:34 EST 2018 on pts/0
[blanid@rhel ~]$ <b>umask</b>
<b>0002</b></pre>
Where are 022 and 002 comming from? from `/etc/bashrc` and `/etc/profile`
<pre>
[blanid@rhel ~]$ <b>grep umask /etc/* 2>/dev/null</b>
<b>/etc/bashrc:    # By default, we want umask to get set. This sets it for non-login shell.</b>
/etc/bashrc:       umask 002
/etc/bashrc:       umask 022
/etc/csh.cshrc:    umask 002
/etc/csh.cshrc:    umask 022
<b>/etc/profile:# By default, we want umask to get set. This sets it for login shell</b>
/etc/profile:    umask 002
/etc/profile:    umask 022
</pre>
Users can change their default umask in `~/.bash_profile`
<pre>
[blanid@rhel ~]$ <b>vi ~/.bash_profile</b>
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
	. ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/.local/bin:$HOME/bin

export PATH
<b>umask 027</b>
[blanid@rhel ~]$ <b>exit</b>
logout
[root@rhel ~]# <b>su - blanid</b>
Last login: Sun Jan  7 19:56:03 EST 2018 on pts/0
[blanid@rhel ~]$ <b>ls -l newfile</b>
-<b>rw-r-----</b>. 1 blanid blanid 0 Jan  7 19:56 newfile
</pre>

### Special permissions
* SUID (`u+s`) 4: on files means to run as owner. On directories has no meaning.
    <pre>
    [blanid@rhel ~]$ <b>ls -l `which passwd`</b>
    -rw<b>s</b>r-xr-x. 1 root root 27832 Jan 29  2014 /bin/passwd
    </pre>
* SGID (`g+s`) 2: on files means to run as group owner. On directories means inherit group owner.
    <pre>
    [root@rhel ~]# <b>mkdir /sales</b>
    [root@rhel ~]# <b>ls -ld /sales</b>
    drwxr-xr-x. 2 root root 6 Jan  7 20:11 /sales
    [root@rhel ~]# <b>groupadd sale_group</b>
    [root@rhel ~]# <b>chown :sale_group /sales/</b>
    [root@rhel ~]# <b>ls -ld /sales</b>
    drwxr-xr-x. 2 root sale_group 6 Jan  7 20:11 /sales
    [root@rhel ~]# <b>chmod g+w /sales</b>
    [root@rhel ~]# <b>ls -ld /sales</b>
    drwxrwxr-x. 2 root sale_group 6 Jan  7 20:11 /sales
    [root@rhel ~]# <b>usermod -aG sale_group blanid</b>
    [root@rhel ~]# su - blanid
    [blanid@rhel ~]$ <b>touch /sales/commit1</b>
    [blanid@rhel ~]$ <b>ls -l /sales/commit1</b>
    -rw-r-----. 1 blanid <b>blanid</b> 0 Jan  7 20:17 /sales/commit1
    [blanid@rhel ~]$ <b>exit</b>
    logout
    [root@rhel ~]# <b>chmod g+s /sales</b>
    [root@rhel ~]# <b>su - blanid</b>
    Last login: Sun Jan  7 20:17:46 EST 2018 on pts/0
    [blanid@rhel ~]$ <b>touch /sales/commit2</b>
    [blanid@rhel ~]$ <b>ls -l /sales/commit2</b>
    -rw-r-----. 1 blanid <b>sale_group</b> 0 Jan  7 20:21 /sales/commit2
    </pre>
* Sticky (`+t`) 1: Means only delete files and directories if the user is owner of the file or owner of directory.
(`ls -ld /sales`). (We apply on the parent directory. e.g `chmod +t /sales`).

### ACLs or the spacial permissions
* Files and directories have permission sets for the owner of the file,
the group associated with the file, and all other users for the system.
However, these permission sets have limitations. For example, different
permissions cannot be configured for different users. The ACLs were implemented.
* Multiple owners.
* Default ACL to substitute `umask`.
* There is an utility in Red Hat `facl` which sets access control lists of files and directories.
on the command line.
* A sequence of command is followed by another sequence of commands.

**Lab:**
<pre>
[root@web ~]# useradd blanid
[root@web ~]# useradd bran
[root@web ~]# groupadd admin
[root@web ~]# mkdir /tmp/project
[root@web ~]# chmod 777 /tmp/project
[root@web ~]# ls -ld /tmp/project/
drwxrwxrwx. 2 root root 6 Jan  4 23:47 /tmp/project/
</pre>
Scenario 1:
* *blanid* with `rwx` permission
* *bran* with no permission
* group *admin* with `rwx` permission
<pre>
[root@web ~]# <b>setfacl -R -m u:blanid:rwx /tmp/project/</b>
[root@web ~]# <b>!ls
ls -ld /tmp/project/</b>
drwxrwxrwx<b>+</b> 2 root root 6 Jan  4 23:47 /tmp/project/
[root@web ~]# <b>setfacl -R -m u:bran:0 /tmp/project/</b>
[root@web ~]# <b>setfacl -R -m g:admin:rwx /tmp/project/</b>
[root@web ~]# <b>getfacl /tmp/project/</b>
getfacl: Removing leading '/' from absolute path names
# file: tmp/project/
# owner: root
# group: root
user::rwx
<b>user:blanid:rwx
user:bran:---</b>
group::rwx
<b>group:admin:rwx</b>
mask::rwx
other::rwx
</pre>
Scenario 2:

Note that when we apply ACL, it affects current files and directories.
If you want ACL to affect the files we are creating later, we have to run
the command once again but with different options. (`d:` stands for default).

Remember,  ACL should be applied twice. Once is without `d:` and once with `d:`. For example:
<pre>
[root@rhel /]# <b>touch sales/file1</b>
[root@rhel /]# <b>getfacl /sales/file1</b>
getfacl: Removing leading '/' from absolute path names
# file: sales/file1
# owner: root
# group: sales_group
user::rw-
group::r--
other::r--

[root@rhel /]# <b>setfacl -R -m g:sales_group:rwx /sales/</b>
[root@rhel /]# <b>getfacl /sales/file1</b>
getfacl: Removing leading '/' from absolute path names
# file: sales/file1
# owner: root
# group: sales_group
user::rw-
group::r--
<b>group:sales_group:rwx</b>
mask::rwx
other::r--

[root@rhel /]# <b>setfacl -m d:g:sales_group:rwx /sales/</b>
[root@rhel /]# <b>touch /sales/file3</b>
[root@rhel /]# <b>getfacl /sales/file3</b>
getfacl: Removing leading '/' from absolute path names
# file: sales/file3
# owner: root
# group: sales_group
user::rw-
group::rwx			#effective:rw-
<b>group:sales_group:rwx		#effective:rw-</b>
mask::rw-
other::r--
[root@rhel /]# <b>getfacl /sales/</b>
getfacl: Removing leading '/' from absolute path names
# file: sales/
# owner: root
# group: sales_group
# flags: -s-
user::rwx
group::rwx
group:sales_group:rwx
mask::rwx
other::r-x
default:user::rwx
default:group::rwx
<b>default:group:sales_group:rwx</b>
default:mask::rwx
default:other::r-x
</pre>
To remove an ACL:
<pre>
[root@web ~]# <b>setfacl -x g:admin /tmp/project/</b></pre>

## Connecting to RHEL server
* Local sessions:
  * `tty`: is virtual console. `tty1` is for `graphical.target`.
  * You can have up to 6 local sessions using tty.
  * `[root@localhost ~]# systemctl isolate multi-user.target`.
    * switch between virtual consoles using 'Alt+F{2-7}'.
    * Back to the `graphical.target` using `[root@localhost ~]# systemctl isolate graphical.target`.
  * su: switch between user accounts without logging out.
    * if you add a `-`, you will then have the same env as the target user.
    * e.g `su - bran` and `su blanid`.
* Remote sessions
  * SSH
    1. Client is issuing command `ssh server_address`
    2. It will connects to **sshd** service on the server
    3. SSH service sends back it's public key (`/etc/ssh/ssh_host.pub`) to the client
    4. Client asks you if you want to continue because you are going to connect to the server for the first time
    5. If you answer yes then the public key of the server is written to `~/.ssh/known_hosts` of the client
    6. If the user doesn't want to enter the password each time connecting to the server, he/she can generate his/her key pair using `ssh-keygen` command and copy his/her public key `~/.ssh/id_rsa.pub` to the his/her account on the server as an authorized key `~/.ssh/authorized_keys`
      * They can copy their public key using `ssh-copy-id server_address` command or copy and paste method.
      * This concept is being run usually for automation tasks between 2 machines.

**Lab**

1. Open a virtual console and log in on that console as user root.
2. Use `chvt` command to activate tty2. `chvt 2`. Verify with command `w`.
3. On this terminal, use `su` to become user *bran*.
4. Establish an SSH session to log in to localhost as user root. Type `exit` to close the SSH session.
5. Back in the virtual console, create an SSH key pair for user *bran*.
6. Use the SSH key pair to log in to localhost without having to enter a password.
7. Try using `sudo fdisk /dev/sda` as user *bran*. Is this successful? Explain why.

## Networking

### Network Device Naming
* In Red Hat Enterprise Linux, **udev** supports a number of different naming schemes.
* **BIOS Naming or `biosdevname=1`**: based on hardware properties
  * On-board net devices get named `em<port>[_<virtual instance>]`
  * Add-on NICs devices get named `p<slot>p<port>[_<virtual instance>]`
  * To swap `eth0` with any interface you have: `biosdevname -i eth0`.
  If conditions are right, you should get something along the lines of `em1` or `p1p1`.`
  * Downside: it doesn't cater for a wider range of interface types/devices.
  It refused to name my usb ethernet dongle.
* **systemd udev**
  * firmware/bios-provided for on-board devices: `enoX`
    * `en` for ethernet
    * `sl` for serial
    * `wl` for wireless
    * `ww` for LTE,...
    * `o` for on-board
  * firmware/bios-provided for pci-express hotplug slot: `ens1` wherein `s` for slot
  * physical/geographical location for PCI devices: `enp2s0`: Ethernet interface on bus 2 slot 0
    * MAC address: `enx78e7d1ea46da`
  * systemd-udev is now standard on most big name distros
* **classical `ethx` naming**
  * To get classical naming, use `biosdevname=0` and `net.ifnames=0` GRUB boot options
* **Logical Naming**
  * `.<vlan>` and `:<alias>`

To change IP address on runtime:
<pre>
[root@rhel /]# <b>ip a a 172.16.0.1/16 dev enp0s3</b>
[root@rhel /]# <b>ip a sh enp0s3</b>
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:66:a4:b5 brd ff:ff:ff:ff:ff:ff
    inet 172.16.0.1/16 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe66:a4b5/64 scope link
       valid_lft forever preferred_lft forever
</pre>
To have a persistent change you have to edit this file for example for `enp0s3`:
<pre>
[root@rhel ~]# cat /etc/sysconfig/network-scripts/ifcfg-enp0s3
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp0s3
UUID=e2ac014a-2064-4cd6-9455-ef7c6b5b9782
DEVICE=enp0s3
ONBOOT=no
</pre>
You will be having 4 options:
* Edit the file directly and then use `systemctl restart network`
* Or use of NetworkManager-wait-online.service
 * Use `nmcli`: use tab completion to use the command
 * Use `nmtui`: Text User Interface for controlling NetworkManager
 * GUI

#### nmcli
<pre>
[root@rhel ~]# <b>nmcli device show enp0s3</b>
GENERAL.DEVICE:                         enp0s3
GENERAL.TYPE:                           ethernet
GENERAL.HWADDR:                         08:00:27:66:A4:B5
GENERAL.MTU:                            1500
GENERAL.STATE:                          100 (connected)
GENERAL.CONNECTION:                     enp0s3
GENERAL.CON-PATH:                       /org/freedesktop/NetworkManager/ActiveConnection/4
WIRED-PROPERTIES.CARRIER:               on
IP4.ADDRESS[1]:                         172.16.0.1/16
IP4.GATEWAY:                            --
IP6.ADDRESS[1]:                         fe80::a00:27ff:fe66:a4b5/64
IP6.GATEWAY:                            --
[root@rhel ~]# <b>nmcli device status</b>
DEVICE      TYPE      STATE      CONNECTION
virbr0      bridge    connected  virbr0
enp0s3      ethernet  connected  enp0s3
enp0s8      ethernet  connected  System enp0s8
lo          loopback  unmanaged  --
virbr0-nic  tun       unmanaged  --
[root@rhel ~]# nmcli connection add ifname enp0s3 type ethernet ipv4.addresses 10.10.10.10/23 gw4 10.10.10.1
Connection 'ethernet-enp0s3' (67657a8a-15d3-42c5-aef1-cb01ddff22da) successfully added.
</pre>

### Routing
* To add a route in runtime, use `ip r add`
* To add a route persistently:
<pre>
[root@rhel ~]# <b>cat /etc/sysconfig/network-scripts/route-enp0s8</b>
ADDRESS0=192.168.53.0
NETMASK0=255.255.255.0
[root@rhel ~]# ip r
default via 192.168.33.1 dev enp0s8 proto static metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100
172.16.0.0/24 dev enp0s3 proto kernel scope link src 172.16.0.1 metric 100
192.168.33.0/24 dev enp0s8 proto kernel scope link src 192.168.33.40 metric 100
<b>192.168.53.0/24 dev enp0s8 proto static scope link metric 100</b>
192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1
</pre>

## File system and storage management
* Disk layout:
  * The area to store metadata
  * The area to store data: partition
* Firmware:

    |  | BIOS | UEFI |
    | --- | --- | --- |
    | Bit level | 16-bit | 32 or 64-bit |
    | Maximum Drive Size | 2TB (with MBR) | > 2 TB (MBR or GPT) |
    | Maximum number of partitions | 4 (or 15 primary and logical) | 128 |
    | Disk management utility in Linux | fdisk | gdisk |
    | CPU Support | CISCO | CISC and RISC
    | Modularity | no | yes |
    | Time Zone and Daylight Saving Support | no | yes |
    | Boot from SAN or Network | extensions on adapters | yes |
* `cat /proc/partitions`
* Create a partition with `gdisk` or `fdisk`
* Format the partition using `mkfs.` then double press tab key to see
the different format options
* Mount the partition: in DOS/Windows each partition gets a drive letter.
You then need to use the drive letter to refer to files and directories.
* In Red Hat EL each partition is used to form part of the storage, necessary
to support a single set of files and directories.
  * This is done by associating a partition with a directory through a process
  known as *mounting*.

### Mounting
You need 3 things for mounting:
* device
* directory
* filesystem

1. Temporarily mounting
<pre>
[vagrant@web ~]$ <b>sudo mkdir /data/</b>
[vagrant@web ~]$ <b>sudo mount /dev/sdb1 /data/</b></pre>
In corporate environment as the storage topology is changing, we usually mount the device by its label.
You can also refer to UUID from command `blkid`
<pre>
[vagrant@web ~]$ <b>sudo mount LABEL=data /data</b></pre>
2. Permanently mounting
<pre>
[vagrant@web ~]$ <b>tail -n1 /etc/fstab</b>
/dev/sdb1	/data	xfs	defaults	0 0
[vagrant@web ~]$ <b>sudo partprobe /dev/sdb1</b> # mount -a also works
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

* To make a swap partition use the following commands:
  1. `mkswap /dev/sdc1`
  2. copy the UUID and add it to `/etc/fstab`: `UUID=xxx swap swap defaults 0 0`
  3. `swapon /dev/sdc1`
  4. Verify with `free -m`

**Lab**
1. Create a 1 GiB ext4 logical partition
2. Make sure this partition is mounted on /data, and will be mounted after reboot as well.
3. Create a 1 GiB swap partition and make sure that is mounted automatically as well.

## Logical Volume Manager (LVM)
* Like dynamic partitions. You can create, resize, and delete LVM partitions while
the system is running.
* Multiple disks (JBOD)
* Snapshots
* If you have more than one disk, the logical volume can extend over more than one disk.
* The physical volumes are combined into *logical volume groups*.
* sda, sdb > physical volumes > volume group (abstract of all storage) > logical volume: */dev/vgroup/logicalvol*
* A physical volume cannot span over more than one drive.
* `/boot/` partition cannot be on an LVM.
* To create:
  1. Create partitions of type Linux LVM.
  2. Create physical volumes using `pvcreate`.
  3. Create a volume group using the physical volumes using `vgcreate` and assign a name.
  4. Create logical volumes using `lvcreate` of a name and a size .
  5. Update the UUID in the `/etc/fstab`.

**Lab: creating an LVM**
<pre>
[root@web ~]# <b>pvcreate /dev/sdc</b>
  Physical volume "/dev/sdc" successfully created
[root@web ~]# <b>vgcreate vgwebserver /dev/sdc</b>
  Volume group "vgwebserver" successfully created
[root@web ~]# <b>vgdisplay  vgwebserver</b>
  --- Volume group ---
  VG Name               vgwebserver
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               4.00 GiB
  PE Size               4.00 MiB
  Total PE              1023
  Alloc PE / Size       1022 / 3.99 GiB
  Free  PE / Size       1 / 4.00 MiB
  VG UUID               iXsm0u-2IHQ-jgEI-JZag-0Bah-zc7D-yKRtig

[root@web ~]# <b>lvcreate -L 4.00G -n lvapache vgwebserver</b>
[root@web ~]# <b>mkfs.xfs /dev/vgwebserver/lvapache</b>
meta-data=/dev/vgwebserver/lvapache  isize=256    agcount=4, agsize=261632 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=0        finobt=0
data     =                       bsize=4096   blocks=1046528, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=0
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@web ~]# <b>mkdir /www</b>
[root@web ~]# <b>mount /dev/vgwebserver/lvapache /www</b>
</pre>

**Lab: Extending an LVM**
<pre>
[root@web ~]# <b>lsblk</b>
NAME               MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                  8:0    0   40G  0 disk
├─sda1               8:1    0  500M  0 part /boot
└─sda2               8:2    0 39.5G  0 part
  ├─centos-root    253:0    0 38.5G  0 lvm  /
  └─centos-swap    253:1    0    1G  0 lvm  [SWAP]
<b>sdb                  8:16   0    4G  0 disk
sdc                  8:32   0    4G  0 disk
└─vgwebserver-lvapache 253:2    0    4G  0 lvm  /www</b>
sr0                 11:0    1  4.2G  0 rom
[root@web ~]# <b>vgextend vgwebserver /dev/sdb</b>
  WARNING: Device for PV 6yaWFD-vyhH-ldwB-oE0e-qGva-oA8H-t3qTfB not found or rejected by a filter.
  Volume group "vgwebserver" successfully extended
[root@web ~]# <b>lvextend -r -L +4G /dev/vgwebserver/lvapache</b>
  Size of logical volume vgwebserver/lvapache changed from 3.99 GiB (1022 extents) to 7.99 GiB (2046 extents).
  Logical volume lvapache successfully resized
[root@web ~]# <b>lvs</b>
  LV     VG        Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root   centos    -wi-ao---- 38.46g
  swap   centos    -wi-ao----  1.00g
  <b>lvapache vgwebserver -wi-ao----  7.99g</b>
[root@web ~]# <b>df -hT</b>
Filesystem                   Type      Size  Used Avail Use% Mounted on
/dev/mapper/centos-root      xfs        39G  2.2G   37G   6% /
devtmpfs                     devtmpfs  220M     0  220M   0% /dev
tmpfs                        tmpfs     229M     0  229M   0% /dev/shm
tmpfs                        tmpfs     229M  4.3M  225M   2% /run
tmpfs                        tmpfs     229M     0  229M   0% /sys/fs/cgroup
/dev/sda1                    xfs       497M  120M  378M  25% /boot
<b>/dev/mapper/vgwebserver-lvapache xfs       4.0G   33M  4.0G   1% /www</b>
[root@web ~]# <b>xfs_growfs /dev/vgwebserver/lvapache</b>
meta-data=/dev/mapper/vgwebserver-lvapache isize=256    agcount=4, agsize=261632 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=0        finobt=0
data     =                       bsize=4096   blocks=1046528, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=0
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 1046528 to 2095104
[root@web ~]# <b>df -Tha /www</b>
Filesystem                   Type  Size  Used Avail Use% Mounted on
<b>/dev/mapper/vgwebserver-lvapache xfs   8.0G   33M  8.0G   1% /www</b></pre>
* We used `xfs_growfs` for an XFS filesystem. For EXT filesystems we could use `resize2fs` command.

**Lab: Reducing a volume**
First we have to `umount` the volume from the directory it is mounted to.
<pre>
[root@web ~]# <b>df -Tha /www</b>
Filesystem                   Type  Size  Used Avail Use% Mounted on
/dev/mapper/vgwebserver-lvapache xfs   8.0G   33M  8.0G   1% /www
[root@web ~]# <b>umount /dev/vgwebserver/lvapache</b>
[root@web ~]# <b>lvreduce -r /dev/vgwebserver/lvapache -L 7G</b></pre>
Then mount it back.
* Note that we cannot shrink xfs filesystems.
* verification commands:
  * `pvs`
  * `vgs`
  * `lvs`
  * `pvdisplay`
  * `vgdisplay`
  * `lvdisplay`

## Encrypted File Systems (EFS)

### Linux Unified Key Setup-on-disk-format (LUKS)
* Allows you to encrypt partitions on your Linux computer.
* You can encrypt your physical drive and removable drives with LUKS.
* Uses the existing device mapper kernel subsystem.
* Provides passphrase strengthening which protects your disk against
dictionary attacks.

**Lab**

Creating a partition:
<pre>
[vagrant@web ~]$ <b>lsblk</b>
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   40G  0 disk
├─sda1            8:1    0  500M  0 part /boot
└─sda2            8:2    0 39.5G  0 part
  ├─centos-root 253:0    0 38.5G  0 lvm  /
  └─centos-swap 253:1    0    1G  0 lvm  [SWAP]
sdb               8:16   0    4G  0 disk
[vagrant@web ~]$ fdisk /dev/sdb
fdisk: cannot open /dev/sdb: Permission denied
[vagrant@web ~]$ sudo -i
[root@web ~]# <b>fdisk /dev/sdb</b>
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0x84b8f34e.

Command (m for help): <b>n</b>
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1):
First sector (2048-8388607, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-8388607, default 8388607):
Using default value 8388607
Partition 1 of type Linux and of size 4 GiB is set

Command (m for help): <b>w</b>
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
</pre>
Now, let's work with LUKS
<pre>
[root@web ~]# <b>yum install cryptsetup</b>
[root@web ~]# <b>cryptsetup -vy luksFormat /dev/sdb1</b>

WARNING!
========
This will overwrite data on /dev/sdb1 irrevocably.

Are you sure? (Type uppercase yes): YES
Enter passphrase:
Verify passphrase:
<b>Command successful.</b>
[root@web ~]# <b>lsblk -f /dev/sdb1</b>
NAME FSTYPE      LABEL UUID                                 MOUNTPOINT
sdb1 <b>crypto_LUKS</b>       b88e988d-6720-4c4d-a3db-ddab9f19ecd8
[root@web ~]# <b>cryptsetup luksOpen /dev/sdb1 important_drive</b>
Enter passphrase for /dev/sdb1:
[root@web ~]# <b>mkfs.xfs /dev/mapper/important_drive</b>
meta-data=/dev/mapper/important_drive isize=256    agcount=4, agsize=261952 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=0        finobt=0
data     =                       bsize=4096   blocks=1047808, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=0
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@web ~]# <b>mount /dev/mapper/important_drive /mnt/important_drive/</b>
[root@web ~]# <b>cryptsetup status /dev/mapper/important_drive</b>
/dev/mapper/important_drive is active and is in use.
  type:    LUKS1
  cipher:  aes-xts-plain64
  keysize: 256 bits
  device:  /dev/sdb1
  offset:  4096 sectors
  size:    8382464 sectors
  mode:    read/write
[root@web ~]# <b>tail -n1 /etc/fstab</b>
/dev/mapper/imprtant_drive	/mnt/important_drive	xfs	defaults 0 0
[root@web ~]# <b>tail -n1 /etc/crypttab</b> # You have to create this file
important_drive /dev/mapper/important_drive none
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

### system crons
<pre>
[uwe@localhost ~]$ <b>sudo vi /etc/cron.d/list</b>
*/5 * * * 1-5 root ls /etc > /tmp/lsList # */5 means every 5 minutes
[uwe@localhost ~]$ <b>watch ls /tmp</b>
</pre>

### user crons
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

### anacron
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

### at
<pre>
[uwe@localhost ~]$ at 7:20 pm
at> cp /etc/hosts /tmp
at> ls $(PWD)
at> <EOT> # Ctrl + D
job 1 at Tue Dec 26 19:20:00 2017
</pre>

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
* Mamber of adm or wheel to read
  * `usermod -aG wheel username`
* `journalctl -n 50 -p err`: Displays last 50 entries with error priority
* By default only in memory: `/run/log/journal`
* To make it persistant:
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


## Configure Red Hat 7 for FTP server
<pre>
[root@web ~]# <b>yum install vsftpd</b>
[root@web pub]# <b>systemctl start vsftpd</b>
[root@web pub]# <b>systemctl enable vsftpd</b>
ln -s '/usr/lib/systemd/system/vsftpd.service' '/etc/systemd/system/multi-user.target.wants/vsftpd.service'
[root@web ~]# cd /var/ftp/pub/
[root@web pub]# <b>vi file1</b>
</pre>
On the client
<pre>
hossein@hossein ~/ftp_downloads $ <b>wget ftp://192.168.33.20/pub/file1</b>
--2018-01-05 19:36:10--  ftp://192.168.33.20/pub/file1
           => ‘file1’
Connecting to 192.168.33.20:21... connected.
Logging in as anonymous ... Logged in!
==> SYST ... done.    ==> PWD ... done.
==> TYPE I ... done.  ==> CWD (1) /pub ... done.
==> SIZE file1 ... 24
==> PASV ... done.    ==> RETR file1 ... done.
Length: 24 (unauthoritative)

file1                              100%[================================================================>]      24  --.-KB/s    in 0s

2018-01-05 19:36:10 (5.53 MB/s) - ‘file1’ saved [24]
</pre>

The configuration file:
<pre>
[root@web ~]# sed '/^\s*#/d;/^$/d' <b>/etc/vsftpd/vsftpd.conf</b>
anonymous_enable=YES
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES
pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES
</pre>

## KVM Virtualization Technology (VT)

We can run command below to see if the system supports VT. My system has 8 processors,
so there are eight separate sections. If disabled, you can enable it through BIOS or UEFI
* vmx - Intel VT-x
* svm - AMD SVM
<pre>
[hossein@localhost ~]$ <b>egrep 'vmx|svm' /proc/cpuinfo</b>
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc aperfmperf eagerfpu pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm epb tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid xsaveopt dtherm ida arat pln pts
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc aperfmperf eagerfpu pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm epb tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid xsaveopt dtherm ida arat pln pts
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc aperfmperf eagerfpu pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm epb tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid xsaveopt dtherm ida arat pln pts
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc aperfmperf eagerfpu pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm epb tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid xsaveopt dtherm ida arat pln pts
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc aperfmperf eagerfpu pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm epb tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid xsaveopt dtherm ida arat pln pts
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc aperfmperf eagerfpu pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm epb tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid xsaveopt dtherm ida arat pln pts
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc aperfmperf eagerfpu pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm epb tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid xsaveopt dtherm ida arat pln pts
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc aperfmperf eagerfpu pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm epb tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid xsaveopt dtherm ida arat pln pts
hossein@localhost ~]$ <b>lscpu | grep V</b>
Vendor ID:             GenuineIntel
Virtualization:        VT-x
</pre>
* We need the support of Linux kernel. We need 2 different modules:
  * kvm
  * kvm-intel or kvm-amd
    <pre>
    [hossein@localhost ~]$ <b>lsmod | grep kvm</b>
    kvm_intel             170086  0 
    kvm                   566340  1 kvm_intel
    irqbypass              13503  1 kvm
    </pre>
  * If Linux kernel doesn't support it, make sure you `yum groups install "Virtualization Host"`
* `libvirtd` is the interface that is used to manage virtualization. It is a daemon and it talks to management programs:
  * virt-manager: GUI
  * virtsh: short for Virtualization Shell
  * virt-install: dedicated to installing the VMs

**Installation:**
<pre>
[hossein@localhost ~]$ <b>sudo yum groups install "Virtualization Host"</b>
Complete!
[hossein@localhost ~]$ <b>rpm -qa | grep virt-manager</b>
[hossein@localhost ~]$ <b>sudo yum install virt-manager</b>
Complete!
[hossein@localhost ~]$ <b>systemctl status libvirtd</b>
● libvirtd.service - Virtualization daemon
   Loaded: loaded (/usr/lib/systemd/system/libvirtd.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2018-01-13 18:22:53 EST; 1min 46s ago
</pre>
To convert our images from Virtualbox (VDI) to KVM (qcow2) use the following command
<pre>
hossein@hossein ~ $ <b>qemu-img convert -O qcow2 '/home/hossein/VirtualBox VMs/CentOS7/CentOS7.vdi' server1.qcow2</b></pre>

<pre>
[hossein@localhost ~]$ <b>sudo virsh list --all</b>
 Id    Name                           State
----------------------------------------------------
 -     server1                        shut off
[hossein@localhost ~]$ <b>sudo virsh start server1</b>
Domain server1 started
[hossein@localhost ~]$ <b>sudo virsh list</b>
 Id    Name                           State
----------------------------------------------------
 3     server1                        running
</pre>

Virtual network has some backend configuration in `/etc/libvirt/qemu/networks/default.xml`
```bash
[hossein@localhost ~]$ sudo cat /etc/libvirt/qemu/networks/default.xml
<!--
WARNING: THIS IS AN AUTO-GENERATED FILE. CHANGES TO IT ARE LIKELY TO BE
OVERWRITTEN AND LOST. Changes to this xml configuration should be made using:
  virsh net-edit default
or other application using the libvirt API.
-->

<network>
  <name>default</name>
  <uuid>5fd92c46-f0a0-4d8f-9ca9-cd7acd476ffd</uuid>
  <forward mode='nat'/>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:83:c3:9f'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
    </dhcp>
  </ip>
</network>
[hossein@localhost ~]$ sudo virsh net-edit default
```
## Modular structure of kernel
* Kernel takes care of hardware management
* What happens if a new hardware is added?
  * udev (device manager for the kernel) takes care about it
* `lsmod` shows which modules are currently loaded
* `modinfo` show more information about a specific module. We can also get the parameters of a specific module
    <pre>
    [root@localhost ~]# <b>modinfo cdrom</b>
    filename:       /lib/modules/3.10.0-693.el7.x86_64/kernel/drivers/cdrom/cdrom.ko.xz
    license:        GPL
    rhelversion:    7.4
    srcversion:     BE3BD0D17D080229D55B173
    depends:
    intree:         Y
    vermagic:       3.10.0-693.el7.x86_64 SMP mod_unload modversions
    signer:         CentOS Linux kernel signing key
    sig_key:        DA:18:7D:CA:7D:BE:53:AB:05:BD:13:BD:0C:4E:21:F4:22:B6:A4:9C
    sig_hashalgo:   sha256
    parm:           debug:bool
    parm:           autoclose:bool
    parm:           autoeject:bool
    parm:           lockdoor:bool
    parm:           check_media_type:bool
    parm:           mrw_format_restart:bool
    </pre>
* `modprobe` can changes the kernel module behaviour
* To add custom arguments to the modules loaded by udev early in the boot process,
  you need to create a custom configuration file for modprobe, which udev uses to load the modules.
  <pre>
  [root@rhcsa ~]# vi /etc/modprobe.d/cdrom.conf
  options cdrom lockdoor=1
  [root@rhcsa ~]# modprobe cdrom
  </pre>
* One other important directory which you can find kernel tuneables is:
<pre>
[root@rhcsa ~]# <b>ls /proc/sys/</b>
abi  crypto  debug  dev  fs  kernel  net  user  vm
</pre>
Here are some examples:
<pre>
[root@rhcsa ~]# <b>cat /proc/sys/net/ipv4/icmp_echo_ignore_all</b>
0
[root@rhcsa ~]# <b>cat /proc/sys/net/ipv4/ip_forward</b>
0 # Not capable for routing
[root@rhcsa ~]# <b>echo 1 > /proc/sys/net/ipv4/ip_forward</b>
[root@rhcsa ~]# !cat
<b>cat /proc/sys/net/ipv4/ip_forward</b>
1
[root@rhcsa ~]# <b>cat /proc/sys/vm/swappiness</b> 
30
</pre>

  * allows the system administrator to immediately enable and disable kernel features
    * is not persistent
* To configure tunables persistently, use `sysctl`
  * Fo example if you changed the swappiness in `/proc/sys/vm/swappiness`, 
  you should use `sysctl vm.swappiness` command
  * The master configuration file is: `/etc/sysctl.conf`
  * In the current RHEL we have `/etc/sysctl.d/` wherein we can put our configuration.
    * The filenames starts with lower numbers has priority over than higher numbers.
<pre>
[root@rhcsa ~]# <b>sysctl -a | wc -l</b>
801
[root@rhcsa ~]# <b>sysctl vm.swappiness</b>
vm.swappiness = 30
[root@rhcsa ~]# <b>vi /etc/sysctl.d/50-swap.conf</b>
vm.swappiness=20
[root@rhcsa ~]# <b>reboot</b>
[root@rhcsa ~]# <b>sysctl vm.swappiness</b>
vm.swappiness = 20
</pre>
How to update the kernel (It is not really updating the kernel but installing a new kernel beside the old one):
<pre>
[root@rhcsa ~]# uname -rv
3.10.0-693.el7.x86_64 #1 SMP Tue Aug 22 21:09:27 UTC 2017
[root@rhcsa ~]# yum update kernel
</pre>
Note that after the update you will have both old and new kernel which you can
choose from in boot.

**Lab**
1. Show a list of all kernel modules (`lsmod`)
2. Request available parameters for the `iwlwifi` module (`modinfo`)
3. Make sure the `iwlwifi` module sets the `led_mode` to 1
4. Manually load the `iwlwifi` module and verify that the load option has been used (`modprobe.d`)

## Boot procedure
* `/etc/default/grub` is the most important configuration file.
* Once the configuration is done we have to push the changes to the boot loader:
  * `grub2-mkconfig -o /boot/grub2/grub.cfg`
* You can access boot menu while the system is booting by pressing **e** to edit
  * The most important line is the line that starts with `linux16`
  * From there you can start troubleshooting mode by `systemd.unit=rescue.target` or `systemd.unit=emergency.target`

<pre>
[root@rhcsa ~]# <b>cat /etc/default/grub</b>
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap <s>rhgb quiet</s>"
GRUB_DISABLE_RECOVERY="true"
[root@rhcsa ~]# <b>grub2-mkconfig -o /boot/grub2/grub.cfg</b> 
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-693.11.6.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-693.11.6.el7.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-693.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-693.el7.x86_64.img
Found linux image: /boot/vmlinuz-0-rescue-4440c6c27bc44a0d88ec3034e7d50528
Found initrd image: /boot/initramfs-0-rescue-4440c6c27bc44a0d88ec3034e7d50528.img
done
</pre>
Modifying *systemd* environment:
systemd configurations are in  `/usr/lib/systemd/system/`. You shouldn't edit this file,
because if an update comes to the package, you will lose your configurations.
You can copy that file to `/etc/systemd/system/` and then edit that file. For example:
<pre>
[root@rhcsa ~]# <b>cp /usr/lib/systemd/system/sshd.service /etc/systemd/system/</b>
[root@rhcsa ~]# <b>vi /etc/systemd/system/sshd.service</b></pre>
To see the different systemd units
<pre>
[root@rhcsa ~]# <b>systemctl -t help</b>
Available unit types:
service
socket
busname
target
snapshot
device
mount
automount
swap
timer
path
slice
scope
[root@rhcsa ~]# <b>systemctl --type=service</b> # Show the current services and their status
</pre>

### Systemd targets
* Just a group
* Final state of system (`AllowIsolate=yes`)
  * emergency
  * rescue
  * multi-user
  * graphical

Switching between isolatable targets
<pre>
[root@rhcsa ~]# <b>systemctl get-default</b>
multi-user.target
[root@rhcsa ~]# <b>systemctl set-default graphical.target</b>
Removed symlink /etc/systemd/system/default.target.
Created symlink from /etc/systemd/system/default.target to /usr/lib/systemd/system/graphical.target.
</pre>
To switch between targets in runtime
<pre>
[root@rhcsa ~]# <b>systemctl isolate graphical.target</b>
</pre>

### Mounting by systemctl instead of /etc/fstab
<pre>
[root@rhcsa ~]# <b>lsblk /dev/sda</b>
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
sda      8:0    0   5G  0 disk 
└─sda1   8:1    0   5G  0 part 
[root@rhcsa ~]# <b>mkdir /data</b>
[root@rhcsa ~]# <b>cp /usr/lib/systemd/system/tmp.mount /etc/systemd/system/data.mount</b>
[root@rhcsa ~]# <b>cat /etc/systemd/system/data.mount</b> 
[Unit]
DefaultDependencies=no
Conflicts=umount.target
Before=local-fs.target umount.target
After=swap.target

[Mount]
What=/dev/sda1
Where=/data
Type=xfs
Options=defaults

[Install]
WantedBy=local-fs.target
[root@rhcsa ~]# <b>systemctl daemon-reload</b>
[root@rhcsa ~]# <b>systemctl status data.mount</b>
● data.mount - /data
   Loaded: loaded (/etc/systemd/system/data.mount; disabled; vendor preset: disabled)
   Active: inactive (dead)
    Where: /data
     What: /dev/sda1
[root@rhcsa ~]# <b>systemctl enable data.mount</b>
Created symlink from /etc/systemd/system/local-fs.target.wants/data.mount to /etc/systemd/system/data.mount.
[root@rhcsa ~]# <b>systemctl start data.mount</b>
[root@rhcsa ~]# <b>lsblk /dev/sda</b>
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
sda      8:0    0   5G  0 disk 
└─sda1   8:1    0   5G  0 part <b>/data</b>
</pre>

### Password recovery
is more complicated than previous version of RHELs.
1. press `e` while the system is booting 
2. at the end of line `linux16` add `rd.break`
3.
<pre>
switch_root:/# <b>mount -o remount,rw /sysroot</b>
switch_root:/# <b>chroot /sysroot/</b>
sh-4.2# <b>passwd</b>
sh-4.2# <b>touch .autorelabel</b> # To satisfy SELinux
sh-4.2# <b>exit</b>
switch_root:/# <b>reboot</b>
</pre>

## Configuring Apache webserver
* Apache webserver is highly modular:
  * Modules are in `/etc/httpd/modules/`
* Configuration file: `/etc/httpd/conf/httpd.conf`
* Document root by default: `/var/www/html/`
* No need to know Apache webserver for RHCSA exam by itself but useful when we want to work with SELinux
* More information in: https://github.com/hosseinoliabak/linux_notes/blob/master/04_web_services.md


## Security-Enhanced Linux (SELinux)
* enabled:
  * Enforcing mode: to toggle between this mode and permissive mode **setenforce** is needed
  * Permissive mode: good for troubleshooting, logs but doesn't block anything
* disables: to toggle between enabled and disables, a reboot is needed

<pre>
[root@rhcsa ~]# <b>getenforce</b> 
Enforcing
[root@rhcsa ~]# <b>setenforce Permissive</b>
[root@rhcsa ~]# <b>getenforce</b> 
Permissive
[root@rhcsa ~]# <b>sed '/^\s*#/d;/^$/d' /etc/sysconfig/selinux</b> 
SELINUX=enforcing # To toggle beteen enabled and disabled we edit this line *disabled* and then reboot the system
SELINUXTYPE=targeted
</pre>

Enforcing:
* Source: such as processes, users
  * Context labels (`ls -Z`, `ps Z`, `ss -Z`): used as an identifier. written down in the SELinux policy
    * `*-u`: user, for advanced SELinux users
    * `*-r`: role, for advanced SELinux users
    * `*-t`: type
* Target: such as files, ports
<pre>
[root@rhcsa ~]# <b>ps auxZ | grep httpd</b>
system_u:system_r:<b>httpd_t</b>:s0    root      2223  0.0  0.3 226240  5140 ?        Ss   20:22   0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0    root      2224  0.0  0.2 240588  3080 ?        S    20:23   0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0    root      2225  0.0  0.2 240588  3080 ?        S    20:23   0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0    root      2226  0.0  0.2 240588  3080 ?        S    20:23   0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0    apache    2227  0.0  0.2 240588  3304 ?        S    20:23   0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0    root      2228  0.0  0.2 240588  3080 ?        S    20:23   0:00 /usr/sbin/httpd -DFOREGROUND
[root@rhcsa ~]# <b>ls -Z /var/www/</b>
drwxr-xr-x. root root system_u:object_r:<b>httpd_sys_script_exec_t</b>:s0 cgi-bin
drwxr-xr-x. root root system_u:object_r:<b>httpd_sys_content_t</b>:s0 html
</pre>
Check services don't conflict with booleans: `getsebool`
<pre>
[root@rhcsa ~]# <b>getsebool -a | grep ftp</b>
ftpd_anon_write --> off
ftpd_connect_all_unreserved --> off
ftpd_connect_db --> off
ftpd_full_access --> off
ftpd_use_cifs --> off
ftpd_use_fusefs --> off
ftpd_use_nfs --> off
ftpd_use_passive_mode --> off
httpd_can_connect_ftp --> off
httpd_enable_ftp_server --> off
tftp_anon_write --> off
tftp_home_dir --> off
[root@rhcsa ~]# <b>setsebool tftp_home_dir on</b> # first on
[root@rhcsa ~]# <b>setsebool -P tftp_home_dir on</b> # -P to make it persistent. second on
[root@rhcsa ~]# <b>getsebool -a | grep tftp</b>
tftp_anon_write --> off
tftp_home_dir --> <b>on</b>
[root@rhcsa ~]# <b>semanage boolean -l | grep tftp</b>
tftp_anon_write                (off  ,  off)  Allow tftp to anon write
tftp_home_dir                  (on   ,   <b>on</b>)  Allow tftp to home dir
</pre>
When we move a file, it keeps its original context label, if we want to change it according
to its parent directory
<pre>
[root@rhcsa ~]# <b>restorecon -v /etc/hosts</b></pre>
What if you want to set a context label for yourself?
<pre>
[root@rhcsa ~]# <b>ls -Zd /var/www</b>
drwxr-xr-x. root root system_u:object_r:<b>httpd_sys_content_t</b>:s0 /var/www
[root@rhcsa ~]# <b>ls -Zd /www</b>
drwxr-xr-x. root root unconfined_u:object_r:<b>default_t</b>:s0 /www
[root@rhcsa ~]# <b>semanage fcontext -a -t httpd_sys_content_t "/www(/.*)?"</b> # writes to the policy
[root@rhcsa ~]# <b>ls -Zd /www</b>
drwxr-xr-x. root root unconfined_u:object_r:default_t:s0 /www
[root@rhcsa ~]# <b>restorecon -Rv /www</b> # writes to inodes
restorecon reset /www context unconfined_u:object_r:default_t:s0->unconfined_u:object_r:httpd_sys_content_t:s0
[root@rhcsa ~]# <b>!ls</b>
ls -Zd /www
drwxr-xr-x. root root unconfined_u:object_r:<b>httpd_sys_content_t</b>:s0 /www
</pre>
Never use `chcon` command because it only writes to the inodes.

<pre>
[root@rhcsa ~]# <b>grep AVC /var/log/audit/audit.log</b> 
type=USER_AVC msg=audit(1515971856.024:10): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='avc:  received setenforce notice (enforcing=0)  exe="/usr/lib/systemd/systemd" sauid=0 hostname=? addr=? terminal=?'
type=USER_AVC msg=audit(1515975172.295:175): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='avc:  received policyload notice (seqno=2)  exe="/usr/lib/systemd/systemd" sauid=0 hostname=? addr=? terminal=?'
type=USER_AVC msg=audit(1515975172.295:176): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='avc:  received policyload notice (seqno=3)  exe="/usr/lib/systemd/systemd" sauid=0 hostname=? addr=? terminal=?'
type=USER_AVC msg=audit(1515979362.209:196): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='avc:  received setenforce notice (enforcing=0)  exe="/usr/lib/systemd/systemd" sauid=0 hostname=? addr=? terminal=?'
type=USER_AVC msg=audit(1515981233.711:202): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='avc:  received policyload notice (seqno=4)  exe="/usr/lib/systemd/systemd" sauid=0 hostname=? addr=? terminal=?'
type=USER_AVC msg=audit(1515981661.377:180): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='avc:  received policyload notice (seqno=2)  exe="/usr/lib/systemd/systemd" sauid=0 hostname=? addr=? terminal=?'
type=USER_AVC msg=audit(1515981661.377:181): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='avc:  received policyload notice (seqno=3)  exe="/usr/lib/systemd/systemd" sauid=0 hostname=? addr=? terminal=?'
type=USER_AVC msg=audit(1515981661.377:182): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='avc:  received policyload notice (seqno=4)  exe="/usr/lib/systemd/systemd" sauid=0 hostname=? addr=? terminal=?'
[root@rhcsa ~]# <b>cat /var/log/messages</b>
</pre>

#### .autorelabel
to relabel:
If this file exists, SELinux performs a complete file system relabel (using the `/sbin/fixfiles -f -F relabel` command), and then deletes `/.autorelabel`.
<pre>
[root@rhcsa ~]# pwd
/root
[root@rhcsa ~]# touch .autorelabel
[root@rhcsa ~]# reboot
</pre>

#### troubleshooting
To figure out if the problem is SELinux and nothing else, set it to 
Permissive mode. Then try to see if the problem is resolved.

**Lab**
1. Configure the Apache web server to listen on port 888.
2. Setup SELinux to allow Apache to bind to this port.

## Firewall Configuration with firewalld
<pre>
[root@web ~]# <b>systemctl status firewalld</b>
firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled)
   Active: active (running) since Fri 2018-01-05 01:42:33 UTC; 10s ago
 Main PID: 3820 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─3820 /usr/bin/python -Es /usr/sbin/firewalld --nofork --nopid

Jan 05 01:42:33 web systemd[1]: Started firewalld - dynamic firewall daemon.
[root@web ~]# <b>firewall-cmd --state</b>
running
[root@web ~]# <b>firewall-config</b> # If you have graphical interface
</pre>
`Firewalld` short for firewall daemon, is a complete firewall solution
available by default on RHEL7. There are 9 firewall zones:
* `drop`: lowest level of trust. All incoming connections are dropped without
reply and only outgoing connections are possible.
* `block`: similar to drop, but instead of simply dropping connections,
incoming requests are rejected with an `icmp-host-prohibited` or `icmp6-adm-prohibited` message
* `public`: Represents public, untrusted networks. You don't trust other computers but may
allow selected incoming connections on a case-by-case basis.
* `external`: means that you are using the firewall as your gateway. It is
configured for NAT masquerading so that your internal network remains private but reachable.
* `internal`: The other side of external zone. Reject incoming traffic unless related to outgoing traffic
or matching the ssh, mdns, ipp-client, samba-client, or dhcpv6-client pre-defined services
* `home`: Same as internal.
* `dmz`: USed for computers located in a DMZ. Only certain incoming connections are allowed.
* `work`: Used for work machine. Trusts most of the computers in the network.
* `trusted`: trusts all of the machines in the network.

<pre>
[root@web ~]# <b>firewall-cmd --get-default-zone</b>
public
[root@web ~]# <b>firewall-cmd --get-active-zone</b>
public
  interfaces: enp0s3
[root@web ~]# <b>firewall-cmd --zone=public --list-all</b>
public (default, active)
  interfaces: enp0s3
  sources:
  services: dhcpv6-client ssh
  ports:
  masquerade: no
  forward-ports:
  icmp-blocks:
  rich rules:
[root@web ~]# <b>firewall-cmd --list-all-zones</b></pre>

#### Changing zones in runtime not permanently:
<pre>
[root@web ~]# <b>firewall-cmd --zone=home --change-interface=enp0s9</b>
success
[root@web ~]# <b>firewall-cmd --get-active-zones</b>
<b>home
  interfaces: enp0s9</b>
public
  interfaces: enp0s3
[root@web ~]# <b>systemctl restart firewalld</b>
[root@web ~]# <b>firewall-cmd --get-active-zones</b>
public
  interfaces: enp0s3
</pre>

#### Making Permanent Zone Changes

<pre>
[vagrant@web ~]$ <b>vi /etc/sysconfig/network-scripts/ifcfg-enp0s9</b>
[root@web ~]# <b>reboot</b>
[vagrant@web ~]$ <b>firewall-cmd --get-active-zones</b>
<b>home
  interfaces: enp0s9</b>
public
  interfaces: enp0s3 enp0s8
</pre>
Or:
1. Modify the zone for the connection in network manager
  * `nmcli conn modify <iface> connection.zone <zone>`
2. Modify zone for the connection in firewalld
  * `firewall-cmd --zone=home --change-interface=eth0 --permanent`
3. Optional - restart the network and firewalld service
  * `systemctl restart network.service`

#### Setting the Default Zone
If all of your interfaces can best be handled by a single zone, it's
probably easier to just select the best default zone and then use that for
your configuration.
<pre>
[root@web ~]# <b>firewall-cmd --set-default-zone=home</b>
success
[root@web ~]# <b>firewall-cmd --get-default-zone</b>
home
[root@web ~]# <b>firewall-cmd --get-active-zones</b>
home
  interfaces: enp0s3 enp0s8 enp0s9
</pre>

#### Add Custom zones:
<pre>
[root@web ~]# <b>firewall-cmd --permanent --new-zone=myPublicWeb</b>
success
[root@web ~]# <b>firewall-cmd --permanent --new-zone=myPrivateDNS</b>
success
[root@web ~]# <b>firewall-cmd --reload</b>
success
[root@web ~]# <b>firewall-cmd --get-zones</b>
block dmz drop external home internal <b>myPrivateDNS myPublicWeb</b> public trusted work
</pre>

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

#### Working with services
To list all services:
<pre>
[root@rhcsa ~]# <b>firewall-cmd --get-services</b>
RH-Satellite-6 amanda-client amanda-k5-client bacula bacula-client bitcoin bitcoin-rpc bitcoin-testnet bitcoin-testnet-rpc ceph ceph-mon cfengine condor-collector ctdb dhcp dhcpv6 dhcpv6-client dns docker-registry dropbox-lansync elasticsearch freeipa-ldap freeipa-ldaps freeipa-replication freeipa-trust ftp ganglia-client ganglia-master high-availability http https imap imaps ipp ipp-client ipsec iscsi-target kadmin kerberos kibana klogin kpasswd kshell ldap ldaps libvirt libvirt-tls managesieve mdns mosh mountd ms-wbt mssql mysql nfs nrpe ntp openvpn ovirt-imageio ovirt-storageconsole ovirt-vmconsole pmcd pmproxy pmwebapi pmwebapis pop3 pop3s postgresql privoxy proxy-dhcp ptp pulseaudio puppetmaster quassel radius rpc-bind rsh rsyncd samba samba-client sane sip sips smtp smtp-submission smtps snmp snmptrap spideroak-lansync squid ssh synergy syslog syslog-tls telnet tftp tftp-client tinc tor-socks transmission-client vdsm vnc-server wbem-https xmpp-bosh xmpp-client xmpp-local xmpp-server
</pre>
To list the current services
<pre>
[root@rhcsa ~]# <b>firewall-cmd --list-services</b>
ssh dhcpv6-client
</pre>
firewalld services are in xml format. You shouldn't edit this file, instead put your file in
`/etc/firewalld/services/`
<pre>
[root@rhcsa ~]# <b>ls /usr/lib/firewalld/services/</b>
amanda-client.xml        freeipa-replication.xml  libvirt-tls.xml           postgresql.xml       spideroak-lansync.xml
amanda-k5-client.xml     freeipa-trust.xml        libvirt.xml               privoxy.xml          squid.xml
bacula-client.xml        ftp.xml                  managesieve.xml           proxy-dhcp.xml       ssh.xml
bacula.xml               ganglia-client.xml       mdns.xml                  ptp.xml              synergy.xml
bitcoin-rpc.xml          ganglia-master.xml       mosh.xml                  pulseaudio.xml       syslog-tls.xml
bitcoin-testnet-rpc.xml  high-availability.xml    mountd.xml                puppetmaster.xml     syslog.xml
bitcoin-testnet.xml      https.xml                mssql.xml                 quassel.xml          telnet.xml
bitcoin.xml              http.xml                 ms-wbt.xml                radius.xml           tftp-client.xml
ceph-mon.xml             imaps.xml                mysql.xml                 RH-Satellite-6.xml   tftp.xml
ceph.xml                 imap.xml                 nfs.xml                   rpc-bind.xml         tinc.xml
cfengine.xml             ipp-client.xml           nrpe.xml                  rsh.xml              tor-socks.xml
condor-collector.xml     ipp.xml                  ntp.xml                   rsyncd.xml           transmission-client.xml
ctdb.xml                 ipsec.xml                openvpn.xml               samba-client.xml     vdsm.xml
dhcpv6-client.xml        iscsi-target.xml         ovirt-imageio.xml         samba.xml            vnc-server.xml
dhcpv6.xml               kadmin.xml               ovirt-storageconsole.xml  sane.xml             wbem-https.xml
dhcp.xml                 kerberos.xml             ovirt-vmconsole.xml       sips.xml             xmpp-bosh.xml
dns.xml                  kibana.xml               pmcd.xml                  sip.xml              xmpp-client.xml
docker-registry.xml      klogin.xml               pmproxy.xml               smtp-submission.xml  xmpp-local.xml
dropbox-lansync.xml      kpasswd.xml              pmwebapis.xml             smtps.xml            xmpp-server.xml
elasticsearch.xml        kshell.xml               pmwebapi.xml              smtp.xml
freeipa-ldaps.xml        ldaps.xml                pop3s.xml                 snmptrap.xml
freeipa-ldap.xml         ldap.xml                 pop3.xml                  snmp.xml
</pre>
To make a custom service
```bash
[root@rhcsa ~]# vi /etc/firewalld/services/myservice.xml 
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>myservice</short>
  <description>This is a test service.</description>
  <port protocol="tcp" port="2242"/>
</service>
```
<pre>
[root@rhcsa ~]# <b>firewall-cmd --reload</b>
success
[root@rhcsa ~]# <b>firewall-cmd --get-services | grep myservice > /dev/null && echo myservice is there</b>
myservice is there
[root@rhcsa ~]# <b>firewall-cmd --add-service myservice --permanent</b>
success
[root@rhcsa ~]# <b>firewall-cmd --reload</b>
success
[root@rhcsa ~]# <b>firewall-cmd --list-services</b>
ssh dhcpv6-client myservice
[root@rhcsa ~]# <b>firewall-cmd --list-all</b>
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0
  sources: 
  services: ssh dhcpv6-client <b>myservice</b>
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
</pre>

**Lab**:
Configure a firewall using firewalld. Make sure this firewall meets following
requirements and that the settings are set in a persistent way:
1. Allow traffic to SSH, NTP, FTP, VNC, Apache, and DNS
2. Create a custom service file that allows access to port TCP 23456 using the name
of your choice.

## Using Kickstart
* To automate installation of multiple servers
* `anaconda-ks.cfg` is default kickstart file. If you use this file, you will be having 
the exact server you are now having.

Creating a custom Kickstart file:
<pre>
[root@rhcsa ~]# <b>yum search kickstart</b>
======================================================= N/S matched: kickstart ========================================================
pykickstart.noarch : A python library for manipulating kickstart files
<b>system-config-kickstart</b>.noarch : A graphical interface for making kickstart files

  Name and summary matches only, use "search all" for everything.
[root@rhcsa ~]# <b>yum install system-config-kickstart</b>
[root@rhcsa ~]# <b>system-config-kickstart</b></pre>
Using the file for automatic installation:
* When booting system and the installation option is appeared, press tab and then
at the end of line `vmlimuz` type: `ks=ftp://myserver.com/kickstart.xml`.
* pxe boot must be supported

**Lab**
Modify the anaconda-ks.cfg file to meet the following requirements
1. The installer prompts for a password.
 Hint: Everything that is not at kickstart file you will be prompted for
2. Network connectivity will be enabled on boot
3. The local machine name is set to `server2.example.com`

## LDAP, IPA, Microsoft AD
### Connecting the client to an LDAP server
<pre>
[root@client1 ~]# <b>yum groups install "Directory Client"</b>
[root@client1 ~]# <b>systemctl status sssd</b>
● sssd.service - System Security Services Daemon
   Loaded: loaded (/usr/lib/systemd/system/sssd.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/sssd.service.d
           └─journal.conf
   Active: <b>active (running)</b> since Sun 2018-01-14 19:12:51 EST; 56s ago
</pre>
* Configuration file is in: `/etc/sssd/sssd.conf`
* We can use graphical utility as well:
<pre>   
[root@client1 ~]# <b>yum install authconfig-gtk</b> 
[root@client1 ~]# <b>authconfig-gtk &</b></pre>

![ldap1](https://user-images.githubusercontent.com/31813625/35826132-d5527336-0a85-11e8-8d07-51c9d6dfe08b.png)

![ldap2](https://user-images.githubusercontent.com/31813625/35826133-d562c574-0a85-11e8-9064-f2e0b4765184.png)

<pre>
[root@client1 ~]# <b>getent passwd ldapuser1</b>
ldapuser1:*:1308400001:1308400001:ldapuser1 ldapuser1:/home/ldap/ldapuser1:/bin/sh
[root@client1 ~]# <b>su - ldapuser1</b>
Last login: Mon Feb  5 15:03:10 EST 2018 on pts/1
su: warning: cannot change directory to /home/ldap/ldapuser1: No such file or directory
-sh-4.2$ 
</pre>
As we can see there is no home directory for the `ldapuser1`. Let's make a home
directory in `/home/ldap/ldapuser1` for this user.
<pre>
[root@client1 ~]# <b>systemctl enable autofs</b>
Created symlink from /etc/systemd/system/multi-user.target.wants/autofs.service to /usr/lib/systemd/system/autofs.service.
[root@client1 ~]# <b>echo "/home/ldap /etc/anyname.home" > /etc/auto.master.d/anyname.autofs</b>
[root@client1 ~]# <b>cat /etc/auto.master.d/anyname.autofs</b> 
/home/ldap /etc/anyname.home
[root@client1 ~]# <b>echo "ldapuser1 -rw,sync serveripa.example.com:home/ldap/ldapuser1" > /etc/anyname.home</b>
[root@client1 ~]# <b>cat /etc/anyname.home</b>
ldapuser1 -rw,sync serveripa.example.com:/home/ldap/ldapuser1
[root@client1 ~]# <b>systemctl start autofs</b>
[root@client1 ~]# <b>su - ldapuser1</b>
Last login: Mon Feb  5 15:21:00 EST 2018 on pts/0
-sh-4.2$ <b>pwd</b>
/home/ldap/ldapuser1
</pre>
Above we configured home directory for only one user `ldapuser1`. What if we want to 
configure for any user? I changed our `/etc/anyname.home` file.
<pre>
[root@client1 ~]# <b>cat /etc/anyname.home</b>
#ldapuser1 -rw,sync serveripa.example.com:/home/ldap/ldapuser1
* -rw,sync serveripa.example.com:/home/ldap/&
[root@client1 ~]# <b>!sys</b>
systemctl start autofs
[root@client1 ~]# <b>su - ldapuser2</b>
-sh-4.2$ <b>pwd</b>
/home/ldap/ldapuser2
</pre>

### Connecting to an IPA server
Before working, be sure that your LDAP server is your DNS server. Check `/etc/resolv.conf` 
<pre>
[root@client1 ~]# <b>yum install ipa-client -y</b>
[root@client1 ~]# <b>ipa-client-install</b>
</pre>

### Connecting to a Microsoft AD server
I use Windows Server 2012 as a DC.
Installed Identity Management for UNIX Components:
<pre>
Dism.exe /online /enable-feature /featurename:adminui /all # to install the administration tools for Identity Management for UNIX.
Dism.exe /online /enable-feature /featurename:nis /all   # to install Server for NIS.
Dism.exe /online /enable-feature /featurename:psync /all # to install Password Synchronization.
</pre>
![image](https://user-images.githubusercontent.com/31813625/35012477-dcdd8220-fad7-11e7-92da-66a3d872b77f.png)
![image](https://user-images.githubusercontent.com/31813625/35012580-3a340e8a-fad8-11e7-891e-6be1baa86f0e.png)

Let's check if our DNS server which is on DC works:
<pre>
[root@client1 ~]# <b>grep "DNS1\|IPADDR" /etc/sysconfig/network-scripts/ifcfg-eth0</b>
IPADDR=192.168.122.10
DNS1=192.168.122.8
[root@client1 ~]# <b>ping mtl-dc1.domain.local</b>
PING mtl-dc1.domain.local (192.168.122.8) 56(84) bytes of data.
64 bytes from 192.168.122.8 (192.168.122.8): icmp_seq=1 ttl=128 time=0.191 ms
64 bytes from 192.168.122.8 (192.168.122.8): icmp_seq=2 ttl=128 time=0.399 ms
64 bytes from 192.168.122.8 (192.168.122.8): icmp_seq=3 ttl=128 time=0.398 ms
64 bytes from 192.168.122.8 (192.168.122.8): icmp_seq=4 ttl=128 time=0.469 ms
</pre>
Installation
<pre>
[root@client1 ~]# <b>yum install realmd</b>
[root@client1 ~]# <b>realm discover mtl-dc1.domain.local</b> # Discovers settings of the domain
domain.local
  type: kerberos
  realm-name: DOMAIN.LOCAL
  domain-name: domain.local
  configured: no
  server-software: active-directory
  client-software: sssd
  required-package: oddjob
  required-package: oddjob-mkhomedir
  required-package: sssd
  required-package: adcli
  required-package: samba-common-tools
[root@client1 ~]# <b>realm join mtl-dc1.domain.local</b> 
Password for Administrator: 
[root@client1 ~]# <b>realm permit --realm domain.local --all</b> # Enables logins on AD
</pre>

## NFS (Network File System)
NFS server: The NFS server is the traditional way of sharing or exporting file systems
in UNIX and Linux. Often, home directories in Linux are centralized on a single server
with other systems mounting the remote file systems.
<pre>
[root@client1 ~]# <b>yum install nfs-utils</b>
[root@client1 ~]# <b>showmount -e serveripa.example.com</b>
Export list for serveripa.example.com:
/srv/nfs   *
/home/ldap *
[root@client1 ~]# <b>mkdir /mnt/{nfs-1,nfs-2}</b>
[root@client1 ~]# <b>mount -t nfs serveripa.example.com:/srv/nfs /mnt/nfs-1</b>
[root@client1 ~]# <b>echo "serveripa.example.com:/srv/nfs /mnt/nfs-2 nfs _netdev 0 0" >> /etc/fstab</b> 
[root@client1 ~]# <b>tail -n1 /etc/fstab</b> 
serveripa.example.com:/srv/nfs /mnt/nfs-2 nfs _netdev 0 0
[root@client1 ~]# <b>df -h | grep nfs</b>
serveripa.example.com:/srv/nfs   17G  7.4G  9.7G  44% /mnt/nfs-2
[root@client1 ~]# <b>cd /mnt/nfs-2</b>
[root@client1 nfs-2]# <b>ls</b></pre>
## Samba client
It is better than NFS. (nowadays NFS is combined with Kerberos to provide the security)
<pre>
[root@client1 ~]# <b>yum install samba-client cifs-utils</b>
[root@client1 ~]# <b>smbclient -L serveripa.example.com</b>
Enter SAMBA\root's password: 
Anonymous login successful
OS=[Windows 6.1] Server=[Samba 4.6.2]

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	<b>data</b>            Disk      data share
	public          Disk      Public Directory
	IPC$            IPC       IPC Service (Samba 4.6.2)
Anonymous login successful
OS=[Windows 6.1] Server=[Samba 4.6.2]

	Server               Comment
	---------            -------

	Workgroup            Master
	---------            -------
	SAMBA                SERVERIPA
[root@client1 ~]# <b>mount -t cifs -o username=sambauser1,password=password //serveripa.example.com/data /mnt/manual</b>
[root@client1 ~]# <b>ls /mnt/manual/</b>
samba-user-1  samba-user-2  samba-user-3
# To mount permanently we can use:
[root@client1 ~]# <b>mkdir /smbdata</b>
[root@client1 ~]# <b>vi /root/cred</b>
username=sambauser1
password=password
[root@client1 ~]# <b>echo "//serveripa.example.com/data /smbdata cifs _netdev,credentials=/root/cred 0 0" >> /etc/fstab</b>
[root@client1 ~]# <b>mount -a</b>
[root@client1 ~]# <b>df -hT | grep serveripa</b>
//serveripa.example.com/data cifs       17G  7.4G  9.7G  44% /smbdata
[root@client1 ~]# <b>cd /smbdata/</b>
[root@client1 smbdata]# <b>ls</b>
samba-user-1  samba-user-2  samba-user-3
</pre>