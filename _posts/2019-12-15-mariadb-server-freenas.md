---
title: Installing a MariaDB server on FreeNAS
excerpt: >
    Learn how to install a MariaDB server on FreeNAS. Configure ZFS datasets to
    store the data and log files and optimize the throuput of your databases.
    You can configure the server to accept connections locally or from remote
    hosts over the network.
date: 2019-12-17
categories:
tags:
  - mariadb
  - freenas
  - jails
  - zfs
toc: true
---


## Creating ZFS datasets

You can store the database files in specific ZFS datasets on your FreeNAS
server, which can provide some performance benefits. For example, you can create
a dataset with a record size of 16 kilobytes, which matches the [default page
size][5]{: target="external"} used in MariaDB.

In this guide, you create datasets for the corresponding `datadir`,
`innodb_data_home_dir`, and `innodb_log_group_home_dir` properties of MariaDB.
To create the datasets, run the following commands in a [FreeNAS
shell][0]{: target="external"}:

1. Create a dataset named `innodb_data` in the `tank` pool:
   ```shell
   zfs create tank/innodb_data
   ```
1. Set the following properties on the `innodb_data` dataset:
   * atime: `off`
   * compression: `off`
   * primarycache: `metadata`
   * recordsize: `16K`
   ```shell
   zfs set atime=off tank/innodb_data
   zfs set compression=off tank/innodb_data
   zfs set primarycache=metadata tank/innodb_data
   zfs set recordsize=16K tank/innodb_data
   ```
1. Create a dataset named `innodb_log` in the `tank` pool:
   ```shell
   zfs create tank/innodb_log
   ```
1. Set the following properties on the `innodb_log` dataset:
   * atime: `off`
   * compression: `off`
   * primarycache: `metadata`
   ```shell
   zfs set atime=off tank/innodb_log
   zfs set compression=off tank/innodb_log
   zfs set primarycache=metadata tank/innodb_log
   ```
1. Create a dataset named `datadir` in the `tank` pool:
   ```shell
   zfs create tank/datadir
   ```
1. Set the following properties on the `datadir` dataset:
   * atime: `off`
   * compression: `off`
   * primarycache: `metadata`
   * recordsize: `16K`
   ```shell
   zfs set atime=off tank/datadir
   zfs set compression=off tank/datadir
   zfs set primarycache=metadata tank/datadir
   zfs set recordsize=16K tank/datadir
   ```

Note that these are the settings that we recommend for good balance of
performance improvement and minimize the risk of data corruption. You should
evaluate the right settings for your workloads. To further improve performance,
consider [moving the ZFS Intent Log (ZIL) to a fast device](#move-zil), such as
a low-latency SSD.

{% assign jail-name = "mariadb" %}
{% include devpromedia/create-jail.md
   jail-name=jail-name
   packages="mariadb104-server" %}

Create folders in the jail where you are going to mount the datasets. Assign the
`mysql` user as the owner:
```shell
mkdir -p /var/db/mysql/innodb_data
mkdir -p /var/db/mysql/innodb_log
mkdir -p /var/db/mysql/datadir

chown -R mysql:mysql /var/db/mysql/innodb_data
chown -R mysql:mysql /var/db/mysql/innodb_log
chown -R mysql:mysql /var/db/mysql/datadir
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
1. Mount the `data` and `log` datasets on the corresponding folders in the jail:
   ```shell
   iocage fstab {{ jail-name }} --add /tank/innodb_data /var/db/mysql/innodb_data nullfs rw 0 0
   iocage fstab {{ jail-name }} --add /tank/innodb_log  /var/db/mysql/innodb_log  nullfs rw 0 0
   iocage fstab {{ jail-name }} --add /tank/datadir  /var/db/mysql/datadir  nullfs rw 0 0
   ```
1. Restart the jail:
   ```shell
   iocage start {{ jail-name }}
   ```


## Configure the service

Open a session on the jail:
```shell
iocage console {{ jail-name }}
```

Create the `/var/db/mysql/my.cnf` file with the following contents:
```yaml
[mysqld]
# Uncomment the following line to enable access from remote hosts.
# bind-address    = 0.0.0.0
innodb_data_home_dir      = /var/db/mysql/innodb_data
innodb_log_group_home_dir = /var/db/mysql/innodb_log
datadir                   = /var/db/mysql/datadir
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
