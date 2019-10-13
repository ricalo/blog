---
title: Installing GitWeb on FreeNAS
excerpt: >
    Learn how to install the GitWeb interface to browse the repositories in your
    Git server. Build the Git package from the Ports Collection to install the
    GitWeb frontend.
date: 2019-10-12
categories:
tags:
  - git
  - gitweb
  - freenas
  - jails
  - apache
  - ports
toc: true
---

Learn how to install the [GitWeb][2]{: target="external"} interface to browse
the repositories in your Git server. Build the Git package from the Ports
Collection to install the GitWeb frontend.

The following screenshot shows the GitWeb interface displaying a couple
repositories:

![GitWeb screenshot][screenshot]

## Preparing the jail

The instructions in this post host GitWeb in a jail on the FreeNAS server. To
learn more about why we use jails to host the applications, check the
[Application server][3] section of our self-hosted architecture post.

In this section, you'll perform the following tasks:

* Create a jail.
* Configure networking on the jail.
* Install the prerequisite packages.

Run the commands from a session in your FreeNAS server. You can use the
[FreeNAS shell][0]{: target="external"} for this purpose.

To create a jail:

1. Fetch or update the release version of FreeBSD for jail usage:
   ```sh
   iocage fetch --release 11.2-RELEASE
   ```
   If the release version of FreeBSD on your system isn't `11.2`, you can check
   the current version with the `freebsd-version` command and use that instead.
1. Create a jail named `gitweb`:
   ```shell
   iocage create --name gitweb --release 11.2-RELEASE
   ```

To configure networking on the jail:

1. Configure the IP address. The following example sets the IP address to
   `192.168.1.123` using a subnet mask of `255.255.255.0` on the `re0`
   interface. The command uses the [CIDR notation][10]{: target="external"},
   which translates to `192.168.1.123/24`:
   ```shell
   iocage set ip4_addr="re0|192.168.1.123/24" gitweb
   ```
1. Configure the default router. The following example sets the default router
   to `192.168.1.1`:
   ```shell
   iocage set defaultrouter=192.168.1.1 gitweb
   ```

Start the jail and open a session to complete the rest of the tasks in this
section:

```shell
iocage console gitweb
```

GitWeb requires a web server to work. This post uses Apache to serve the GitWeb
content. To install the Apache package:

1. Update the packages in the jail:
   ```shell
   pkg update
   ```
1. Install the Git and Apache packages:
   ```shell
   pkg install --yes apache24
   ```

## Installing GitWeb from Ports

GitWeb is not included by default in the Git package. To get the GitWeb
binaries, you need to install Git from the Ports Collection.

1. Download and extract the Ports Collection:
   ```shell
   portsnap fetch extract update
   ```
1. Move to the directory of the Git package:
   ```shell
   cd /usr/ports/devel/git
   ```
1. Build and install the Git package the GitWeb binaries:
   ```shell
   make config-recursive install distclean GITWEB="on" BATCH="yes"
   ```

## Configuring the web service

1. Copy the example GitWeb directory that is included with the Git package:
   ```shell
   cp -R /usr/local/share/examples/git/gitweb /usr/local/www/gitweb
   ```
1. In the `/usr/local/etc/apache24/httpd.conf` file:
   1. Enable the CGI module by uncommenting the `LoadModule` directive in the
      following snippet:
      ```xml
      <IfModule mpm_prefork_module>
          #LoadModule cgi_module libexec/apache24/mod_cgi.so
      </IfModule>
      ```
   1. Replace the following `DocumentRoot` and `Directory` entries:
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

## Create the repository root

The repositories must be stored in a common repository root, which by default is
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

[screenshot]: /assets/images/gitweb-screenshot.png
[0]: https://www.ixsystems.com/documentation/freenas/11.2-U4.1/shell.html
[1]: /git-server-freenas/#optional-storing-the-repositories-on-a-zfs-dataset
[2]: https://git-scm.com/docs/gitweb
[3]: /self-hosted-architecture/#application-server
[4]: /git-server-freenas/
[5]: http://192.168.1.123
[10]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
