---
title: Installing OpenLDAP in a jail on FreeNAS
excerpt: >
    Learn how to install OpenLDAP in an iocage jail on your FreeNAS server.
date: 2019-06-23
categories:
tags:
  - OpenLDAP
  - iocage
  - jails
  - certificates
toc: true
---

An LDAP directory provides a central location to store information about the
users and computers in your organization. Business applications can use this
information to support services such as file sharing, collaboration and chat.

In this tutorial, learn how to install [OpenLDAP][0]{: target="external"}—an
open source LDAP server—in a jail on your FreeNAS appliance. This tutorial also
shows how to configure transport layer security (TLS) using a
[Let's Encrypt][1]{: target="external"} certificate.

## Prerequisites

To complete this tutorial, you need:

* Name resolution for the ldap server
* [Let's Encrypt][4]{: target="external"} certificate

## Preparing the jail

Run the following procedure from a session in your FreeNAS server. You can use
the [FreeNAS shell][2]{: target="external"} for this purpose.

1. Fetch or update your release of FreeBSD for jail usage:
   ```sh
   iocage fetch --release 11.2-RELEASE
   ```
   You can check your current release with the `freebsd-version` command.
1. Create the jail. The following example sets the name to `ldapserver`:
   ```sh
   iocage create --name ldapserver --release 11.2-RELEASE
   ```
1. Configure the IP address. The following example sets the IP address to
   `192.168.1.123` using a subnet mask of `255.255.255.0` on the `re0`
   interface. The command uses the [CIDR notation][10]{: target="external"},
   which translates to `192.168.1.123/24`:
   ```sh
   iocage set ip4_addr="re0|192.168.1.123/24" ldapserver
   ```
1. Configure the default router. The following example sets the default router
   to `192.168.1.1`:
   ```sh
   iocage set defaultrouter=192.168.1.1 ldapserver
   ```
1. Start the jail and open a session:
    ```sh
    iocage console ldapserver
    ```
1. Once in the jail session, install the required packages:
   ```sh
   pkg install --yes openldap-server
   pkg install --yes openssl
   pkg install --yes perl5
   pkg install --yes ca_root_nss
   ```
1. Copy the your certificate files to the jail, including the files with the
   `crt`, `key`, and `fullchain` extensions. Register the certificate with the
   following command:
   ```sh
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
   * Instances of `ldap.example.org`, which you should replace with the host of
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
   ```sh
   service slapd stop
   ```
1. Delete current configuration database:
   ```sh
   rm -rf /usr/local/etc/openldap/slapd.d
   mkdir /usr/local/etc/openldap/slapd.d
   ```
1. Delete current data folder:
   ```sh
   rm -rf /usr/local/etc/openldap/data
   mkdir /usr/local/etc/openldap/data
   ```
1. Import the `slapd.ldif` file:
   ```sh
   /usr/local/sbin/slapadd -n0 -F /usr/local/etc/openldap/slapd.d/ -l slapd.ldif
   ```
1. Start the service:
   ```sh
   /usr/local/libexec/slapd -F /usr/local/etc/openldap/slapd.d/
   ```
1. Load the base objects of the domain. Replace `cn=admin,dc=example,dc=org`
   with the user ID of your LDAP administrator account.
   ```sh
   ldapadd -W -D "cn=admin,dc=example,dc=org" -f domain.ldif
   ```

## Configuring the service startup

Use the `sysrc` command to start the LDAP service when the jail boots. Replace
`ldap.example.org` with the hostname of your LDAP server.

```sh
sysrc slapd_enable="YES"
sysrc slapd_flags='-h "ldapi://%2fvar%2frun%2fopenldap%2fldapi/ ldap://ldap.example.org/"'
sysrc slapd_sockets="/var/run/openldap/ldapi"
sysrc slapd_cn_config="YES"
```

This is a good time to restart the jail. Exit the session and restart the jail
from the [Jails][3]{: target="external"} page or running the following command
on the [Shell][2]{: target="external"} page:

```sh
iocage restart ldapserver
```

## Querying the OpenLDAP service

To check that the service is running, you can use a query from a host that has
the LDAP tools installed. In Debian-based distributions of Linux, such as
Ubuntu, you can install the LDAP tools with the following command:

```sh
apt install ldap-utils
```

The following queries test different aspects of the service:
* To query the service anonymously:
  ```sh
  ldapwhoami -H ldap://ldap.example.org -x
  ```
  Expected output: `anonymous`
* To query the service in the context of a user:
  ```sh
  ldapwhoami -H ldap://ldap.example.org -x -D "uid=admin@example.org,dc=example,dc=org" -W
  ```
  Expected output: `dn:uid=admin@example.org,dc=example,dc=org`
* To query the service using TLS:
  ```sh
  ldapwhoami -H ldap://ldap.example.org -x -ZZ -D "uid=admin@example.org,dc=example,dc=org" -W
  ```
  Expected output: `dn:uid=admin@example.org,dc=example,dc=org`

You can add more objects to your LDAP database using the
[`ldapmodify`][5]{: target="external"} or `ldapadd` commands.

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
