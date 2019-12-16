---
title: Installing a MariaDB server on FreeNAS
excerpt: >
    Learn how to install a MariaDB server on FreeNAS. Configure ZFS datasets to
    store the data and log files and optimize the throuput of your databases.
date: 2019-12-17
categories:
tags:
  - mariadb
  - freenas
  - jails
  - zfs
toc: true
---

Learn how to install a MariaDB server on FreeNAS. Configure ZFS datasets to
store the data and log files and optimize the throuput of your databases. You
can configure the server to accept connections locally or from remote hosts over
the network.


## Creating ZFS datasets

You can store the data and log files in specific ZFS datasets on your FreeNAS
server, which can provide some performance benefits. For example, you can create
a dataset with a record size of 16 kilobytes, which matches the [default page
size][5]{: target="external"} used in MariaDB.

To create the datasets, run the following commands in a [FreeNAS
shell][0]{: target="external"}:

1. Create a dataset named `data` in the `tank` pool:
   ```shell
   zfs create tank/data
   ```
1. Set the following properties on the data dataset:
   * atime: `off`
   * compression: `off`
   * primarycache: `metadata`
   * recordsize: `16K`
   ```shell
   zfs set atime=off tank/data
   zfs set compression=off tank/data
   zfs set primarycache=metadata tank/data
   zfs set recordsize=16K tank/data
   ```
1. Create a dataset named `logs` in the `tank` pool:
   ```shell
   zfs create tank/log
   ```
1. Set the following properties on the data dataset:
   * atime: `off`
   * compression: `off`
   * primarycache: `metadata`
   ```shell
   zfs set atime=off tank/log
   zfs set compression=off tank/log
   zfs set primarycache=metadata tank/log
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

Create folders in the jail where you are going to mount the data and log
datasets. Assign the `mysql` user as the owner:
```shell
mkdir -p /var/db/mysql/engine/data
mkdir -p /var/db/mysql/engine/log
chown -R mysql:mysql /var/db/mysql/engine
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
1. Mount the `repos` dataset on the `repos` folder created in the previous
   section:
   ```shell
   iocage fstab {{ jail-name }} --add /tank/data /var/db/mysql/engine/data nullfs rw 0 0
   iocage fstab {{ jail-name }} --add /tank/log  /var/db/mysql/engine/log  nullfs rw 0 0
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
