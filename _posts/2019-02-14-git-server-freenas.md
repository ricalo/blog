---
title: Installing a Git server on FreeNAS
excerpt: >
    Learn how to install a lightweight Git server on your FreeNAS server.  Host
    the repositories that contain critical business data on your own
    infrastructure.
date: 2019-09-20
categories:
tags:
  - git
  - freenas
  - iocage
  - jails
  - ssh
toc: true
---

Learn how to install a lightweight Git server on your FreeNAS server. Host the
repositories that contain critical business data on your own infrastructure.

A centralized Git server provides multiple benefits, which we explain in the
[version control system][2] section of our architecture for a self-hosted
business guide.

## Preparing the jail

This instructions in this post host the Git server in a jail on the FreeNAS
server. To learn more about why we use jails to host the applications, check the
[Application server][3] section of our self-hosted architecture post.

In this section, you'll perform the following tasks:

* Create a jail.
* Configure networking on the jail.
* Configure a user for Git operations.
* Install the Git package.

Run the commands from a session in your FreeNAS server. You can use the
[FreeNAS shell][0]{: target="external"} for this purpose.

To create a jail:

1. Fetch or update your version of FreeBSD for jail usage:
   ```shell
   iocage fetch --release 11.2-RELEASE
   ```
   You can check your current release with the `freebsd-version` command.
1. Create a jail named `gitserver`:
   ```shell
   iocage create --name gitserver --release 11.2-RELEASE
   ```

To configure networking on the jail:

1. Configure the IP address. The following example sets the IP address to
   `192.168.1.123` using a subnet mask of `255.255.255.0` on the `re0`
   interface. The command uses the [CIDR notation][10]{: target="external"},
   which translates to `192.168.1.123/24`:
   ```shell
   iocage set ip4_addr="re0|192.168.1.123/24" gitserver
   ```
1. Configure the default router. The following example sets the default router
   to `192.168.1.1`:
   ```shell
   iocage set defaultrouter=192.168.1.1 gitserver
   ```

Start the jail and open a session to complete the rest of the tasks in this
section:

```shell
iocage console gitserver
```

To configure a user for Git operations:

1. Create a `repos` folder to store the repositories:
   ```shell
   mkdir -p /git/repos
   ```
1. Create a new `git` user:
   ```shell
   pw useradd -n git -d /git
   ```
1. Transfer ownership of the `/git` folder, which includes the folder that
   stores the repositories, to the `git` user:
   ```shell
   chown -R git /git
   ```
1. Prompt for a new password for the `git` user:
   ```shell
   passwd git
   ```

The server requires the Git package to manage the repos. To install the Git
package:

1. Update the packages in the jail:
   ```shell
   pkg update
   ```
1. Install the package:
   ```shell
   pkg install --yes git
   ```

## (Optional) Storing the repositories on a ZFS dataset

You can store the repositories in a specific ZFS dataset on your FreeNAS server.
To learn about the benefits, check the [Application server][3] section of our
self-hosted architecture post.

To store the repos in a dataset, run the following commands in a [FreeNAS
shell][0]{: target="external"} or an SSH session on the FreeNAS server. Make
sure that you're not in the jail session from the previous section. If you are
in the jail session, use the `exit` command to go to the FreeNAS shell.

1. Create a dataset named `repos` in the `tank` pool:
   ```shell
   zfs create tank/repos
   ```
1. Use the following command to stop the jail:
   ```shell
   iocage stop gitserver
   ```
1. Mount the `repos` dataset on the `repos` folder created in the previous
   section:
   ```shell
   iocage fstab gitserver --add /mnt/tank/repos /git/repos nullfs rw 0 0
   ```

## Configuring the SSH service

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

## Testing the Git server

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

[0]: https://www.ixsystems.com/documentation/freenas/11.2-U4.1/shell.html
[1]: https://iocage.readthedocs.io/en/latest/
[2]: /self-hosted-architecture/#version-control-system
[3]: /self-hosted-architecture/#application-server
[10]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
