---
title: Our AIDE configuration for jails hosted on FreeNAS
excerpt: >
    The AIDE system helps verify the integrity of files that can be compromised
    when a host is accessed. Configuring AIDE in a jail can be tricky. A
    configuration that is too strict can lead to many false positives, which
    limits the usefulness of the integrity check.
date: 2019-09-13
categories:
tags:
  - aide
  - freenas
  - jails
  - iocage
---

After much trial and error, we have found a configuration that works for our
purposes. We didn't want to be alerted by changes caused by regular automated
tasks, such as periodic scrubs. In contrast, we should be notified if the
software in the jail was updated or if anybody—including us—opened an SSH
session.

**Warning:** These constraints work well for our situation, but might not be
appropriate for yours. This example configuration is provided as is. You
should validate that your configuration is adequate for your purposes.
{: .notice--warning }

To start, install and configure Advanced Intrusion Detection Environment (AIDE).
For more information, check the [project website][0]. Then, configure the
following entries in the `/usr/local/etc/aide.conf` configuration file:

```
/dev                L-n
!/dev/fd/5$
!/dev/pts$

/etc                R-tiger-rmd160-sha1-m-c
/etc/localtime      R-m-c-i
/etc/resolv.conf    R-m-c

!/usr/local/lib/perl5/5.28/perl/man
!/usr/local/lib/perl5/5.28/perl/man/mandoc.db
!/usr/local/lib/perl5/site_perl/man
!/usr/local/lib/perl5/site_perl/man/mandoc.db
!/usr/share/man
!/usr/share/man/mandoc.db
!/usr/share/openssl/man
!/usr/share/openssl/man/mandoc.db
```

AIDE should report differences in the files the next time it runs. This is
expected because of the updated configuration.

AIDE is great at helping detect intruders. However, you should have other
security measures in place to prevent access to your systems.

[0]: https://aide.github.io/
