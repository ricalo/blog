---
title: An architecture for a self-hosted business
excerpt: >
    Learn about a self-hosted architecture that supports services usually
    required in a small business or home office.
date: 2019-09-21
teaser: /assets/images/architecture_overview.svg
toc: true
categories:
tags:
  - freenas
  - pfsense
  - architecture
  - home office
  - small business
  - openldap
  - mariadb
  - git
  - raspberry pi
---

## Business requirements

The architecture supports the following requirements of the business:

* Access the data from the internal and external network
* Register users
* Manage code and other business artifacts

**Note:** In general, we don't recommend hosting your own email service.
Managing an email server is complicated, particularly keeping it secure and free
of spam.
{: .notice }

## Architecture overview

The architecture consists of two servers:

* A perimeter server that provides the network services that allows users to
  access the data.
* An application server that hosts the services required to register users, work
  with files and calendars, and manage business artifacts.

The follow diagram shows a view of the servers and the services they support:

![Architecture diagram][overview]

## Physical architecture

The physical architecture consists of the following servers:

* A perimeter server
* An application server

### Perimeter server

The perimeter server allows users to access services from outside of your
network by using features such as _reverse proxy_ and _dynamic DNS_. The server
also provides other network services such as an internal DNS, firewall, and even
a VPN service.

The server requires multiple network interface cards, which allows it to route
the traffic from outside and inside your network. As you can imagine, the
perimeter server sits between your internal network and your internet service
provider.

We use [pfSense][1]{: target="external"} as the software of our perimeter
server. You can buy an appliance from Netgate—the developer of pfSense—or you
can download the software and install it on your own device.

### Application server

The application server provides the majority of the user-facing services. This
should be your most powerful server in terms of processing power. It should also
have multiple hard drives to provide plenty of storage for your applications and
some level of resilience. For an example, check our [3rd Gen AMD Ryzen
build][12].

The server hosts applications using a virtualization or container technology,
which allows you to create independent environments isolated from the operating
system. Hosting the applications in these independent environments improves the
security of your infrastructure as a whole because it's more difficult that a
single application has access to the other environments or the operating system.

We chose [FreeNAS][2]{: target="external"} as the software of the application
server. FreeNAS uses the [iocage][10]{: target="external"} container manager to
create independent environments isolated from the operating system. Such
environments are called _jails_.

FreeNAS supports the [ZFS][11]{: target="external"} filesystem, which offers
multiple features—such as redundancy, integrity checking, and snapshots—that
help ensure that the data stays consistent and available. You can enable the ZFS
features on a particular dataset on the filesystem. By specifying on which
dataset to store the data, you have more control over what ZFS features to
use for your applications.

## Network services

The perimeter server hosts the network services we need in this architecture.
The software, pfSense, is an open source network security solution that provides
the features that we need. pfSense has great documentation that allows you to
configure the system to suit your needs.

### Dynamic DNS

If your internet service provider (ISP) doesn't assign a static IP address,
you require a mechanism to update the entry of your internal domain in your
external DNS. This allows clients accessing from the internet to find the IP
address if your internal network.

For example, if your top domain is `example.org` and your internal domain is
`home.example.org`, you can configure pfSense to update the `home` entry in your
external DNS every time your ISP assigns a new IP address.

### Internal DNS

The internal DNS allows your clients to find services in your network. For
example, if your internal domain is `home.example.org` you could configure the
following entries in your internal DNS:

* `ldap.home.example.org` for your directory services
* `sql.home.example.org` for your database
* `vcs.home.example.org` for your version control system

### Reverse proxy

A reverse proxy allows clients in the internet to use web services in your
internal network. The external DNS resolves all requests to your internal
network as the same address. The reverse proxy catches these requests from the
internet and sends them to the right server in your internal network.

## Application layer

The software that support the business services is hosted in jails in the
application server. We use open source software (OSS) because of its community.
The OSS community is great at providing guidance on how to install the software
and suggesting solutions to the issues that you might run into.

### Directory services

The directory services software allows the business to register users and
computers. Other applications can use the information in the directory for
authentication and authorization purposes.

We recommend **OpenLDAP** as the directory services software. Check our guide
on how to [install OpenLDAP in a jail on your FreeNAS server][3].

### Database

The database software supports other applications that need to store structured
data. For example, the VCS software below requires a database. Your end users
most likely won't use the database server directly, unless they are developing
apps that need a database themselves.

Some applications come with a prepackaged database server. In this case, you
don't need to consider a separate server. However, installing a separate
database server allows you to fine tune your configuration and plan for future
applications that also need a database.

We use **MariaDB** as the software for our database server because our selection
of VCS software works well with it. As you can see, your selection of database
depends on the application software that uses its services. Check our guide on
how to [install MariaDB on FreeNAS][13].

### Version control system

A version control system (VCS) is useful in managing changes to the core
artifacts of your business. The software industry largely uses it to manage
source code. However, businesses in other industries can use it as well. For
example, an online magazine or blog can use a VCS system to manage the articles
they publish.

You can host a centralized VCS, which offers the following benefits:

* Facilitates collaboration between members of the team. They can compare and
  review different versions of the business artifacts using familiar tools.
* Keeps one version of the truth for files in the repositories, which you can
  use to make business decisions. For example:
  * Decide what version of the code to use to build your app.
  * Decide what version of a post to publish on your online blog.
* Allows you to store the data on a server computer in addition to storing the
  data on client computers, such as laptops.

**Git** is the most commonly used VCS system. If you have to learn a VCS system,
it's a good idea to choose Git. It's used in small projects and large
organizations around the world.

Initially, we chose to install a simple Git server because we just needed basic
collaboration and a central source of truth for the state of our artifacts.
Check our guide on how to [install a lightweight Git server on FreeNAS][5].
However, as our requirements evolve, we're looking into other solutions that
provide better collaboration features, such as [Gitea][15]{: target="external"}.

## Additional considerations

Consider the following additional items when planning your self-hosted
environment:

Internet service provider
: The architecture assumes you have an ISP that allows the communication
  required to access the internal network from the internet. Some ISPs actively
  block this communication.

Domain and external DNS hosting
: The architecture assumes that you have a provider that hosts your domain and
  external DNS. The hosting service allows you to configure the DNS entries that
  you need to redirect the users to the internal network when accessing the
  services from the internet. If you don't have a static IP address assigned,
  your external DNS should provide a mechanism to automatically update a DNS
  entry. Such mechanism makes it easy to update the DNS entry of your internal
  network each time your ISP assigns a new IP address.

Backup strategy
: While the application server has some resilience to errors, you should have a
  plan that allows you to recover your data in the case of an emergency.
  Ideally, the backups are stored in a remote location away from the physical
  location where the servers reside.

Certificates
: Encrypting the communication between clients and servers is a must these days.
  You can get a free certificate for this purpose from
  [Let's Encrypt][7]{: target="external"}.

PGP and hardware keys
: We use a combination of PGP and hardware keys to open SSH sessions to manage
  the servers. We use the hardware keys from Yubico that support the Open PGP
  function, check their [compare products][8]{: target="external"} table for
  more information.

Virtual private network
: The perimeter server can provide a virtual private network (VPN) that you can
  use to access your internal network. This is useful for opening SSH sessions
  without exposing the SSH port to the internet. It usually only takes a few
  minutes after you expose the SSH ports of your servers to the internet before
  someone tries to get unauthorized access.

Raspberry Pi
: There are scenarios where the FreeNAS server needs to access an OpenLDAP
  server. For example, when you configure storage space for a user directly in
  FreeNAS. In this situation, the FreeNAS server tries to access the OpenLDAP
  server whenever it boots, but fails because the OpenLDAP server is hosted in a
  jail that has not yet started. We solved this issue by getting a Raspberry Pi
  and configuring it as a secondary OpenLDAP server. The FreeNAS server uses the
  secondary server at boot time. The secondary server gets changes in user data
  thanks to the replication feature of OpenLDAP. All the other LDAP clients in
  the network use the more reliable primary server hosted as a jail on FreeNAS.

Intruder detection
: You can configure the Advanced Intrusion Detection Environment (AIDE) system,
  which helps identify when an intruder accesses your servers. Check [our AIDE
  configuration for jails hosted on FreeNAS][9] guide.

[overview]: /blog/assets/images/architecture_overview.svg
[1]: https://pfsense.org
[2]: https://freenas.org
[3]: /blog/ldap-server-freenas/
[4]: https://nextcloud.com/
[5]: /blog/git-server-freenas/
[6]: https://about.gitlab.com/install/
[7]: https://letsencrypt.org/
[8]: https://www.yubico.com/products/yubikey-hardware/compare-products-series/
[9]: /blog/aide-freenas-jail/
[10]: https://iocage.readthedocs.io/en/latest/
[11]: https://www.freenas.org/zfs/
[12]: /blog/amd-ryzen-freenas/
[13]: /blog/mariadb-server-freenas/
[14]: /blog/nextcloud-server-freenas/
[15]: https://gitea.io/
