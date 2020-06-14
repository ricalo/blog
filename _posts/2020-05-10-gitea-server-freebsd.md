---
title: Installing a Gitea server on FreeBSD
excerpt: >
    Learn how to install a Gitea server in a jail on a FreeBSD host. Configure
    the Gitea service to work without a reverse proxy.
date: 2020-06-12
categories:
tags:
  - gitea
  - freebsd
  - jails
  - zfs
toc: true
---

## Prerequisites

* A MariaDB server, which can be the same as your Gitea server or a separate
  host.
* To configure access over HTTPS, you need an SSL certificate, such as the ones
  provided by [Let's Encrypt](https://letsencrypt.org/){: target="external"}.


## Creating ZFS datasets

You can store the repositories in a specific ZFS dataset on your FreeBSD server,
which can make it easier to perform some administrative tasks, such as backups.

To create the dataset, run the following command in the FreeBSD host:
```shell
zfs create tank/repositories
```
The previous command creates a dataset named `repositories` in the `tank` pool.

{% assign jail-name = "gitea" %}
{% include devpromedia/create-jail.md
   jail-name=jail-name
   packages="ca_root_nss git openssl gitea" %}


Create the folder in the jail where you are going to mount the dataset. Assign
the `git` user as the owner:
```shell
mkdir -p /mnt/repositories
chown -R git:git /mnt/repositories
```

Close the session in the jail so you can mount the datasets from your FreeBSD
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
   iocage fstab {{ jail-name }} --add /tank/repositories /mnt/repositories nullfs rw 0 0
   ```
1. Restart the jail:
   ```shell
   iocage start {{ jail-name }}
   ```


## Creating the database

Gitea requires a database server, you can use MySQL / MariaDB, PostgreSQL,
MSSQL, or SQLite. This guide uses a MariaDB server to host the database. For
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
1. Create a database user named `gitea`. Make sure to replace `localhost`
   with the hostname of the Gitea server and `db-user-password` with a
   strong password:
   ```sql
   CREATE USER 'gitea'@'localhost' IDENTIFIED BY 'db-user-password';
   ```
   If your MariaDB server is [configured with access over TLS][8] you can append
   `REQUIRE SSL` to the previous command to make sure that the communication
   between the servers is encrypted.
1. Create the database:
   ```sql
   CREATE DATABASE gitea;
   ```
1. Grant permissions to the user on the database:
   ```sql
   GRANT ALL ON gitea.* TO 'gitea'@'localhost';
   FLUSH PRIVILEGES;
   ```
   Make sure to replace `localhost` with the hostname of the Gitea server.
1. Close the database connection:
   ```sql
   exit
   ```


## Allow the git user to bind to the HTTPS port

Since the Gitea service runs under the git user, it must be allowed to bind to
the HTTPS port. This allows Gitea to run without a reverse proxy. To allow the
user to bind to the HTTPS port:

* Configure a port ACL rule on the host.
* Set the `securelevel` property of the jail to `-1`.
* Configure the reserved ports on the jail.

To configure the port ACL rule on the host:

1. Enable the mac_portacl module by adding the following line to
   `/boot/loader.conf`:
   ```ini
   mac_portacl_load="YES"
   ```
1. Get the id of the jail's git user:
   ```shell
   iocage exec {{ jail-name }} -- id -u git
   ```
1. Configure the rule by adding the following line to `/etc/sysctl.conf`:
   ```ini
   security.mac.portacl.rules=uid:git-user-id:tcp:443
   ```
   Replace `git-user-id` with the value from the previous step.
1. Restart the host.

To set the security level on the jail to `-1`, run the following command from
the FreeBSD host:
```
iocage set securelevel=-1 {{ jail-name }}
```

To configure the reserved ports on the jail:

1. Configure the `reservedhigh` property to a value lower than `443` by adding
   the following line to `/etc/sysctl.conf`:
   ```ini
   net.inet.ip.portrange.reservedhigh=442
   ```
1. Restart the jail.


## Configure the service

Open a session on the {{ jail-name }} jail:
```shell
iocage console {{ jail-name }}
```

1. Copy the `crt` and `key` files of your certificate to
   `/usr/local/etc/gitea/cert` in the jail.
1. Generate a 64 byte pseudo-random string to use as the `INTERNAL_TOKEN` value
   in the service configuration:
   ```
   openssl rand -base64 64
   ```
1. Generate a 32 byte pseudo-random string to use as the `JWT_SECRET` value in
   the service configuration:
   ```
   openssl rand -base64 32
   ```
1. Configure Gitea by editing `/usr/local/etc/gitea/conf/app.ini`. Pay special
   attention to the following attributes:
   ```ini
   [database]
   DB_TYPE  = mysql
   HOST     = database_host:database_port # port usually is 3306
   NAME     = database_dbname
   PASSWD   = `database_password`
   USER     = database_user

   [repository]
   ROOT        = /mnt/repositories

   [server]
   DOMAIN           = your_domain
   HTTP_ADDR        = 0.0.0.0
   HTTP_PORT        = 443
   ROOT_URL         = https://%(DOMAIN)s/
   START_SSH_SERVER = true
   CERT_FILE        = cert/your_certificate.crt
   KEY_FILE         = cert/your_certificate.key
   PROTOCOL         = https

   [security]
   INSTALL_LOCK   = true
   INTERNAL_TOKEN = ABCDEF # 64-byte string from the previous step
   SECRET_KEY     = any_password_of_your_choosing

   [oauth2]
   JWT_SECRET = ABCDEF # 32-byte string from the previous step
   ```
1. Configure the gitea service startup and restart the service:
   ```shell
   sysrc gitea_enable=yes
   service gitea onestart
   ```

Browse to **https://your_domain** and you should see the Gitea welcome page.
