{% assign documentroot = include.documentroot | default: "/usr/local/www/apache24/data" %}
{% assign servername = include.servername | default: "www.example.org" %}

## Configure access over HTTPS

To configure access over HTTPS, you need an SSL certificate, such as the ones
provided by [Let's Encrypt](https://letsencrypt.org/){: target="external"}.

1. Copy the `ca`, `crt`, and `key` files of your certificate to a folder in the
   jail.
1. By default, any `.conf` file in the `Includes` folder is added to the Apache
   configuration. Create the `/usr/local/etc/apache24/Includes/my-conf.conf`
   with the following contents:
   ```xml
   LoadModule ssl_module libexec/apache24/mod_ssl.so
   LoadModule socache_shmcb_module libexec/apache24/mod_socache_shmcb.so
   Listen 443
   <VirtualHost *:443>
     DocumentRoot "{{ documentroot }}"
     ServerName {{ servername }}:443
     SSLEngine on
     SSLCertificateFile      /path/to/certificate.crt
     SSLCertificateKeyFile   /path/to/certificate.key
     SSLCertificateChainFile /path/to/certificate.ca
   </VirtualHost>
   ```
1. Restart the web server:
   ```shell
   service apache24 onerestart
   ```
