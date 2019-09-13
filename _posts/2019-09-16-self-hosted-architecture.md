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

# Business requirements considered

The architecture supports the following requirements of the business:

* Register users and computers
* Manage code and other business artifacts
* File and calendar sharing

> **Note:** In general, we don't recommend hosting your own email service.
> Managing an email server is complicated, particularly keeping it secure and
> free of spam.

# Physical architecture

The physical architecture consists of the following servers:

* A perimeter server
* An application server

## Perimeter server

The perimeter server allows users to access services from outside of your
network by using a feature called _reverse proxy_. The server also provides
other network services such as DNS, firewall, and even a VPN service. The server
requires multiple network interface cards, which allows it to route the traffic.

We use [pfSense][1] as the software of our perimeter server. You can buy an
appliance from Netgate—the developer of pfSense—or you can download the software
and install it on your own device.

## Application server

The application server provides the majority of the user-facing services. This
should be your most powerful server in terms of processing power. It should also
have multiple hard drives to provide redundancy for your data. The server allows
you to host applications using a virtualization or container technology.

We use [FreeNAS][2] as the software of our application server. FreeNAS can host
applications in jails, a container technology. It also uses ZFS as the file
system, which is popular for its redundancy and integrity checking features.

# Application layer

The software that support the business services are hosted in jails in the
application server. The architecture consider the following base software:

## OpenLDAP

## MariaDB

## Nextcloud

## Git server


[0]: http://bsdadventures.com/harden-freebsd/
[1]: https://pfsense.org
[2]: https://freenas.org
