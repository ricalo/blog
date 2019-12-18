---
title: Installing an OpenLDAP server on FreeNAS
excerpt: >
    Learn how to install an OpenLDAP server in a jail on your FreeNAS server.
    Use OpenLDAP as a central store of user information that you can use in
    business applications, such as file sharing, collaboration, and chat.
date: 2019-06-23
categories:
tags:
  - openldap
  - iocage
  - jails
  - certificates
  - apache
toc: true
---

In this tutorial, learn how to install [OpenLDAP][0]{: target="external"}—an
open source LDAP server—in a jail on your FreeNAS appliance. This tutorial also
shows how to configure transport layer security (TLS) using a
[Let's Encrypt][1]{: target="external"} certificate and how to install
[phpLDAPadmin][12]{: target="external"} to manage the directory using a web
browser.

{% include devpromedia/create-jail.md
   jail-name="ldapserver"
   packages="openldap-server openssl perl5 ca_root_nss" %}

Finally, copy the certificate files to the jail, including the files with the
`crt`, `key`, and `fullchain` extensions. Then, register the certificate with
the following command:

```shell
c_rehash PATH_TO_CERTIFICATE_FILES
```

## Creating the database definition files

You can use the LDAP Data Interchange Format (LDIF) to define the OpenLDAP
databases. You need the following information to create the LDIF files:

* The hostname of the LDAP server
* The absolute path to the files of your certificate

You need an LDIF file that defines the configuration database and another that
defines the base objects of the domain. The `slapd.ldif` and `domain.ldif` files
represent the configuration database and base domain objects, respectively.

To create the `slapd.ldif` file:

1. Create a hashed password for the LDAP administrator account using the
   `slappasswd` command. Save the original password and take note of the hashed
   password, which uses the following notation:
   ```
   {SSHA}hashed_password
   ```
1. Using a text editor, create a `slapd.ldif` file using the sample below. Edit
   the following sections of the file to suit your needs:
   * The `olcTLSCertificate` attributes, which specify the location of your
     certificate files.
   * The `olcAccess` attribute of the frontend settings. The value of the
     attribute specified below defines the following access level:
       * Users can edit their own entries.
       * Authenticated users can read all entries.
   * The `olcRootPW` attribute, which contains the hashed password of the
     administrator account. The `slapd.ldif` files uses this attribute twice.
   * Instances of `ldapserver.example.org`, which you should replace with the host of
     your LDAP server.
   * Instances of `dc=example,dc=org`, which you should replace with your own
     domain components.

   ```yaml
   # See slapd-config(5) for details on configuration options.
   # This file should NOT be world readable.
   dn: cn=config
   objectClass: olcGlobal
   cn: config

   # Define global ACLs to disable default read access.
   olcArgsFile: /var/run/openldap/slapd.args
   olcPidFile: /var/run/openldap/slapd.pid
   olcTLSCertificateFile: ABSOLUTE_PATH_TO_YOUR_CERTIFICATE'S_CRT_FILE
   olcTLSCertificateKeyFile: ABSOLUTE_PATH_TO_YOUR_CERTIFICATE'S_KEY_FILE
   olcTLSCACertificateFile: ABSOLUTE_PATH_TO_YOUR_CERTIFICATE'S_FULLCHAIN_FILE
   olcTLSCipherSuite: ALL:!NULL
   olcTLSProtocolMin: 3.1
   olcTLSVerifyClient: never

   # Load dynamic backend modules:
   dn: cn=module,cn=config
   objectClass: olcModuleList
   cn: module
   olcModulepath: /usr/local/libexec/openldap
   olcModuleload: back_mdb.la

   dn: cn=schema,cn=config
   objectClass: olcSchemaConfig
   cn: schema

   include: file:///usr/local/etc/openldap/schema/core.ldif
   include: file:///usr/local/etc/openldap/schema/cosine.ldif
   include: file:///usr/local/etc/openldap/schema/inetorgperson.ldif
   include: file:///usr/local/etc/openldap/schema/nis.ldif

   # Frontend settings
   dn: olcDatabase={-1}frontend,cn=config
   objectClass: olcDatabaseConfig
   objectClass: olcFrontendConfig
   olcDatabase: {-1}frontend
   olcAccess: to *
          by self write
          by users read
          by anonymous auth

   dn: olcDatabase={0}config,cn=config
   objectClass: olcDatabaseConfig
   olcDatabase: {0}config
   olcAccess: to * by * none
   olcRootPW: HASHED_PASSWORD_INCLUDING_{SSHA}_PREFIX

   dn: olcOverlay=syncprov,olcDatabase={0}config,cn=config
   objectClass: olcOverlayConfig
   objectClass: olcSyncProvConfig
   olcOverlay: syncprov

   # LMDB database definitions
   dn: olcDatabase={1}mdb,cn=config
   objectClass: olcDatabaseConfig
   objectClass: olcMdbConfig
   olcDatabase: {1}mdb
   olcSuffix: dc=example,dc=org
   olcRootDN: cn=admin,dc=example,dc=org
   olcRootPW: HASHED_PASSWORD_INCLUDING_{SSHA}_PREFIX
   olcDbDirectory: /usr/local/etc/openldap/data
   olcDbIndex: objectClass eq

   dn: olcOverlay=syncprov,olcDatabase={1}mdb,cn=config
   objectClass: olcOverlayConfig
   objectClass: olcSyncProvConfig
   olcOverlay: syncprov
```

The `domain.ldif` file includes the following base objects of the domain:
* The *organization* object, which serves as the top level object in the
  database.
* A *groups* organizational unit, which hosts all the groups in the database.
* A *users* group, which hosts the users of the organization.
* An example user.

To create the `domain.ldif` file:

1. Create a hashed password for the example user account using the `slappasswd`
   command. Save the original password and take note of the hashed password,
   which uses the following notation:
   ```
   {SSHA}hashed_password
   ```
1. Using a text editor, create a `domain.ldif` file using the sample below. Edit
   the following sections of the file to suit your needs:
   * Instances of `dc=example,dc=org`, which you should replace with your own
     domain components.
   * Instances of `example.org`, which you should replace with the domain part
     of your LDAP server.
   * The `userPassword` attribute, which contains the hashed password of the
     example user account.

   ```yaml
   dn: dc=example,dc=org
   objectClass: dcObject
   objectClass: organization
   o: example.org
   dc: home

   dn: ou=groups,dc=example,dc=org
   objectClass: top
   objectClass: organizationalunit
   ou: groups

   dn: ou=users,dc=example,dc=org
   objectClass: top
   objectClass: organizationalunit
   ou: users

   dn: uid=user@example.org,ou=users,dc=example,dc=org
   objectClass: top
   objectClass: posixAccount
   objectClass: shadowAccount
   objectClass: inetOrgPerson
   cn: Example User
   givenName: Example
   sn: User
   mail: user@example.org
   uid: user@example.org
   uidNumber: 10001
   gidNumber: 10001
   homeDirectory: /home/user/
   loginShell: /bin/bash
   userPassword: HASHED_PASSWORD_INCLUDING_{SSHA}_PREFIX
   ```

## Loading the database definition files

1. Stop the sladp service:
   ```shell
   service slapd stop
   ```
1. Delete current configuration database:
   ```shell
   rm -rf /usr/local/etc/openldap/slapd.d
   mkdir /usr/local/etc/openldap/slapd.d
   ```
1. Delete current data folder:
   ```shell
   rm -rf /usr/local/etc/openldap/data
   mkdir /usr/local/etc/openldap/data
   ```
1. Import the `slapd.ldif` file:
   ```shell
   /usr/local/sbin/slapadd -n0 -F /usr/local/etc/openldap/slapd.d/ -l slapd.ldif
   ```
1. Start the service:
   ```shell
   /usr/local/libexec/slapd -F /usr/local/etc/openldap/slapd.d/
   ```
1. Load the base objects of the domain. Replace `cn=admin,dc=example,dc=org`
   with the user ID of your LDAP administrator account.
   ```shell
   ldapadd -W -D "cn=admin,dc=example,dc=org" -f domain.ldif
   ```

## Configuring the service startup

Use the `sysrc` command to start the LDAP service when the jail boots. Replace
`ldapserver.example.org` with the hostname of your LDAP server.

```shell
sysrc slapd_enable="YES"
sysrc slapd_flags='-h "ldapi://%2fvar%2frun%2fopenldap%2fldapi/ ldap://ldapserver.example.org/"'
sysrc slapd_sockets="/var/run/openldap/ldapi"
sysrc slapd_cn_config="YES"
```

This is a good time to restart the jail. Exit the session and restart the jail
from the [Jails][3]{: target="external"} page or running the following command
on the [Shell][2]{: target="external"} page:

```shell
iocage restart ldapserver
```

## Querying the OpenLDAP service

To check that the service is running, you can use a query from a host that has
the LDAP tools installed. In Debian-based distributions of Linux, such as
Ubuntu, you can install the LDAP tools with the following command:

```shell
apt install ldap-utils
```

The following queries test different aspects of the service:
* To query the service anonymously:
  ```shell
  ldapwhoami -H ldap://ldapserver.example.org -x
  ```
  Expected output: `anonymous`
* To query the service in the context of a user:
  ```shell
  ldapwhoami -H ldap://ldapserver.example.org -x -D "uid=admin@example.org,dc=example,dc=org" -W
  ```
  Expected output: `dn:uid=admin@example.org,dc=example,dc=org`
* To query the service using TLS:
  ```shell
  ldapwhoami -H ldap://ldapserver.example.org -x -ZZ -D "uid=admin@example.org,dc=example,dc=org" -W
  ```
  Expected output: `dn:uid=admin@example.org,dc=example,dc=org`

You can add more objects to your LDAP database using the
[`ldapmodify`][5]{: target="external"} or `ldapadd` commands.


## (Optional) Installing phpLDAPadmin

[phpLDAPadmin][11]{: target="external"} is a web-based LDAP client that you can
use to manage your directory. To use phpLDAPadmin:

Install the required packages and enable the web server:

1. Install the apache24, mod_php73, and phpldapadmin-php73 packages:
   ```shell
   pkg install --yes apache24 mod_php73 phpldapadmin-php73
   ```
1. Enable the web server at startup:
   ```
   sysrc apache24_enable="yes"
   ```

To configure the web server, update the following entries in the
`/usr/local/etc/apache24/httpd.conf` file:

1. To configure the web server to process php files using the PHP module, add
   the following section:
   ```xml
   <FilesMatch "\.php$">
    SetHandler application/x-httpd-php
   </FilesMatch>
   <FilesMatch "\.phps$">
       SetHandler application/x-httpd-php-source
   </FilesMatch>
   ```
1. To configure `index.php` as the resource to look for when clients request an
   index of the directory, add `index.php` to the `DirectoryIndex` entry:
   ```xml
   <IfModule dir_module>
       DirectoryIndex index.php index.html
   </IfModule>
   ```
   Note that the previous `DirectoryIndex` entry should already exist. Just add
   `index.php` to the list of files.
1. Replace `/usr/local/www/apache24/data` in the `DocumentRoot` and `Directory`
   entries with `/usr/local/www/phpldapadmin/htdocs`:
   ```xml
   DocumentRoot "/usr/local/www/apache24/data"
   <Directory "/usr/local/www/apache24/data">
   ```

To configure phpLDAPadmin, add the following entries to the
`/usr/local/www/phpldapadmin/config/config.php` file:
```php
$servers->setValue('server','host','ldapserver.example.org');
$servers->setValue('server','port',389);
$servers->setValue('server','base',array('dc=example,dc=org'));
$servers->setValue('login','auth_type','session');
$servers->setValue('server','tls',true);
```

To configure access over HTTPS, you need an SSL certificate, such as the ones
provided by [Let's Encrypt][12]{: target="external"}. Copy the `crt` and `key`
files of your certificate to a folder in the jail. To configure your
certificate:

1. In the `/usr/local/etc/apache24/httpd.conf` file:
   * Uncomment the LoadModule directives to enable the SSL and socache_shmcb
     modules by removing the leading # character:
     ```
     LoadModule ssl_module libexec/apache24/mod_ssl.so
     LoadModule socache_shmcb_module libexec/apache24/mod_socache_shmcb.so 
     ```
   * Uncomment the Include directive to add the SSL configuration file:
     ```
     Include etc/apache24/extra/httpd-ssl.conf
     ```
1. In the `/usr/local/etc/apache24/extra/httpd-ssl.conf` file.
   * Replace the DocumentRoot declaration. Replace the following line:
     ```
     DocumentRoot "/usr/local/www/apache24/data"
     ```
     with
     ```
     DocumentRoot "/usr/local/www/phpldapadmin/htdocs"
     ```
   * Replace the ServerName declaration. Replace the following line:
     ```
     ServerName www.example.org:443
     ```
     with the server name that matches your host and the server on the
     certificate:
     ```
     ServerName ldapserver.example.org:443
     ```
   * Update the `SSLCertificateFile` and `SSLCertificateKeyFile` entries with
     the path of the `crt` and `key` files of your certificate:
     ```
     SSLCertificateFile "path_to_your_certificate.crt" 
     SSLCertificateKeyFile "path_to_your_certificate.key"
     ```

Restart the web server:
```shell
service apache24 onerestart
```

Go to `https://ldapserver.example.org` and sign in with the
`cn=admin,dc=example,dc=org` username.


[0]: https://www.openldap.org/
[1]: https://letsencrypt.org/
[2]: https://www.ixsystems.com/documentation/freenas/11.2-U4.1/shell.html
[3]: https://www.ixsystems.com/documentation/freenas/11.2-U4.1/jails.html
[4]: https://letsencrypt.org/
[5]: https://www.openldap.org/software//man.cgi?query=ldapadd&sektion=1&apropos=0&manpath=OpenLDAP+2.4-Release
[6]: https://nextcloud.com/
[7]: https://openvpn.net/
[8]: https://www.pfsense.org
[9]: https://www.samueldowling.com/2018/12/08/install-nextcloud-on-freenas-iocage-jail-with-hardened-security/
[10]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[11]: http://phpldapadmin.sourceforge.net/
[12]: http://letsencrypt.org
