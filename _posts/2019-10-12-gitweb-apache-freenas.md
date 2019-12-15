---
title: Installing GitWeb on FreeNAS
excerpt: >
    Learn how to install GitWeb to browse the repositories on your server. Build
    the Git package from the Ports collection to install GitWeb on an Apache
    server.
date: 2019-10-12
categories:
tags:
  - git
  - gitweb
  - freenas
  - jails
  - apache
  - ports
  - openldap
  - letsencrypt
toc: true
---

Learn how to install [GitWeb][2]{: target="external"} to browse the repositories
on your server. Build the Git package from the Ports collection to install
GitWeb on an Apache server. Optionally, configure access over HTTPS and LDAP
authentication.

The following screenshot shows the GitWeb interface displaying two repositories:

![GitWeb screenshot][screenshot]

{% include devpromedia/create-jail.md
   jail-name="gitweb"
   packages="apache24 portmaster" %}

## Installing GitWeb from Ports

GitWeb is not included by default in the Git package. To get the GitWeb
binaries, you need to install Git from the Ports collection.

1. Download and extract the Ports Collection:
   ```shell
   portsnap fetch extract update
   ```
1. Build and install the Git package the GitWeb binaries:
   ```shell
   portmaster -m GITWEB="on" /usr/ports/devel/git
   ```
   Make shows the configuration prompts for the Git package and its
   dependencies, but the GitWeb option is preselected. Accepting the default
   options work well for most installations.

## Configuring the web service

1. Copy the example GitWeb directory that is included with the Git package:
   ```shell
   cp -R /usr/local/share/examples/git/gitweb /usr/local/www/gitweb
   ```
1. In the `/usr/local/etc/apache24/httpd.conf` file:
   * Uncomment the `LoadModule` directives to enable the CGI modules by
     removing the leading **#** character:
     ```xml
     <IfModule !mpm_prefork_module>
         LoadModule cgid_module libexec/apache24/mod_cgid.so
     </IfModule>
     <IfModule mpm_prefork_module>
         LoadModule cgi_module libexec/apache24/mod_cgi.so
     </IfModule>
     ```
   * Replace the following `DocumentRoot` and `Directory` entries:
     ```xml
     DocumentRoot "/usr/local/www/apache24/data"
     <Directory "/usr/local/www/apache24/data">
         ...
     </Directory>
     ```
     with the entries of your GitWeb directory:
     ```xml
     DocumentRoot "/usr/local/www/gitweb"
     <Directory "/usr/local/www/gitweb">
         AuthType None
         Options +FollowSymLinks +ExecCGI
         DirectoryIndex gitweb.cgi
         AddHandler cgi-script .cgi .pl
         Satisfy Any
         Allow from all
     </Directory>
     ```
1. Configure service start:
   ```shell
   sysrc apache24_enable="yes"
   ```
1. Restart the web server:
   ```shell
   service apache24 onerestart
   ```

## Create the common root directory

The repositories must be stored in a common root directory, which by default is
located in the `/pub/git` directory. To create the `/pub/git` directory:
```shell
mkdir -p /pub/git
```

If you followed our post about installing a Git server on FreeNAS and chose the
option to [store the repositories on a ZFS dataset][1], then you can just mount
the dataset on `/pub/git` by running the following command from a
[FreeNAS shell][0]{: target="external"}.

```shell
iocage fstab gitweb --add /mnt/tank/repos /pub/git nullfs ro 0 0
```

Otherwise, create a repository to test the installation:

1. Create a repository:
   ```shell
   mkdir -p /pub/git/repo1
   cd /pub/git/repo1
   git init
   ```
1. Create a commit in the repository:
   ```shell
   touch file1
   git add file1
   git commit -m "Adds file1 to repo"
   ```

Open a browser and go to [http://192.168.1.123][5]{: target="external"} to check
the GitWeb interface.

## (Optional) Configure access over HTTPS

To configure access over HTTPS, you need an SSL certificate, such as the ones
provided by [Let's Encrypt][6]{: target="external"}. Copy the `crt` and `key`
files of your certificate to a folder in the jail.

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
     DocumentRoot "/usr/local/www/gitweb"
     ```
   * Replace the ServerName declaration. Replace the following line:
     ```
     ServerName www.example.org:443
     ```
     with the server name that matches your host and the server on the
     certificate:
     ```
     ServerName gitweb.example.org:443
     ```
   * Update the `SSLCertificateFile` and `SSLCertificateKeyFile` entries with
     the path of the `crt` and `key` files of your certificate:
     ```
     SSLCertificateFile "path_to_your_certificate.crt" 
     SSLCertificateKeyFile "path_to_your_certificate.key"
     ```
1. Restart the web server:
   ```shell
   service apache24 onerestart
   ```

## (Optional) Configure LDAP authentication

You can configure Apache to validate the users against an LDAP directory, such
as OpenLDAP. If you configure authentication using this method, make sure to
[enable HTTPS](#optional-configure-access-over-https) because the credentials
are transmitted over the network in plain text.

This section assumes you already have a working LDAP server. For more
information, check the [Installing an OpenLDAP server][7] tutorial.

Apache doesn't include the modules required for LDAP authentication by default.
To get the modules, you need to install Apache from the Ports collection. To
install Apache from the Ports collection:

1. Remove the apache24 package:
   ```
   pkg delete apache24
   ```
1. Install the apr1 package. Make sure to select the **LDAP** option from the
   first dialog:
   ```
   portmaster /usr/ports/devel/apr1
   ```
1. Install the apache24 package. Make sure to select the **AUTHNZ_LDAP** and
   **LDAP** options from the first dialog:
   ```
   portmaster /usr/ports/www/apache24
   ```

In `/usr/local/etc/apache24/httpd.conf`, configure the web server to use the
LDAP modules and provide your LDAP server parameters:

1. To enable the LDAP modules, add the following `LoadModule` directives:
   ```
   LoadModule ldap_module libexec/apache24/mod_ldap.so
   LoadModule authnz_ldap_module libexec/apache24/mod_authnz_ldap.so
   ```
1. Replace the `Directory` element added in the [Configuring the web
   service](#configuring-the-web-service) section with the following:
   ```xml
   <Directory "/usr/local/www/gitweb">
     AuthType BasicAuth
     Name "LDAP authentication required"
     AuthBasicProvider ldap
     AuthLDAPURL ldap://ldapserver.example.org/ou=users,dc=example,dc=org?uid STARTTLS
     AuthLDAPBindDN uid=gitweb_service@example.org,ou=services,dc=example,dc=org
     AuthLDAPBindPassword gitweb_service_password
     Options +FollowSymLinks +ExecCGIDirectoryIndex gitweb.cgi
     AddHandler cgi-script .cgi .pl
     Require valid-user
   </Directory>
   ```
1. Restart the web server:
   ```
   service apache24 onerestart
   ```

   Let's review the new parameters in this configuration:
   * `AuthType`: Selects that method that is used to authenticate the user.
     The`BasicAuth` method sends the credentials through the network in plain
     text. This is the reason why you must configure HTTPS before enabling LDAP
     authentication.
   * `AuthBasicProvider`: Selects the provider to use with basic authentication.
     In this case, LDAP.
   * `AuthLDAPURL`: This parameter configures several options:
     * Protocol: Depending on your LDAP service, you can select `ldap` or
     `ldaps`.
     * LDAP URL: The URL of your LDAP server. In the example,
       `ldapserver.example.org`.
     * Container where to look for users. In this case,
       `ou=users,dc=example,dc=org`.
     * Attribute to match with the username entered in the login field. In this
       case, `uid`. It's advised to use a field that is unique in the LDAP
       directory, using an attribute that can be shared in multiple accounts,
       such as `cn` can lead to authentication issues.
     * Connection type. In this example, `STARTTLS`, which establishes a secure
       connection on the default LDAP port (389). You can establish an unsecure
       connection on the default LDAP port using the `NONE` option.
   * `AuthLDAPBindDN`: The account used to bind to and query the LDAP directory.
   * `AuthLDAPBindPassword`: The password of the account used to bind to the
      LDAP directory.
   * `Require`: Provides the authorization part of the process. In this case,
     `valid-user` accepts any user in the LDAP directory.


[screenshot]: /assets/images/gitweb-screenshot.png
[0]: https://www.ixsystems.com/documentation/freenas/11.2-U4.1/shell.html
[1]: /git-server-freenas/#optional-storing-the-repositories-on-a-zfs-dataset
[2]: https://git-scm.com/docs/gitweb
[4]: /git-server-freenas/
[5]: http://192.168.1.123
[6]: https://letsencrypt.org/
[7]: /ldap-server-freenas/
