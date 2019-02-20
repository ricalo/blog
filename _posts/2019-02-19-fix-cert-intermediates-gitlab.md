---
title: Certificate misses intermediates error on GitLab Pages
excerpt: >
    Learn how to fix the *Certificates misses intermediates* error when you
    register a Let's Encrypt certificate for your site hosted on GitLab Pages.
date: 2019-02-19
categories:
  - tips
tags:
  - gitlab
  - letsencrypt
  - ssl
  - https
---

Learn how to fix the *Certificates misses intermediates* error when you register
a Let's Encrypt certificate for your site hosted on GitLab Pages.

Follow these steps to reproduce the error:

1. Log in to GitLab and select your project.
1. Go to **Settings** > **Pages**.
1. Click **New Domain** or select **Details** on a domain that you already have
   registered.
1. Enter your certificate in the **Certificate (PEM)** field and your
   certificate key in the **Key (PEM)** field.
1. Click **Create New Domain** (or **Save Changes**) if you're updating an
   existing domain.

The page displays the following error:

> Certificate misses intermediates

## Possible cause

GitLab Pages requires you to also upload the intermediate certificates in the
chain of trust.

## Solution

Download and provide the Let's Encrypt intermediate certificate. According to
[Intermediate Certificates][1], Let's Encrypt uses an intermediate that is
cross-signed by another authority, IdenTrust. Use this cross-signed certificate
as the intermediate in your GitLab Pages domain registration:

1. Copy the [cross-signed certificate][2] text from the Let's Encrypt site.
1. In the **Certificate (PEM)** field, enter your certificate, followed by a new
   line, and the cross-signed certificate afterwards, as shown in the following
   example:
   ```
   -----BEGIN CERTIFICATE-----
   Your_certificate
   ...
   ...
   -----END CERTIFICATE-----
   -----BEGIN CERTIFICATE-----
   The cross-signed_certificate
   ...
   ...
   -----END CERTIFICATE-----
   ```
1. Click **Save Changes**.

[1]: https://letsencrypt.org/certificates#intermediate-certificates
[2]: https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem.txt
