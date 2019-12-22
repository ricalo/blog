---
title: Installing a Nextcloud server on FreeNAS
excerpt: >
    Learn how to install a Nextcloud server on FreeNAS. Configure ZFS datasets
    to store the files of your users.
date: 2019-12-21
categories:
tags:
  - nextcloud
  - freenas
  - jails
  - zfs
toc: true
---


## Creating ZFS datasets

You can store the files that users upload to Nextcloud in a specific ZFS dataset
on your FreeNAS server, which can make it easier to perform some administrative
tasks, such as backups.

To create the dataset, run the following command in a [FreeNAS
shell][0]{: target="external"}:
```shell
zfs create tank/files
```
The previous command creates a dataset named `files` in the `tank` pool.

{% assign jail-name = "nextcloud" %}
{% include devpromedia/create-jail.md
   jail-name=jail-name
   packages="apache24 nextcloud-php73 mod_php73" %}


Create the folder in the jail where you are going to mount the dataset. Assign
the `www` user as the owner:
```shell
mkdir -p /usr/local/www/files
chown -R www:www /usr/local/www/files
```

Close the session in the jail so you can mount the datasets from your FreeNAS
session:
```shell
exit
```

Mount the datasets on the jail:

1. Use the following command to stop the jail:
   ```shell
   iocage stop {{ jail-name }}
   ```
1. Mount the `files` dataset on the folder created in the previous section:
   ```shell
   iocage fstab {{ jail-name }} --add /tank/files /usr/local/www/files nullfs rw 0 0
   ```
1. Restart the jail:
   ```shell
   iocage start {{ jail-name }}
   ```


## Creating the database

Nextcloud requires a database server, you can use MySQL / MariaDB, PostgreSQL,
or Oracle. This guide uses a MariaDB server to host the Nextcloud database. For
information on how to install a MariaDB server, check
[Installing a MariaDB server on FreeNAS][1].

Once you have the MariaDB server installed, create a user and database:

1. Open a session on your MariaDB server:
   ```shell
   iocage console mariadb-server
   ```
1. Once in the MariaDB server, open a connection to the database engine:
   ```shell
   mysql -u root -p
   ```
1. Create a database user named `nextcloud`. Make sure to replace
   `db-user-password` with a strong password:
   ```sql
   CREATE USER 'nextcloud' IDENTIFIED BY 'db-user-password';
   ```
1. Create the database:
   ```sql
   CREATE DATABASE nextcloud;
   ```
1. Grant permissions on the database to the nextcloud user:
   ```sql
   GRANT ALL ON nextcloud.* TO 'nextcloud';
   FLUSH PRIVILEGES;
   ```
1. Close the database connection:
   ```sql
   exit
   ```

## Configure the service

Open a session on the {{ jail-name }} jail:
```shell
iocage console {{ jail-name }}
```

Add the following to the `/usr/local/etc/apache24/httpd.conf` file:
```
<FilesMatch "\.php$">
    SetHandler application/x-httpd-php
</FilesMatch>
```

```shell
sysrc apache24_enable=yes
```

### Configure SSL

Copy the ca, crt and key files of your certificate to a folder in the jail.

Create a file `/usr/local/etc/apache24/Includes/nextcloud.conf`:

```xml
LoadModule ssl_module libexec/apache24/mod_ssl.so
LoadModule socache_shmcb_module libexec/apache24/mod_socache_shmcb.so
Listen 443
<VirtualHost *:443>
  DocumentRoot "/usr/local/www/nextcloud"
  ServerName nextcloud.example.org
  DirectoryIndex /index.php index.php
  SSLEngine on
  SSLCertificateFile      /path/to/certificate.crt
  SSLCertificateKeyFile   /path/to/certificate.key
  SSLCertificateChainFile /path/to/certificate.ca
</VirtualHost>
```

## Run the wizard

## Configure proxy_fcgi and php-fpm

https://cwiki.apache.org/confluence/display/httpd/php


## Follow Nextcloud guidance

The Nextcloud documentation provides great advice on further improving your
server installation. The following resources are particularly useful at this
point:

* [Hardening and security guidance][2]{: target="external"}
* [Server tuning][3]{: target="external"}



Create the `/var/db/mysql/my.cnf` file with the following contents:
```yaml
[mysqld]
# Uncomment the following line to enable access from remote hosts.
# bind-address    = 0.0.0.0
innodb_data_home_dir      = /var/db/mysql/engine/data
innodb_log_group_home_dir = /var/db/mysql/engine/log
```
Uncomment the `bind-address` option to enable access from other hosts in the
network. Otherwise, connections are only accepted from the jail. If you decide
to accept connections from other hosts, you should [configure access over
TLS](#enable-tls).


Configure the service startup and start the service:

```shell
sysrc mysql_enable=yes
service mysql-server start
```

Run the script to improve the security of the installation:
```
mysql_secure_installation
```


## Testing the installation

To test the installation, open a connection using the following command from
within the jail:
```shell
mysql --user=root --password
```
After entering the password of the root user, you should see a message similar
to the following:
```
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 12
Server version: 10.4.10-MariaDB FreeBSD Ports

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

From the MariaDB prompt, you can list the existing databases:
```sql
show databases;
```


## Configuring access over TLS
{: #enable-tls }

To configure access over TLS, you need an SSL certificate, such as the ones
provided by [Let’s Encrypt][6]{: target="external"}. Copy the `crt` and `key`
files of your certificate to a folder in the jail.

Then, configure MariaDB to use the certificate by adding the following entries
to the `[mysqld]` section of the `/var/db/mysql/my.cnf` file:

```yaml
[mysqld]
...
ssl_cert        = /path/to/certificate.crt
ssl_key         = /path/to/certificate.key
tls_version     = TLSv1.2,TLSv1.3
```

MariaDB provides support for TLS version 1.1 by default. However, it's
recommended to use TLS version 1.2 and above according to the [PCI Security
Standards Council][7]{: target="external"}. The `tls_version` option specified
in the previous example removes support for TLS version 1.1.


## Move the ZIL to a low-latency device
{: #move-zil }

For better write performance, consider moving the ZIL to a low-latency device,
such as an NVMe drive. If you have a pair of devices, you can use the following
command to add the devices to the **tank** pool as a mirrored log devices:

```shell
zpool add tank log mirror nvd0 nvd1
```

Where `nvd0` and `nvd1` are the low-latency devices.

If you only have one drive, you can add it as a log device with the following
command:
```shell
zpool add tank log nvd0
```

To confirm that the pool is using the devices, run `zpool status tank` and check
that the devices are listed in the logs section, as shown in the following
example:

```shell
$ zpool status tank
  pool: tank
 state: ONLINE
  scan: scrub repaired 0 in 0 days ...
config:

        NAME        STATE     READ WRITE CKSUM
        tank        ONLINE       0     0     0
          mirror-0  ONLINE       0     0     0
            ada0    ONLINE       0     0     0
            ada1    ONLINE       0     0     0
        logs
          mirror-1  ONLINE       0     0     0
            nvd0    ONLINE       0     0     0
            nvd1    ONLINE       0     0     0

errors: No known data errors
```


[0]: https://www.ixsystems.com/documentation/freenas/11.2-U4.1/shell.html
[5]: https://mariadb.com/kb/en/library/innodb-system-variables/#innodb_page_size
[6]: https://letsencrypt.org/
[7]: https://blog.pcisecuritystandards.org/resource-guide-migrating-from-ssl-and-early-tls
[1]: /mariadb-server-freenas/
[2]: https://docs.nextcloud.com/server/17/admin_manual/installation/harden_server.html
[3]: https://docs.nextcloud.com/server/17/admin_manual/installation/server_tuning.html
