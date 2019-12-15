{% assign jail-name = include.jail-name %}
{% assign freebsd-release = include.freebsd-release | default: "11.2-RELEASE" %}
{% assign ip4_addr = include.ip4_addr | default: "re0|192.168.1.123/24" %}
{% assign defaultrouter = include.defaultrouter | default: "192.168.1.1" %}
{% assign packages = include.packages %}

## Preparing the jail

The instructions in this post host the app server in a jail on FreeNAS. To learn
why we use jails for this purpose, check the
[Application server](/self-hosted-architecture/#application-server) section of
our self-hosted architecture post.

In this section, you'll perform the following tasks:

* Create a jail.
* Configure networking on the jail.
* Install the prerequisite packages.

Run the commands from a session in your FreeNAS server. You can use the
[FreeNAS shell](https://www.ixsystems.com/documentation/freenas/11.2-U4.1/shell.html){: target="external"} for this purpose.

To create a jail:

1. Fetch or update the release version of FreeBSD for jail usage:
   ```sh
   iocage fetch --release {{ freebsd-release }}
   ```
1. Create a jail named `{{ jail-name }}`:
   ```shell
   iocage create --name {{ jail-name }} --release {{ freebsd-release }}
   ```

To configure networking on the jail:

1. Configure the IP address. The following example sets the IP address to
   `{{ ip4_addr | split: "|" | last | split: "/" | first }}` using a subnet mask
   of `{{ ip4_addr | split: "|" | last | split: "/" | last }}` bits on the
   `{{ ip4_addr | split: "|" | first }}` interface. The command uses the
   [CIDR notation](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing){: target="external"}.
   ```shell
   iocage set ip4_addr="{{ ip4_addr }}" {{ jail-name }}
   ```
1. Configure the default router. The following example sets the default router
   to `{{ defaultrouter }}`:
   ```shell
   iocage set defaultrouter={{ defaultrouter }} {{ jail-name }}
   ```

{% if packages %}
Start the jail and open a session to complete the rest of the tasks in this
section:

```shell
iocage start {{ jail-name }}
iocage console {{ jail-name }}
```

{% assign pkg-array = packages | split: " " %}
{% case pkg-array.size %}
{% when 1 %}
Install the **{{ packages }}** package:
{% when 2 %}
Install the **{{ pkg-array | first }}** and **{{ pkg-array | last }}** packages:
{% else %}
Install the following packages:
{% for pkg in pkg-array %}
- {{ pkg }}
{%- endfor -%}
{% endcase %}

```shell
pkg update
pkg install --yes {{ packages }}
```
{% endif %}
