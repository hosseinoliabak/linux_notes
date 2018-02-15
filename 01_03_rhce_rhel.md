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
