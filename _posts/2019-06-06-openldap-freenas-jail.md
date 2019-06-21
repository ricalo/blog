---
title: Installing OpenLDAP in a jail on FreeNAS
excerpt: >
    Learn how to install an OpenLDAP server in an iocage jail on your FreeNAS
    appliance.
date: 2019-06-20
categories:
  - tools
tags:
  - OpenLDAP
  - iocage
  - jails
  - certificates
---

An LDAP directory provides a central location to store information about the
users and computers in your organization. Business applications can use this
information to support services such as file sharing, collaboration and chat.

In this tutorial, learn how to install [OpenLDAP][0]—an open source LDAP
server—in a jail on your FreeNAS appliance. This tutorial shows how to configure
transport layer security (TLS) using a [Let's Encrypt][1] certificate.

## Prerequisites

* DNS entries
* Let's Encrypt certificates
* FreeNAS appliance

## Create the iocage jail

## Prepare the jail

* Install packages
* Copy certificates
* Register certificates

## Loading the configuration database

1. Stop the server
1. Delete current configuration database
   ```
   rm -rf /usr/local/etc/openldap/slapd.d
   mkdir /usr/local/etc/openldap/slapd.d
   ```
1. Delete current data
   ```
   rm -rf /usr/local/etc/openldap/data
   mkdir /usr/local/etc/openldap/data
   ```

1. Load slapd.ldif
   ```
   /usr/local/sbin/slapadd -n0 -F /usr/local/etc/openldap/slapd.d/ -l /root/openldap-config/slapd.ldif
   ```

Start server
```
/usr/local/libexec/slapd -F /usr/local/etc/openldap/slapd.d/
```

1. Load initial objects
   ```
   ldapadd -W -D "cn=admin,dc=home,dc=ricalo,dc=com" -f /root/openldap-config/domain.ldif
   ```

Test TLS configuration
```
ldapwhoami -H ldap://ldap.home.ricalo.com -x -ZZ
ldapwhoami -H ldap://ldap.home.ricalo.com -x -ZZ -D "uid=chino@ricalo.com,ou=users,dc=home,dc=ricalo,dc=com" -W
```

## Loading sample data


[0]: https://www.openldap.org/
[1]: https://letsencrypt.org/
