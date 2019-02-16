---
title: Hosting a lightweight Git server on FreeNAS
excerpt: >
    Learn how to host a lightweight Git server on your FreeNAS machine that you
    can use for personal repositories.
date: 2019-02-14
tags:
  - git
  - freenas
---

Adding a simple git server to your FreeNAS


# Procedure:

# Creating iocage jail

1. Fetch the release used in this tutorial
   ```
   iocage fetch --release 11.2-RELEASE
   ```

   OUTPUT:
   ```
   Fetching: 11.2-RELEASE

   Extracting: base.txz...
   Extracting: lib32.txz...
   Extracting: doc.txz...
   Extracting: src.txz...

   * Updating 11.2-RELEASE to the latest patch level...
   src component not installed, skipped
   Looking up update.FreeBSD.org mirrors... 3 mirrors found.
   Fetching metadata signature for 11.2-RELEASE from update2.freebsd.org... done.
   Fetching metadata index... done.
   Fetching 2 metadata patches.. done.
   Applying metadata patches... done.
   Inspecting system... done.
   Preparing to download files... done.
   Fetching 1 patches. done.
   Applying patches... done.
   The following files will be updated as part of updating to
   11.2-RELEASE-p9:
   ...

   src component not installed, skipped
   Installing updates... done.
   ```
1. List availables releases
   ```
   iocage list --release
   ```
1. Create jail
   ```
   iocage create --name gitserver --release 11.2-RELEASE dhcp="on" vnet="on" bpf="yes"
   ```

   OUTPUT
   ```
   gitserver successfully created!
   ```
1. Create user for git operations
   ```
   iocage exec --host_user root gitserver mkdir -p /git/repos
   iocage exec --host_user root gitserver pw useradd -n git -d /git
   iocage exec --host_user root gitserver passwd git
   Changing local password for git
   New Password:
   Retype New Password:
   iocage exec --host_user root gitserver chown -R git /git
   ```

1. (Optional) Store the repos in a ZFS dataset
   1. Create a dataset
   ```
   zfs create scraps/repos
   ```
   1. Stop the jail
   ```
   iocage stop gitserver
   * Stopping gitserver
     + Running prestop OK
     + Stopping services OK
     + Tearing down VNET OK
     + Removing devfs_ruleset: 11 OK
     + Removing jail process OK
     + Running poststop OK
   ```
   1. Mount the dataset
   ```
   iocage fstab gitserver --add /mnt/scraps/repos /git/repos nullfs rw 0 0
   Successfully added mount to gitserver's fstab
   ```

# Install Git

1. Log to the console on the jail.
   ```
   iocage console gitserver
   ```
1. Update the pkg repository
   ```
   pkg update
   ```
1. Install Git package
   ```
   pkg install -y git
   ```
# Configure SSH service

1. Configure service to start on boot
   ```
   iocage exec --host_user root gitserver sysrc sshd_enable="yes"
   ```
1. Test the ssh connection
   ```
   ssh git@gitserver
   The authenticity of host 'gitserver (192.168.10.101)' can't be established.
   ECDSA key fingerprint is SHA256:EUwWPX8wdW1boiVxnmkcjw/Yf11uivGG87qiI/zlo8s.
   Are you sure you want to continue connecting (yes/no)? yes
   Warning: Permanently added 'gitserver,192.168.10.101' (ECDSA) to the list of known hosts.
   Password for git@gitserver:
   ```

# Create repository on server

1. Open an SSH session to the server.
   ```
   ssh git@gitserver
   Password for git@gitserver:
   ```
1. Create folder to host repo
   ```
   mkdir repos/my-repo
   cd repos/my-repo
   ```
1. Initialize repo
   ```
   git init --bare
   ```

# Add remote and push to server

1. Commit a test file in a repo in your local machine.
   ```
   mkdir my-repo
   cd my-repo
   git init
   echo "A test file" >> test-file
   git add test-file
   git commit -m "A test commit"
   ```
1. Register the server as a remote in your repo
   ```
   git remote add origin git@gitserver:repos/my-repo
   ```
1. Push to git server
   ```
   git push origin master
   Password for git@gitserver:
   Counting objects: 3, done.
   Writing objects: 100% (3/3), 881 bytes | 881.00 KiB/s, done.
   Total 3 (delta 0), reused 0 (delta 0)
   To gitserver:repos/my-repo
    * [new branch]      master -> master
   ```

