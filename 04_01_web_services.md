# Apache 2.4 web server
* Available free of charge under the Apache open-source license.
* Ported to Linux, UNIX, FreeBSD, Solaris, Windows
* The Apache foundation hosts many additional projects
  * Cocoon (A web development framework)
  * Tomcat (Implementation of Java Servlet and JSP APIs)
  * Hadoop (A distributed computing platform)
  * Lucene (A text-based search engine)
  * Many others ...

What we will be covering:
* Install Apache onto a standard Linux installation
* Serve a single site
* Serve multiple sites with virtual hosts
* Perform user authentication and access control
* Set up a secure site (https)
* Serve multiple sites with virtual hosts
* Control logging and status reporting

### Installation
<pre>
[root@web ~]# <b>yum install httpd</b>
[root@web ~]# <b>systemctl enable httpd</b>
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
[root@web ~]# <b>systemctl start httpd</b>
[root@web ~]# <b>systemctl list-unit-files | grep httpd</B>
httpd.service                               enabled
[root@web ~]# <b>yum install httpd-manual</b>
[root@web ~]# <b>systemctl restart httpd</b>
[root@web ~]# <b>curl -IL http://localhost/manual/</b>
HTTP/1.1 <b>200 OK</b>
Date: Fri, 29 Dec 2017 23:01:27 GMT
Server: Apache/2.4.6 (CentOS)
Last-Modified: Thu, 19 Oct 2017 20:39:58 GMT
ETag: "22b1-55bec5c082380"
Accept-Ranges: bytes
Content-Length: 8881
Content-Type: text/html; charset=UTF-8
</pre>

#### Alternatively using `yum groups install`
<pre
[root@client1 ~]# <b>yum groups list</b>
Available Environment Groups:
   Minimal Install
   Compute Node
   Infrastructure Server
   File and Print Server
   <b>Basic Web Server</b>
   Virtualization Host
   Server with GUI
   GNOME Desktop
   KDE Plasma Workspaces
   Development and Creative Workstation
Available Groups:
   Compatibility Libraries
   Console Internet Tools
   Development Tools
   Graphical Administration Tools
   Legacy UNIX Compatibility
   Scientific Support
   Security Tools
   Smart Card Support
   System Administration Tools
   System Management
Done
[root@client1 ~]# <b>yum groups install "Basic Web Server"</b>
</pre>

### Apache's Modular Architecture
* Modules:
  * The apache binary will load shared modules with the `LoadModule` directive.
  * Apache directives are a set of rules which define how your server should run, number of clients that can access your server, etc.
    * List of directives in Apache 2.4: https://httpd.apache.org/docs/2.4/mod/directives.html
  * The configuration file contains directives. Some of which require modules to be loaded to support their operation.
    * `/etc/httpd/conf/httpd.conf` (Ubuntu uses `/etc/apache2`) tells Apache which modules to load.
    * The default config file loads a large number of modules.
    * Alternatively, start with an empty list and load only the modules you need as your site evolves.
  * `sudo apachectl -M` displays loaded modules
  * List of modules: `/usr/lib64/httpd/modules`

<pre>
[root@web ~]# <b>tree /etc/httpd</b>
/etc/httpd
├── conf
│   ├── httpd.conf
│   └── magic
├── conf.d
│   ├── autoindex.conf
│   ├── manual.conf
│   ├── README
│   ├── userdir.conf
│   └── welcome.conf
├── conf.modules.d
│   ├── 00-base.conf
│   ├── 00-dav.conf
│   ├── 00-lua.conf
│   ├── 00-mpm.conf
│   ├── 00-proxy.conf
│   ├── 00-systemd.conf
│   └── 01-cgi.conf
├── logs -> ../../var/log/httpd
├── modules -> ../../usr/lib64/httpd/modules
└── run -> /run/httpd

6 directories, 14 files
</pre>

### A tour of the Configuration file
Configuration and logfile names: If the filenames do *not* begin
with `/`, the value of ServerRoot is prepended -- so `log/access_log`
with ServerRoot set to `/www` will be interpreted by the
server as `/www/log/access_log`, where as `/log/access_log` will be
interpreted as `/log/access_log`.

<pre>
[root@web ~]# <b>sed '/^\s*#/d;/^$/d' /etc/httpd/conf/httpd.conf</b></pre>
```apacheconf
ServerRoot "/etc/httpd"
Listen 80
Include conf.modules.d/*.conf
User apache
Group apache
ServerAdmin root@localhost
<Directory />
    AllowOverride none
    Require all denied
</Directory>
DocumentRoot "/var/www/html"
<Directory "/var/www">
    AllowOverride None
    Require all granted
</Directory>
<Directory "/var/www/html">
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>
<Files ".ht*">
    Require all denied
</Files>
ErrorLog "logs/error_log"
LogLevel warn
<IfModule log_config_module>
    LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
    LogFormat "%h %l %u %t \"%r\" %>s %b" common
    <IfModule logio_module>
      LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O" combinedio
    </IfModule>
    CustomLog "logs/access_log" combined
</IfModule>
<IfModule alias_module>
    ScriptAlias /cgi-bin/ "/var/www/cgi-bin/"
</IfModule>
<Directory "/var/www/cgi-bin">
    AllowOverride None
    Options None
    Require all granted
</Directory>
<IfModule mime_module>
    TypesConfig /etc/mime.types
    AddType application/x-compress .Z
    AddType application/x-gzip .gz .tgz
    AddType text/html .shtml
    AddOutputFilter INCLUDES .shtml
</IfModule>
AddDefaultCharset UTF-8
<IfModule mime_magic_module>
    MIMEMagicFile conf/magic
</IfModule>
EnableSendfile on
IncludeOptional conf.d/*.conf
```
* Directives:
  * `ServerName`: ServerName gives the name and port that the server uses to identify itself.
  This can often be determined automatically, but we recommend you specify it explicitly to prevent problems during startup.
  If your host doesn't have a registered DNS name, enter its IP address here. Example: `ServerName www.example.com:80`
  * `ServerRoot`: ServerRoot: The top of the directory tree under which the server's configuration, error, and log files are kept.
  Do not add a slash at the end of the directory path
  * `ServerAdmin`: Your address, where problems with the server should be e-mailed.
  * The `DocumentRoot` defines the top-level directory of the website.
  * `Listen 80`
  * `User` and `Group` Set an identity
    * Apache needs a user identity to run as.
      * User `apache` and group `apache` were added when the httpd package was installed.
      * Used to perform access control checks to the file system.
  * Deny access to the entirety of your server's filesystem. You must
explicitly permit access to web content directories in other `<Directory>` blocks below.
    * ```apcheconf
      <Directory />
      AllowOverride none
      Require all denied
      </Directory>
      ```
    * Note that from this point forward you must specifically allow
particular features to be enabled - so if something's not working as
you might expect, make sure that you have specifically enabled it below.
  * DocumentRoot: The directory out of which you will serve your documents. By default, all requests are taken from this directory
  * ```
    <Directory "/var/www">
    AllowOverride None
    Allow open access:
    Require all granted
    </Directory>
    ```
  * `DirectoryIndex`: We need `dir_module` for this. That is why it is in `<IfModule dir_module>` tag
  * `ErrorLog`: part of the core module
  * `LogFormat`: Fields to include in the log. For example `%h` means hostname of the client. `%u`: the username. `%t` timestamp.
  * To improve response time, apache manages a pool of *spare* servers processes. These numbers control the size of the pool
    * StartServers 8
    * MinSpareServers 5
    * MAxSpareServers 20
    * ServerLimit 256
    * MaxClients 256
#### Test the configuration
<pre>
[root@web ~]# <b>apachectl configtest</b>
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1. Set the '<b>ServerName</b>' directive globally to suppress this message
Syntax OK
</pre>
Then I added `ServerName web` to the `httpd.conf` file and there is no error
<pre>
[root@web ~]# <b>apachectl configtest</b>
Syntax OK
[root@web ~]# <b>systemctl restart httpd</b>
</pre>

### Virtual Hosts
Apache Virtual hosts will allow for different DocumentRoot settings for
sites accessed with different host names, IP addresses, or ports.

#### Virtual Host Blocks
This example shows a Named-Based Virtual Host. When accessing the host
`sales.example.com` we will be taken to the specified DocumentRoot.
Whilst in Apache 2.4 it is not required, Apache 2.2
and earlier need the Server directive `NameVirtualHost.
```apacheconf
<VirtualHost *:80>
  DocumentRoot "/srv/vhosts/sales"
  ServerName sales
  <Directory /srv/vhosts/sales >
    Require all granted
  </Directory>
</VirtualHost>
```
**LAB**
<pre>
[root@web ~]# <b>mkdir -p /srv/vhosts/sales</b>
[root@web ~]# <b>echo Sales > /srv/vhosts/sales/index.html</b>
[root@web ~]# <b>mkdir/etc/httpd/conf/extra</b>
[root@web ~]# <b>vim /etc/httpd/conf/extra/vhost-sales.conf</b></pre>
```apacheconf
<VirtualHost *:80>
  DocumentRoot "/srv/vhosts/sales"
  ServerName sales
  <Directory /srv/vhosts/sales >
    Require all granted
  </Directory>
</VirtualHost>
```
<pre>
[root@web ~]# <b>tail -n1 /etc/httpd/conf/httpd.conf</b> # I have added this line to the httpd.conf
Include conf/extra/vhost-*.conf
[root@web ~]# <b>apachectl configtest</b>
Syntax OK
[root@web ~]# <b>!sys</b>
systemctl restart httpd
[root@web ~]# <b>echo "127.0.0.1 sales" >> /etc/hosts</b>
</pre>

To test:
<pre>
[root@web ~]# <b>curl sales</b>
Sales
[root@web ~]# <b>curl web</b>
Sales
[root@web ~]# <b>curl localhost</b>
Sales
</pre>
We get the same result and this is not correct. We have to fix this.
<pre>
[root@web ~]# <b>cat /etc/httpd/conf/extra/vhost-web.conf</b></pre>

```apacheconf
<VirtualHost *:80>
  DocumentRoot "/var/www/html"
  ServerName web
  <Directory /var/www/html >
    Require all granted
  </Directory>
</VirtualHost>
```
<pre>
[root@web ~]# <b>apachectl configtest</b>
Syntax OK
[root@web ~]# <b>apachectl configtest</b>
Syntax OK
[root@web ~]# !sys
systemctl restart httpd
[root@web ~]# <b>curl web</b>
Hello
</pre>

**Creating port-based virtual host**
<pre>
[root@web ~]# vim !$
<b>vim /etc/httpd/conf/extra/vhost-port.conf</b></pre>
```apacheconf
listen 9000
<VirtualHost *:9000>
  DocumentRoot "/srv/vhosts/9000"
  <Directory /srv/vhosts/9000 >
    Require all granted
  </Directory>
</VirtualHost>
```

<pre>
[root@web ~]# <b>mkdir /srv/vhosts/9000</b>
[root@web ~]# <b>apachectl configtest</b>
Syntax OK
[root@web ~]# <b>echo 9000 > /srv/vhosts/9000/index.html</b>
[root@web ~]# <b>!sys</b>
systemctl restart httpd
[root@web ~]# <b>curl localhost:9000</b>
9000
</pre>
To create IP-based virtual host, we just have to make sure that we have got
multiple IP addresses assigned to our machine and then we can go through
and set up in our VirtualHost block, instead of putting the port that we want to go through,
where we've been using 9000, we would go through and specify the IP address
within that virtual host setting.
```apacheconf
<VirtualHost 10.1.1.1:80>
```

## Apache access Control
With Apache 2.4, access control is maintained using the `Require` directive for both hosts and users

### Server Status Page:
* Can be viewed via a web browser when this is enabled.
* Access to this page maybe restricted to the server itself and perhaps your management workstation.
* Configuring the status page:
  * For this we introduce the `Location` block and load two additional modules.
  * A location block is similar to a `Directory` block, however it is used to secure items outside of the filesystem,
    unlike the Directory block. In this case we use it to secure access to the module using the URI `/status`.

### Displaying and securing the server status page

<pre>
[root@web extra]# <b>vim vhost-p9000.conf</b>
</pre>
```apacheconf
listen 9000
<VirtualHost *:9000>
  DocumentRoot "/var/www/9000"
  <Directory /var/www/9000 >
    Require all granted
  </Directory>
  <Location "/status">
    SetHandler server-status
    Require ip 127.0.0.1
    Require ip 192.168.33.1
  </Location>
</VirtualHost>
```
<pre>
[root@web extra]# <b>systemctl restart httpd</b>
[root@web extra]# <b>curl -IL http://127.0.0.1:9000/status</b>
HTTP/1.1 <b>200 OK</b>
Date: Sat, 30 Dec 2017 20:36:32 GMT
Server: Apache/2.4.6 (CentOS)
Content-Length: 3397
Content-Type: text/html; charset=ISO-8859-1
</pre>

### User access
To be able to authenticate users we need to load additional 3 modules as a minimum.
An *Authentication Type* `auth`, and *Authentication Provider*, and an *Authorization module*.
We have to be sure that both `authn` (Authentication Provider) and `authz` (Authorization) modules are loaded
<pre>
LoadModule auth_basic_module modules/mod_auth_basic.so
LoadModule auth_file_module modules/mod_auth_file.so
LoadModule auth_user_module modules/mod_auth_user.so
LoadModule authn_core_module modules/mod_authn_core.so
LoadModule authz_core_module modules/mod_authz_core.so
</pre>

To manage file based users and passwords we have the `htpasswd` command.
* `htpasswd -c /etc/httpd/sales.pwd uwe` # Creates `sales.pwd` and add user `uwe`
* `htpasswd /etc/httpd/sales.pwd adolf`
* `htpasswd -D /etc/httpd/sales.pwd adolf` # Delete a user
* `htpasswd -v /etc/httpd/sales.pwd` # Verify the password

<pre>
[root@web ~]# <b>mkdir /var/www/html/download</b>
[root@web ~]# <b>cp /etc/hosts !$</b>
cp /etc/hosts /var/www/download
[root@web extra]# <b>vi vhost-web.conf</b></pre>
```apacheconf
<VirtualHost *:80>
  DocumentRoot "/var/www/html"
  ServerName web
  <Directory /var/www/html >
    Require all granted
  </Directory>
  <Directory /var/www/html/downloads >
    Options Indexes
    AuthType Basic
    AuthName "Only users allowed"
    AuthUserFile /etc/httpd/sales.pwd
    Require valid-user
  </Directory>
</VirtualHost>
```
<pre>
[root@web extra]# <b>!apach</b>
apachectl configtest
Syntax OK
[root@web extra]# <b>!sys</b>
systemctl restart httpd
[root@web extra]# <b>grep AuthUserFile vhost-web.conf</b>
    AuthUserFile /etc/httpd/sales.pwd
[root@web <b>httpd</b>]# <b>htpasswd -c sales.pwd uwe</b>
New password:
Re-type new password:
Adding password for user uwe
[root@web httpd]# <b>cat sales.pwd</b>
uwe:$apr1$UkN8uNOO$Z8PsUIa0AKsHyDWjUwhg10
[root@web httpd]# <b>echo passtest | htpasswd -i sales.pwd valerie</b> # This is a good way to add users via script
Adding password for user valerie
[root@web httpd]# <b>cat sales.pwd</b>
uwe:$apr1$UkN8uNOO$Z8PsUIa0AKsHyDWjUwhg10
valerie:$apr1$YAdST0m7$hywV4V05OolPZstp1pFdp1
</pre>

### Group access

What if the number of users grow? We'd better use groups.
<pre>
<b>[root@web httpd]# cat conf/extra/vhost-web.conf </b></pre>
```apacheconf
<VirtualHost *:80>
  DocumentRoot "/var/www/html"
  ServerName web
  <Directory /var/www/html >
    Require all granted
  </Directory>
  <Directory /var/www/html/downloads >
    Options Indexes
    AuthType Basic
    AuthName "Only users allowed"
    AuthUserFile /etc/httpd/sales.pwd
    AuthGroupFile /etc/httpd/groups
    Require group sales
  </Directory>
</VirtualHost>
```
<pre>
[root@web httpd]# <b>vim groups</b>
sales: uwe jo
</pre>
Now we can see user `valerie` has no longer access to download page but `uwe` has.

## Using scripts to deliver dynamic content
* vhost-web.conf

```apacheconf
<VirtualHost *:80>
  DocumentRoot "/var/www/html"
  ServerName web
  <Directory /var/www/html >
    Require all granted
  </Directory>
  <Directory /var/www/html/downloads >
    Options Indexes
    AuthType Basic
    AuthName "Only users allowed"
    AuthUserFile /etc/httpd/sales.pwd
    AuthGroupFile /etc/httpd/groups
    Require group sales
  </Directory>
  ScriptAlias "/cgi-bin/" "/var/www/cgi-bin/"
  <Directory "/var/www/cgi-bin" >
    AddHandler cgi-script .sh .pl
    Options +ExecCGI
    Require all granted
  </Directory>
</VirtualHost>
```

<pre>
[root@web cgi-bin]# <b>vim report.sh</b></pre>


```bash
#!/bin/bash
echo 'Content-type: text/html'
echo ''
echo "<h1>`uname -n`</h1>"
echo "<h2>Root Filesystem</h2>"
echo "`df -hT / | grep -v Filesystem`"
```

<pre>
[root@web cgi-bin]# <b>chmod +x report.sh</b>
[root@web cgi-bin]# <b>lynx http://web/cgi-bin/report.sh</b>
                                                                    web

Root Filesystem

   /dev/mapper/centos-root xfs 39G 924M 38G 3% /
</pre>

### .htaccess
A `.htaccess` file provides a way to make configuration changes on a per-directory basis
without the need of restarting the server when configuration changes are made.

As an alternative to adding the settings within the virtual host we can create a
`.htaccess` file. Add this file to the `/var/www/cgi-bin` folder.

**Lab**

* vhost-web.conf

```apacheconf
<VirtualHost *:80>
  DocumentRoot "/var/www/html"
  ServerName web
  <Directory /var/www/html >
    Require all granted
  </Directory>
  <Directory /var/www/html/downloads >
    Options Indexes
    AuthType Basic
    AuthName "Only users allowed"
    AuthUserFile /etc/httpd/sales.pwd
    AuthGroupFile /etc/httpd/groups
    Require group sales
  </Directory>
  ScriptAlias "/cgi-bin/" "/var/www/cgi-bin/"
  <Directory "/var/www/cgi-bin" >
    AllowOverride AuthConfig Options FileInfo
  </Directory>
</VirtualHost>
```

<pre>
[root@web extra]# <b>cd /var/www/cgi-bin/</b>
[root@web cgi-bin]# <b>vim .htaccess</b></pre>

```apacheconf
AddHandler cgi-script .sh .pl
Options +ExecCGI
Require all granted
```

Verify with:
<pre>
[root@web cgi-bin]# <b>lynx http://web/cgi-bin/report.sh</b></pre>

### PHP and Apache
* `yum install php php-apache`
*  `mod_php` can't work with MPM worker; so we have to run apache in prefork mode
  * `#LoadModule mpm_event_modules module/mod_mpm_event.so`
  * `LoadModule mpm_prefork_module modules/mod_mpm_prefork.so`

## Securing Apache with HTTPS
### CIA Triad
* Confidentiality: Encryption
* Integrity: Hash
* Authenticity: Authentication

### 1. Creating private keys

#### Server private key
<pre>
[root@web ~]# <b>openssl version</b> # Should be 1.0.1f or later
OpenSSL <b>1.0.2k</b>-fips  26 Jan 2017
[root@web ~]# <b>cd /etc/httpd/conf</b>
[root@web conf]# <b>openssl genrsa -out server.key 2048</b>
Generating RSA private key, 2048 bit long modulus
........................+++
............+++
e is 65537 (0x10001)
[root@web conf]# <b>cat server.key</b>
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAohhJ6mkJm+9FjZh7EuXahl1BHEJ9fAGW6OfKSXY09bs98VWe
ni/VCLX6oAXEoj30ZxeAqouewXh9TmZovNSP9XjNoh6HcWlrXx01b3M3XS0QOAXZ
Wke3vPRTtqIkiN9ZUrQsTi/Z6jFRFsnuUgXQEq7wPqCm40EJp8qRlB3rPe1vl0Gs
...
[root@web conf]# <b>openssl rsa -noout -text -in server.key</b>
Private-Key: (2048 bit)
modulus:
    00:a2:18:49:ea:69:09:9b:ef:45:8d:98:7b:12:e5:
    da:86:5d:41:1c:42:7d:7c:01:96:e8:e7:ca:49:76:
...
</pre>

### 2. Creating a certificate signing request (CSR)
<pre>
[root@web conf]# <b>openssl req -new -key server.key -out server.csr</b>
</pre>

### 3. Self-sign the SCR
<pre>
[root@web conf]# <b>openssl x509 -req -sha256 -in server.csr -signkey server.key -out server.crt</b>
Signature ok
subject=/C=XX/L=Default City/O=Default Company Ltd/CN=web
Getting Private key
[root@web conf]# <b>openssl x509 -noout -text -in server.crt</b>
Certificate:
    Data:
        Version: 1 (0x0)
        Serial Number:
            ab:74:22:f8:49:05:c0:f3
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=XX, L=Default City, O=Default Company Ltd, CN=web
        Validity
            Not Before: Dec 31 02:23:11 2017 GMT
            Not After : Jan 30 02:23:11 2018 GMT
        Subject: C=XX, L=Default City, O=Default Company Ltd, CN=web
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:a2:18:49:ea:69:09:9b:ef:45:8d:98:7b:12:e5:
                    ...
    Signature Algorithm: sha256WithRSAEncryption
         4c:86:72:45:de:1d:76:66:41:14:fa:2f:04:b0:24:b4:fb:02:
         ...
</pre>
Should you need more information, use manual pages for `x509`, `rsa`, `req`

### 4. Configure Apache to use TLS
* First in `/etc/httpd/conf/httpd.conf` include `listen 443` then be sure that
`ssl_module modules/mod_ssl.so` is loaded, and you have your cipher suits:

```apacheconf
SSLCipherSuite HIGH:MEDIUM:!3DES:!RC4:MD5:!SSLv3
SSLHonorCipherOrder on
SSLProtocol all -SSLv2 -SSLv3
```

And the VirtualHost
```apacheconf
<VirtualHost *:80>
  DocumentRoot "/var/www/lab"
  ServerName lab
  <Directory /var/www/lab >
    Require all granted
  </Directory>
</VirtualHost>

<VirtualHost *:443>
  DocumentRoot "/var/www/lab"
  ServerName lab
  SSLEngine on
  DirectoryIndex "index.html"
  SSLCertificateFile "/etc/httpd/conf/server.crt"
  SSLCertificateKeyFile "/etc/httpd/conf/server.key"
  <Directory /var/www/lab >
    Require all granted
  </Directory>
</VirtualHost>
```
To test:
<pre>
[root@web conf]# <b>lynx https://192.168.33.20/</b></pre>

### Redirect HTTP to HTTPS
These 2 lines added:
* `Redirect permanent / https://192.168.33.20` to `VirtualHost *:80`
* `Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"` to `VirtualHost *:443`
```apacheconf
<VirtualHost *:80>
  DocumentRoot "/var/www/lab"
  ServerName lab
  Redirect permanent / https://192.168.33.20
  <Directory /var/www/lab >
    Require all granted
  </Directory>
</VirtualHost>

<VirtualHost *:443>
  DocumentRoot "/var/www/lab"
  Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
  ServerName lab
  SSLEngine on
  DirectoryIndex "index.html"
  SSLCertificateFile "/etc/httpd/conf/server.crt"
  SSLCertificateKeyFile "/etc/httpd/conf/server.key"
  <Directory /var/www/lab >
    Require all granted
  </Directory>
</VirtualHost>
```
Verify
* Open a browser and try to login with `http://`
* chrome://net-internals/#hsts

## Load balancing http requests
* In real world, the load balancer would have public and private IPs.
Internal hosts would have private IPs only
* In our scenario, we will be having an Apache Virtual Host balancer and
2 Apache Virtual Hosts (Host1 and Host2 only on the private network)
* `mod_proxy`, `mod_proxy_balancer`, `mod_proxy_http`, `mod_lbmethod_byrequests`,
and `mod_slotmem_shm` will be needed.
* We need to disable the standard proxy: `ProxyRequest off`
* We never use the balanver to display any content, our content will be on **host1** and **host2**.

**Lab**

<pre>
[root@web ~]# <b>echo "127.0.0.1 balancer" >> /etc/hosts</b>
[root@web ~]# <b>echo "127.0.0.1 host1" >> /etc/hosts</b>
[root@web ~]# <b>echo "127.0.0.1 host2" >> /etc/hosts</b>
[root@web ~]# <b>mkdir /var/www/balancer</b>
[root@web ~]# <b>mkdir /var/www/host{1,2}</b>
[root@web ~]# <b>ls /var/www</b>
9000  <b>balancer</b>  cgi-bin  <b>host1  host2</b>  html  lab  sales
[root@web ~]# echo "Welcome to host 1" > /var/www/host1/index.html
[root@web ~]# echo "Welcome to host 2" > /var/www/host2/index.html
</pre>

`proxy.con` file

```apacheconf
[root@web extra]# cat proxy.conf
ProxyRequests off
<VirtualHost *:80 >
  DocumentRoot /var/www/balancer
  ServerName balancer
  <Directory /var/www/balancer >
    Require all granted
  </Directory>
  <Proxy balancer://webfarm>
    BalancerMember http://host1:80
    BalancerMember http://host2:80
    ProxySet lbmethod=byrequests
  </Proxy>
  ProxyPass "/" "balancer://webfarm/"
  ProxyPassReverse "/" "balancer://webfarm/"
</VirtualHost>

<VirtualHost *:80 >
  DocumentRoot /var/www/host1
  ServerName host1
  <Directory /var/www/host1 >
    Require all granted
  </Directory>
</VirtualHost>

<VirtualHost *:80 >
  DocumentRoot /var/www/host2
  ServerName host2
  <Directory /var/www/host2 >
    Require all granted
  </Directory>
</VirtualHost>
```

`httpd.conf` file
```apacheconf
ServerRoot "/etc/httpd"
ServerName web
Listen 80
Listen 443
Include conf.modules.d/*.conf
User apache
Group apache
ServerAdmin root@localhost
<Directory />
    AllowOverride none
    Require all denied
</Directory>
DocumentRoot "/var/www/html"
<Directory "/var/www">
    AllowOverride None
    Require all granted
</Directory>
<Directory "/var/www/html">
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
    DirectoryIndex index.html
<Files ".ht*">
    Require all denied
</Files>
ErrorLog "logs/error_log"
LogLevel warn
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
LogFormat "%h %l %u %t \"%r\" %>s %b" common
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O" combinedio
CustomLog "logs/access_log" combined
ScriptAlias /cgi-bin/ "/var/www/cgi-bin/"
<Directory "/var/www/cgi-bin">
    AllowOverride None
    Options None
    Require all granted
</Directory>
    TypesConfig /etc/mime.types
    AddType application/x-compress .Z
    AddType application/x-gzip .gz .tgz
    AddType text/html .shtml
    AddOutputFilter INCLUDES .shtml
AddDefaultCharset UTF-8
    MIMEMagicFile conf/magic
EnableSendfile on
Include conf/extra/proxy.conf
```
To test:
<pre>
[root@web conf]# <b>lynx http://balancer/</b></pre>
Then refresh the page multiple time using `ctrl+r`
* Here we have 2 different contents but in reality we have the same contents

# Implementing a web proxy with squid

### Installation
<pre>
[root@web ~]# <b>yum install squid</b>
[root@web ~]# <b>systemctl enable squid</b>
ln -s '/usr/lib/systemd/system/squid.service' '/etc/systemd/system/multi-user.target.wants/squid.service'
[root@web ~]# <b>systemctl start squid</b>
</pre>

### Investigating the Squid configuration file
* ACL directives: Squid ACL entries are used to name entities to be used to control
rsource access. They consist of a name, acl type, and value.
* HTTP_Access: To gain access to resources, ACL names can be included with the http_acess directive.
It is often that the last http_access will deny any non-matched entries.
* Squid normally listens to port 3128.

<pre>
[root@web ~]# <b>cd /etc/squid/</b>
[root@web squid]# <b>sed '/^#/d;/^$/d' squid.conf</b>
acl localnet src 10.0.0.0/8	# RFC1918 possible internal network
acl localnet src 172.16.0.0/12	# RFC1918 possible internal network
acl localnet src 192.168.0.0/16	# RFC1918 possible internal network
acl localnet src fc00::/7       # RFC 4193 local private network range
acl localnet src fe80::/10      # RFC 4291 link-local (directly plugged) machines
acl SSL_ports port 443
acl Safe_ports port 80		# http
acl Safe_ports port 21		# ftp
acl Safe_ports port 443		# https
acl Safe_ports port 70		# gopher
acl Safe_ports port 210		# wais
acl Safe_ports port 1025-65535	# unregistered ports
acl Safe_ports port 280		# http-mgmt
acl Safe_ports port 488		# gss-http
acl Safe_ports port 591		# filemaker
acl Safe_ports port 777		# multiling http
acl CONNECT method CONNECT
http_access deny !Safe_ports
http_access deny CONNECT !SSL_ports
http_access allow localhost manager
http_access deny manager
http_access allow localnet
http_access allow localhost
http_access deny all
http_port 3128
coredump_dir /var/spool/squid
refresh_pattern ^ftp:		1440	20%	10080
refresh_pattern ^gopher:	1440	0%	1440
refresh_pattern -i (/cgi-bin/|\?) 0	0%	0
refresh_pattern .		0	20%	4320
[root@web squid]# <b>ss -ltn</b>
State      Recv-Q Send-Q                                      Local Address:Port                                        Peer Address:Port
LISTEN     0      128                                                     *:22                                                     *:*
LISTEN     0      100                                             127.0.0.1:25                                                     *:*
LISTEN     0      128                                                    :::80                                                    :::*
LISTEN     0      128                                                    :::22                                                    :::*
LISTEN     0      128                                                    :::3128                                                  :::*
LISTEN     0      100                                                   ::1:25                                                    :::*
LISTEN     0      128                                                    :::443                                                   :::*
</pre>

Then configure your browser to use the proxy:

![image](https://user-images.githubusercontent.com/31813625/34464410-99978328-ee4b-11e7-9cce-e0fbbd5edd22.png)

Now check the Squid log to verify
<pre>
[root@web squid]# <b>tail -f /var/log/squid/access.log</b></pre>

### Authenticating users

<pre>
[root@web squid]# <b>htpasswd -c squid.users user1</b>
New password:
Re-type new password:
Adding password for user user1
[root@web squid]# <b>!sed</b>
sed '/^#/d;/^$/d' squid.conf
<b>auth_param basic program /usr/lib64/squid/basic_ncsa_auth /etc/squid/squid.users
acl ncsa_users proxy_auth REQUIRED</b>
acl localnet src 10.0.0.0/8	# RFC1918 possible internal network
acl localnet src 172.16.0.0/12	# RFC1918 possible internal network
acl localnet src 192.168.0.0/16	# RFC1918 possible internal network
acl localnet src fc00::/7       # RFC 4193 local private network range
acl localnet src fe80::/10      # RFC 4291 link-local (directly plugged) machines
acl SSL_ports port 443
acl Safe_ports port 80		# http
acl Safe_ports port 21		# ftp
acl Safe_ports port 443		# https
acl Safe_ports port 70		# gopher
acl Safe_ports port 210		# wais
acl Safe_ports port 1025-65535	# unregistered ports
acl Safe_ports port 280		# http-mgmt
acl Safe_ports port 488		# gss-http
acl Safe_ports port 591		# filemaker
acl Safe_ports port 777		# multiling http
acl CONNECT method CONNECT
http_access deny !Safe_ports
http_access deny CONNECT !SSL_ports
http_access allow localhost manager
http_access deny manager
<b># http_access allow localnet</b>
http_access allow localhost
<b>http_access allow ncsa_users</b>
http_access deny all
http_port 3128
coredump_dir /var/spool/squid
refresh_pattern ^ftp:		1440	20%	10080
refresh_pattern ^gopher:	1440	0%	1440
refresh_pattern -i (/cgi-bin/|\?) 0	0%	0
refresh_pattern .		0	20%	4320
</pre>

# NGINX
* The NGINX Web Server is part of the core Arch repositories. Likewise with Ubuntu
but CentOS will require you to add EPEL repository
<pre>
[root@web ~]# <b>yum install epel-release</b>
[root@web ~]# <b>yum install nginx</b>
[root@web ~]# <b>systemctl stop httpd</b>
[root@web ~]# <b>systemctl disable httpd</b>
rm '/etc/systemd/system/multi-user.target.wants/httpd.service'
[root@web ~]# <b>systemctl start nginx</b>
[root@web ~]# <b>systemctl enable nginx</b> # to start on boot
ln -s '/usr/lib/systemd/system/nginx.service' '/etc/systemd/system/multi-user.target.wants/nginx.service'
</pre>

* Configuring NGINX
<pre>
[root@web ~]# <b>cd /etc/nginx/</b>
[root@web nginx]# <b>sed -i.`date +%F` '/^\s*#/d;/^$/d' nginx.conf</b>
[root@web nginx]# <b>cat nginx.conf</b>
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
include /usr/share/nginx/modules/*.conf;
events {
    worker_connections 1024;
}
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;
    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;
    include /etc/nginx/conf.d/*.conf;
    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;
        include /etc/nginx/default.d/*.conf;
        location / {
        }
        error_page 404 /404.html;
            location = /40x.html {
        }
        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
}
</pre>

### Reverse Proxy
NGINX is fast at delivering static content but not PHP pages.
We might pass PHP to Apache Servers; or in our case, we can make use of
the Apache Load Balancer to demonstrate Reverse Proxy.

Where we use proxy_pass in a location block we can redirect the URL to the
reverse proxied location. Please note that the end of the location URL
and the ProxyPass URL must be a forward slash when referring to directories
rather than pages.
<pre>
http {
    server {
        location /balancer/ { # New location block in the server block
            proxy_pass http://192.168.33.20/;
        }
</pre>
To test
<pre>
lynx 192.168.33.20/balancer/
</pre>