---
title: Hosting a lightweight Git server on FreeNAS
excerpt: >
    Learn how to host a lightweight Git server in an iocage jail on your
    FreeNAS appliance. Access the Git server from clients that have an
    authorized certificate to establish an SSH connection.
date: 2019-02-14
categories:
  - tools
tags:
  - git
  - freenas
  - iocage
  - jails
  - ssh
---

Hosting Git repositories (repos) on a server allows you to keep track of and
store files used in your projects, even those files that contain sensitive data.
You can use a hosting service like GitHub to manage your repos. However, you
should not use such services for repos that contain sensitive information. For
repos that store sensitive data, use a Git server hosted by a trusted party.
This post shows how to set up a lightweight Git server on your FreeNAS
appliance.

## Creating an iocage jail

FreeNAS uses the iocage manager to create independent environments isolated from
the operating system. Such environments are called jails. This post uses a jail
to host the Git server.

The following steps show how to create an iocage jail and configure an user for
git operations. Run the commands in a shell on the FreeNAS server:

1. This post uses FreeBSD version `11.2-RELEASE` to create the jail. Run the
   following command to fetch or update the version of FreeBSD for jail usage:
   ```shell
   $ iocage fetch --release 11.2-RELEASE
   ```
1. Run the following command to create a jail named `gitserver` using the
   version of FreeBSD fetched in the previous step:
   ```shell
   $ iocage create \
       --name gitserver --release 11.2-RELEASE \
       dhcp="on" vnet="on" bpf="yes"
   ```

   The previous command creates a jail that uses DHCP to configure the IP
   settings on a virtual network.
1. The following commands create a new user, configure the folder to store the
   repos, and prompts for a new password:
   ```shell
   $ iocage exec --host_user root gitserver \
       mkdir -p /git/repos
   $ iocage exec --host_user root gitserver \
       pw useradd -n git -d /git
   $ iocage exec --host_user root gitserver \
       chown -R git /git
   $ iocage exec --host_user root gitserver \
       passwd git
   ```

   The last command changes the password for the user. Type and make a note of
   the new password.

## (Optional) Storing the repos in a ZFS dataset

One of the advantages of hosting your Git server is that you can decide the
location where to store the repos. The following procedure shows how to
configure a ZFS dataset to store the repos.

To store the repos in a dataset, run the following commands in a shell on the
FreeNAS server:

1. Create a dataset named `repos` in the `scraps` pool:
   ```shell
   $ zfs create scraps/repos
   ```
1. Use the following command to stop the jail:
   ```shell
   $ iocage stop gitserver
   ```
1. Mount the `repos` dataset on the `repos` folder created in the previous
   section:
   ```shell
   $ iocage fstab gitserver --add \
       /mnt/scraps/repos /git/repos nullfs rw 0 0
   ```

## Installing Git on the server

The server requires the Git binaries to manage the repos. The following
procedure shows how to install Git on the server.

Run the following commands in a shell on the FreeNAS server:

1. Update the `pkg` repo.
   ```shell
   $ iocage exec --host_user root gitserver \
       pkg update
   ```
1. Install the `git` package.
   ```shell
   $ iocage exec --host_user root gitserver \
       pkg install -y git
   ```

## Configuring the SSH service

Git clients use an SSH connection to communicate with the server. This section
shows how to configure the SSH service to accept connections from clients.

On your client machine, verify that you have a certificate that you can use to
communicate with the server, or create a new one:

1. Check your existing SSH keys.
   ```shell
   $ ls -la ~/.ssh
   ```
   Usually, you can use the key stored in the `id_rsa.pub` file. If the previous
   command lists the `id_rsa.pub` file go to step three. Otherwise continue to
   step two.
1. Generate an SSH key. Replace *email@example.com* with your email address.
   ```shell
   $ ssh-keygen -t rsa -b 4096 -C "email@example.com"
   ```
   The previous command prompts for the location to which to save the key.
   Accept the default value of `id_rsa`.
1. Copy the SSH public key as an authorized key.
   ```shell
   $ ssh-copy-id -i ~/.ssh/id_rsa.pub git@gitserver
   ```

On the FreeNAS server, configure the SSH service:

1. Configure the SSH service to start on boot.
   ```shell
   $ iocage exec --host_user root gitserver \
       sysrc sshd_enable="yes"
   ```
1. Start (or restart) the SSH service.
   ```shell
   $ iocage exec --host_user root gitserver \
       service sshd onerestart
   ```

To validate the configuration, connect to the Git server by running the
following ssh command from your client machine:

```shell
$ ssh git@gitserver
```
Verify that you can connect to the server without using a password.

## Testing the Git server with a new repo

To test the Git server, create an empty repo that clients can use to push
commits.

1. Open an SSH session to the server (or reuse the connection created in the
   previous section):
   ```shell
   $ ssh git@gitserver
   ```
1. Create a new `my-repo.git` folder in the `repos` location created previously.
   ```shell
   $ mkdir repos/my-repo.git
   $ cd repos/my-repo.git
   ```
1. Initialize the repo to accept commits from clients.
   ```shell
   $ git init --bare
   ```

On the client machine, create a repo and push a commit to the server.

1. Create and initialize a repo.
   ```shell
   $ mkdir my-repo
   $ cd my-repo
   $ git init
   ```
1. Add a test file and commit the changes to the repo.
   ```shell
   $ echo "A test file" >> test-file
   $ git add test-file
   $ git commit -m "A test commit"
   ```
1. Register the repo on the server as a tracked repo.
   ```shell
   $ git remote add origin git@gitserver:repos/my-repo.git
   ```
1. Push the new commit to the Git server
   ```shell
   $ git push origin master
   ```
   The previous command pushes the commit to the server. You can clone, pull,
   and push additional commits from clients that have access to the server.

This post showed how to host a Git server in an iocage jail on FreeNAS. 
