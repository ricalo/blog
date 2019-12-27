---
title: Installing a Nextcloud server on FreeNAS
excerpt: >
    Learn how to install a Nextcloud server on FreeNAS. Configure ZFS datasets
    to store the files of your users.
date: 2019-12-27
categories:
tags:
  - nextcloud
  - freenas
  - jails
  - zfs
toc: true
---

Use the values in this guide as an example to tune your system. You might need
to do some research to find your optimal configuration.
{: .notice--warning }

## Prerequisites

* A MariaDB server, which can be the same as your Nextcloud server or a separate
  host.
* To configure access over HTTPS, you need an SSL certificate, such as the ones
  provided by [Let's Encrypt](https://letsencrypt.org/){: target="external"}.


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
   packages="apache24 nextcloud-php73 php73-pecl-imagick" %}


Create the folder in the jail where you are going to mount the dataset. Assign
the `www` user as the owner:
```shell
mkdir -p /usr/local/www/files
chown -R www:www /usr/local/www/files
chown -R www:www /usr/local/www/nextcloud
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
1. Create a database user named `nextcloud`. Make sure to replace `localhost`
   with the hostname of the Nextcloud server and `db-user-password` with a
   strong password:
   ```sql
   CREATE USER 'nextcloud'@'localhost' IDENTIFIED BY 'db-user-password';
   ```
   If your MariaDB server is [configured with access over TLS][8] you can append
   `REQUIRE SSL` to the previous command to make sure that the communication
   between the servers is encrypted.
1. Create the database:
   ```sql
   CREATE DATABASE nextcloud;
   ```
1. Grant permissions on the database to the nextcloud user:
   ```sql
   GRANT ALL ON nextcloud.* TO 'nextcloud'@'localhost';
   FLUSH PRIVILEGES;
   ```
   Make sure to replace `localhost` with the hostname of the Nextcloud server.
1. Close the database connection:
   ```sql
   exit
   ```

## Configure the service

Open a session on the {{ jail-name }} jail:
```shell
iocage console {{ jail-name }}
```


1. Copy the `php.ini-production` example file to the `php.ini` file:
   ```shell
   cp /usr/local/etc/php.ini-production /usr/local/etc/php.ini
   ```
1. Copy the `ca`, `crt`, and `key` files of your certificate to a folder in the
   jail.
1. By default, any `.conf` file in the `Includes` folder is added to the Apache
   configuration. Create the `/usr/local/etc/apache24/Includes/my-conf.conf`
   with the following contents:
   ```xml
   LoadModule proxy_module libexec/apache24/mod_proxy.so
   LoadModule proxy_fcgi_module libexec/apache24/mod_proxy_fcgi.so
   LoadModule rewrite_module libexec/apache24/mod_rewrite.so
   LoadModule socache_shmcb_module libexec/apache24/mod_socache_shmcb.so
   LoadModule ssl_module libexec/apache24/mod_ssl.so
   DocumentRoot "/usr/local/www/nextcloud"
   <Directory "/usr/local/www/nextcloud">
       Options Indexes FollowSymLinks
       AllowOverride all
       Require all granted
   </Directory>
   Listen 443
   <VirtualHost *:80>
     ServerName nextcloud.example.org
     Redirect permanent / https://nextcloud.example.org/
   </VirtualHost>
   <VirtualHost *:443>
     DirectoryIndex index.php
     DocumentRoot "/usr/local/www/nextcloud"
     Protocols h2 http/1.1
     ServerName nextcloud.example.org:443
     SSLEngine on
     SSLCertificateFile      /path/to/certificate.crt
     SSLCertificateKeyFile   /path/to/certificate.key
     SSLCertificateChainFile /path/to/certificate.ca
     <FilesMatch "\.php$">
       SetHandler "proxy:fcgi://127.0.0.1:9000/"
     </FilesMatch>
     <IfModule mod_headers.c>
       Header always set Strict-Transport-Security "max-age=15552000; includeSubDomains"
     </IfModule>
   </VirtualHost>
   ```
1. Configure the web service startup and restart the web server:
   ```shell
   sysrc apache24_enable=yes
   sysrc php_fpm_enable=yes
   service php-fpm onestart
   service apache24 onestart
   ```


## Run the installation wizard

This is a good time to run the installation wizard since you already configured
access over HTTPS. Running the installation wizard includes providing a
password for an admin account, which is transmitted over the network.

To run the wizard, open a browser and go to your server's URL, for example
`https://nextcloud.example.org`. The following screenshot shows the installation
wizard:

![nextcloud-setup][screenshot-wizard]

Provide the following information to the wizard and click __Finish setup__.

* Username: The username of the new admin account
* Password: The password of the new admin account
* Data folder: `/usr/local/www/files`
* Database user: `nextcloud`
* Database password: `db-user-password`
* Database name: `nextcloud`
* Database host: The MariaDB server, including the port. For example:
  `localhost:3306`.

After a few seconds, your browser displays the welcome screen.
![nextcloud-welcome][screenshot-welcome]

At this time, you should go to __Settings__ > __Administration__ > __Overview__
to update Nextcloud to the latest version. You might see some warnings about
further configuration of your server. We cover some of them in the following
section.

## Further improvements

In this section, we show how to implement some of the improvements described in
the [Hardening and security guidance][2]{: target="external"} and
[Server tuning][3]{: target="external"} sections of the Nextcloud documentation.

### Increase PHP memory limit

You might see the following warning in the __Settings__ > __Administration__ >
__Overview__ page of your server:

> The PHP memory limit is below the recommended value of 512MB.

To increase the memory limit:

1. Update the following line in the `/usr/local/etc/php.ini` file:
   ```
   memory_limit = 512M
   ```
1. Restart the php-fpm service:
   ```shell
   service php-fpm restart
   ```

After restarting the php-fpm service, the message should disappear from the
__Overview__ page.

### Give PHP read access to /dev/urandom

Nextcloud uses _urandom_ along with other sources to generate random numbers. To
provide access to _urandom_:

1. Update the following line in the `/usr/local/etc/php.ini` file:
   ```
   open_basedir = /dev/urandom:/usr/local/www/nextcloud/:/usr/local/www/files/:/var/log/nextcloud/:/tmp/
   ```
1. Restart the php-fpm and web service:
   ```shell
   service php-fpm restart
   service apache24 restart
   ```

### Tune PHP-FPM

Per Nextcloud's documentation:

> Each simultaneous request of an element is handled by a separate PHP-FPM
> process. So even on a small installation you should allow more processes to
> run.

1. To tune php-fpm, update the following settings in
   `/usr/local/etc/php-fpm.d/www.conf`:
   ```
   pm = dynamic
   pm.max_children = 120
   pm.start_servers = 12
   pm.min_spare_servers = 6
   pm.max_spare_servers = 18
   ```
1. Restart the php-fpm and web service:
   ```shell
   service php-fpm restart
   service apache24 restart
   ```

Use the values in the previous example as a starting point to tune your system.

### Enable PHP OPcache

The OPcache improves the performance of PHP applications by caching precompiled
bytecode. To enable the OPcache:

1. Update the following settings in `/usr/local/etc/php.ini`:
   ```
   opcache.enable=1
   opcache.interned_strings_buffer=8
   opcache.max_accelerated_files=10000
   opcache.memory_consumption=128
   opcache.save_comments=1
   opcache.revalidate_freq=1
   ```
1. Restart the php-fpm and web service:
   ```shell
   service php-fpm restart
   service apache24 restart
   ```

### Using cron to perform background jobs

Nextcloud requires to run background jobs on a regular basis. The jobs can run
using AJAX, Webcron, or cron. The recommended method is cron. To configure cron
to run the background jobs:

1. Run the following command to edit the cron jobs for the __www__ user:
   ```
   crontab -u www -e
   ```
1. In the editor, enter the following line:
   ```
   */5  *  *  *  * /usr/local/bin/php -f /var/www/nextcloud/cron.php
   ```


[screenshot-wizard]: /assets/images/nextcloud-wizard.png
[screenshot-welcome]: /assets/images/nextcloud-welcome.png
[0]: https://www.ixsystems.com/documentation/freenas/11.2-U4.1/shell.html
[5]: https://mariadb.com/kb/en/library/innodb-system-variables/#innodb_page_size
[6]: https://letsencrypt.org/
[7]: https://blog.pcisecuritystandards.org/resource-guide-migrating-from-ssl-and-early-tls
[1]: /mariadb-server-freenas/
[2]: https://docs.nextcloud.com/server/17/admin_manual/installation/harden_server.html
[3]: https://docs.nextcloud.com/server/17/admin_manual/installation/server_tuning.html
[4]: https://cwiki.apache.org/confluence/display/httpd/php
[8]: /mariadb-server-freenas/#enable-tls
