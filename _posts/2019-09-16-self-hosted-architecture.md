---
title: A minimal architecture for a self-hosted home office or small business
excerpt: >
    Learn about a self-hosted architecture that supports the basic services that
    are usually required in a home office or small business.
date: 2019-09-13
categories:
tags:
  - freenas
  - pfsense
  - architecture
  - home office
  - small business
  - nextcloud
  - openldap
  - mariadb
  - git
---

Learn about a self-hosted architecture that supports the basic services that are
usually required in a home office or small business.

# Business requirements

The architecture supports the following requirements of the business:

* Access the data from the internal and external network
* Register users
* File and calendar sharing
* Manage code and other business artifacts

> **Note:** In general, we don't recommend hosting your own email service.
> Managing an email server is complicated, particularly keeping it secure and
> free of spam.

# Architecture overview

The architecture consists of two servers:

* A perimeter server that provides the network services that allows users to
  access the data.
* An application server that hosts the services required to register users, work
  with files and calendars, and manage business artifacts.

The follow diagram shows a view of the servers and the services they support:

![Architecture diagram](/assets/images/architecture_overview.svg)

# Physical architecture

The physical architecture consists of the following servers:

* A perimeter server
* An application server

## Perimeter server

The perimeter server allows users to access services from outside of your
network by using features such as _reverse proxy_ and _dynamic DNS_. The server
also provides other network services such as an internal DNS, firewall, and even
a VPN service.

The server requires multiple network interface cards, which allows it to route
the traffic from outside and inside your network. As you can imagine, the
perimeter server sits between your internal network and your internet service
provider.

We use [pfSense][1] as the software of our perimeter server. You can buy an
appliance from Netgate—the developer of pfSense—or you can download the software
and install it on your own device.

## Application server

The application server provides the majority of the user-facing services. This
should be your most powerful server in terms of processing power. It should also
have multiple hard drives to provide some level of resistance to failure.

The server allows you to host applications using a virtualization or container
technology. The server also provides storage for the applications hosted on top
of it.

We use [FreeNAS][2] as the software of our application server. FreeNAS can host
applications in *jails*, which is a container technology supported on BSD
systems. FreeNAS also uses ZFS as the file system, which is popular for its
redundancy, integrity checking, and snapshot features.

# Application layer

The software applications that support the business services are hosted in jails
in the application server. We only use open source software (OSS) because of its
transparency—OSS is much less likely to use tracking and surveillance tactics on
you. The OSS community is also great at providing guidance on how to install the
software and providing solutions to the issues that you might run into.

The architecture considers the following software:

## Directory services

The directory services software allows the business to register users and
computers. Other applications can use the information in the directory for
authentication and authorization purposes.

We recommend **OpenLDAP** as the directory services software. Check our guide
on how to [install OpenLDAP in a jail on your FreeNAS server][3].

## Database

The database software supports other applications that need to store structured
data. For example, the productivity software below requires a database. Your end
users most likely won't use the database server directly, unless they are
developing apps that need a database themselves.

Some applications come with a prepackaged database server. In this case, you
don't need to consider a separate server. However, installing a separate
database server allows you to fine tune your configuration and plan for future
applications that also need a database.

We use **MariaDB** as the software for our database server because our selection
of productivity software recommends using it over other alternatives. As you can
see, your selection of database depends on the application software that uses
its services.

The productivity and collaboration section lists resources that explain how to
install MariaDB in a jail on your FreeNAS server.

## Productivity and collaboration

The productivity software supports collaboration scenarios for your users. The
most basic scenarios are file and calendar sharing. However, productivity
software also provides services that the community uses to build other types of
apps.

We have chosen **Nextcloud** as our productivity and collaboration software. It
offers synchronization of files across mobile and desktop clients. Your users
can also keep their calendars on the server and use familiar clients, such as
Thunderbird, to access them. There are also apps for more advanced scenarios,
such as password management and online meetings.

For more information about how to install the software, check Samuel Dowling's
excellent guide on [how to install Nextcloud in a jail][4], which covers how to
install the database server.

## Version control system

A version control system (VCS) is useful in managing changes to the core
artifacts of your business. The software industry largely uses it to manage
source code.  However, businesses in other industries can use it as well. For
example, an online magazine or blog can use a VCS system to manage the articles
they publish.

**Git** is the most commonly used VCS system. If you have to learn a VCS system,
it's a good idea to choose git. It's used in small projects and large
organizations alike.

We chose to install a simple git server because we just needed basic
collaboration and a central source of truth for the state of our artifacts.
Check our guide on how to [install a lightweight git server on FreeNAS][5].

# Additional considerations

Consider the following items when planning your self-hosted environment:

Internet service provider
: The architecture assumes you have an internet service provider that allows the
  communication required to access the internal network from the internet.

Domain and external DNS hosting
: The architecture assumes that you have a domain and DNS hosting in place. The
  host allows you to configure the DNS entries that you need to redirect the
  users to the internal network when accessing the services from the internet.

Backup strategy
: While the application server has some resilience to errors, you should have a
  plan that allows you to recover your data in the case of an emergency.
  Ideally, the backups are stored in a remote location away from the physical
  location where the servers reside.

Certificates
: Encrypting the communication between clients and servers is a must these days.
  You can get a free certificate for this purpose from [Let's Encrypt][7].

PGP and hardware keys
: We use a combination of PGP and hardware keys to open SSH sessions to manage
  the servers. We use the hardware keys from Yubico that support the Open PGP
  function, check their [compare products][8] table for more information.

Virtual private network
: The perimeter server can provide a virtual private network (VPN) that you can
  use to access your internal network without relying in the reverse proxy
  server. This is useful for opening SSH sessions without exposing the SSH port
  to the internet.

Raspberry Pi
: There are scenarios where the FreeNAS server needs to access an OpenLDAP
  server. For example, when you configure storage space for a user directly in
  FreeNAS. In this situation, the FreeNAS server tries to access the OpenLDAP
  server whenever it boots, but fails because the OpenLDAP server is hosted in a
  jail that has not yet started. We solved this issue by getting a Raspberry Pi
  and configuring it as a secondary OpenLDAP server. The FreeNAS server uses the
  secondary server at boot time. The secondary server gets changes in user data
  thanks to the replication feature of OpenLDAP.

[0]: http://bsdadventures.com/harden-freebsd/
[1]: https://pfsense.org
[2]: https://freenas.org
[3]: /openldap-freenas-jail/
[4]: https://www.samueldowling.com/2018/12/08/install-nextcloud-on-freenas-iocage-jail-with-hardened-security/
[5]: /git-server-freenas/
[6]: https://about.gitlab.com/install/
[7]: https://letsencrypt.org/
[8]: https://www.yubico.com/products/yubikey-hardware/compare-products-series/
