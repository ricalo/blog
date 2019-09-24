---
title: Installing a Minecraft server on FreeNAS
excerpt: >
    Learn how to install a lightweight Git server on your FreeNAS server.  Host
    the repositories that contain critical business data on your own
    infrastructure.
date: 2019-09-24
categories:
tags:
  - minecraft
  - freenas
  - jails
  - service
  - daemon
toc: true
---

Learn how to install a lightweight Git server on your FreeNAS server. Host the
repositories that contain critical business data on your own infrastructure.

A centralized Git server provides multiple benefits, which we explain in the
[version control system][2] section of our architecture for a self-hosted
business guide.

## Preparing the jail

The instructions in this post host the Minecraft server in a jail on the FreeNAS
server. To learn more about why we use jails to host the applications, check the
[Application server][3] section of our self-hosted architecture post.

In this section, you'll perform the following tasks:

* Create a jail.
* Configure networking on the jail.
* Install the prerequisite packages.

Run the commands from a session in your FreeNAS server. You can use the
[FreeNAS shell][0]{: target="external"} for this purpose.

To create a jail:

1. Fetch or update your version of FreeBSD for jail usage:
   ```shell
   iocage fetch --release 11.2-RELEASE
   ```
   You can check your current release with the `freebsd-version` command.
1. Create a jail named `minecraft`:
   ```shell
   iocage create --name minecraft --release 11.2-RELEASE
   ```

To configure networking on the jail:

1. Configure the IP address. The following example sets the IP address to
   `192.168.1.123` using a subnet mask of `255.255.255.0` on the `re0`
   interface. The command uses the [CIDR notation][10]{: target="external"},
   which translates to `192.168.1.123/24`:
   ```shell
   iocage set ip4_addr="re0|192.168.1.123/24" minecraft
   ```
1. Configure the default router. The following example sets the default router
   to `192.168.1.1`:
   ```shell
   iocage set defaultrouter=192.168.1.1 minecraft
   ```

Start the jail and open a session to complete the rest of the tasks in this
section:

```shell
iocage console minecraft
```

The instructions require the curl package to download the jar file that includes
the binaries of the Minecraft server. To install the curl package:

1. Update the packages in the jail:
   ```shell
   pkg update
   ```
1. Install the package:
   ```shell
   pkg install --yes curl
   ```

## Accepting the EULA

Git clients use an SSH connection to communicate with the server. This section
shows how to configure the SSH service to accept connections from clients.

On your client computer, verify that you have a certificate that you can use to
communicate with the server, or create a new one:

1. Check your existing SSH keys:
   ```shell
   ls -la ~/.ssh
   ```
   Usually, you can use the key stored in the `id_rsa.pub` file. If the previous
   command lists the `id_rsa.pub` file go to step three. Otherwise continue to
   step two.
1. Generate an SSH key. Replace *email@example.org* with your email address.
   ```shell
   ssh-keygen -t rsa -b 4096 -C "email@example.org"
   ```
   The previous command prompts for the location to which to save the key.
   Accept the default value of `id_rsa`.
1. Copy the SSH public key as an authorized key:
   ```shell
   ssh-copy-id -i ~/.ssh/id_rsa.pub git@gitserver
   ```

## Running the server for the first time

To configure the SSH service on the jail, run the following commands in a
session on the FreeNAS server:

1. If you’re not already in a session on the jail, open one:
   ```shell
   iocage console gitserver
   ```
1. Configure the SSH service to start on boot:
   ```shell
   sysrc sshd_enable="yes"
   ```
1. Start (or restart) the SSH service:
   ```shell
   service sshd onerestart
   ```

To validate the configuration, connect to the Git server by running the
following ssh command from your client computer:
```shell
ssh git@gitserver
```
Verify that you can connect to the server _without_ using a password.

## Connecting from a client

To test the Git server, create an empty repository that clients can use to push
commits:

1. Open an SSH session to the server:
   ```shell
   ssh git@gitserver
   ```
1. Create a new `my-repo.git` folder in `/git/repos/`:
   ```shell
   mkdir /git/repos/my-repo.git
   cd /git/repos/my-repo.git
   ```
   The `.git` extension is a convention for the _bare repository_ used in the
   next step. The extension is not strictly required, but recommended.
1. Initialize the repository as a bare repository. A bare repository doesn't
   contain a working directory which is not required in the server because files
   are not edited directly.
   ```shell
   git init --bare
   ```

On the client computer, create a repository and push a commit to the server:

1. Create and initialize a repository:
   ```shell
   mkdir my-repo
   cd my-repo
   git init
   ```
1. Add a test file and commit the changes to the repository:
   ```shell
   echo "A test file" >> test-file
   git add test-file
   git commit -m "A test commit"
   ```
1. Register the remote repository, which is hosted on the Git server:
   ```shell
   git remote add origin git@gitserver:repos/my-repo.git
   ```
1. Push the new commit to the Git server:
   ```shell
   git push origin master
   ```
   The previous command pushes the commit to the server.

You can clone, pull, and push additional commits from clients that have access
to the server.

## Configuring the server as a daemon

[0]: https://www.ixsystems.com/documentation/freenas/11.2-U4.1/shell.html
[1]: https://iocage.readthedocs.io/en/latest/
[2]: /self-hosted-architecture/#version-control-system
[3]: /self-hosted-architecture/#application-server
[10]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
