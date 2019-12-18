---
title: Installing a Minecraft server on FreeNAS
excerpt: >
    Learn how to install a Minecraft server on your FreeNAS server. Configure
    your installation as a service that starts every time the jail starts.
date: 2019-09-24
categories:
tags:
  - minecraft
  - freenas
  - jails
  - service
  - daemon
  - java
toc: true
---

A Minecraft server allows you to play with friends in the same virtual world.
The server keeps track of the state of the environment as users join and leave
the world.


**Note:** The instructions in this tutorial install _Minecraft: Java Edition
server_, which is only compatible with _Minecraft: Java Edition_.
{: .notice }

{% include devpromedia/create-jail.md
   jail-name="minecraft"
   packages="openjdk8 curl" %}

## Running the server for the first time

In this section, you'll perform the following tasks:

* Download the Minecraft server binaries.
* Accept the [Minecraft end user license agreement (EULA)][5]{: target="external"}.
* Start the server.

To download the Minecraft server binaries:

1. Check the [Minecraft server download page][4]{: target="external"} and copy
   the link to the `minecraft_server.x.xx.x.jar` file.

1. Open a session on the jail:
   ```shell
   iocage console minecraft
   ```
1. Create a folder to store the server files:
   ```shell
   mkdir -p /root/minecraft
   cd /root/minecraft
   ```
1. Use curl to download the server binaries. Replace the URL in the following
   example with the URL from the first step in this procedure:
   ```shell
   curl -O https://launcher.mojang.com/v1/objects/3dc3d84a581f14691199cf6831b71ed1296a9fdf/server.jar
   ```

To accept the Minecraft EULA:

1. Run the following command:
   ```shell
   java -jar /root/minecraft/server.jar
   ```
   The command prints a the following message:
   ```
    [main/ERROR]: Failed to load properties from file: server.properties
    [main/WARN]:  Failed to load eula.txt
    [main/INFO]:  You need to agree to the EULA in order to run the server.
                  Go to eula.txt for more info.
   ```
1. The previous command creates a set of files, including `eula.txt`. Make sure
   that you agree with the [Minecraft EULA][4]{: target="external"}, open
   `eula.txt`, and replace `eula=false` with `eula=true`. You can also replace
   the previous text by using the following command:
   ```shell
   sed -i .bak 's/eula=false/eula=true/g' /root/minecraft/eula.txt
   ```

To start the server for the first time, run the following command, which
includes the options:

* `Xms512M`: Sets the initial Java heap size to 512MB.
* `Xmx1024M`: Sets the maximum Java heap size to 1024MB.
* `nogui`: Starts the server without a graphical user interface.

```shell
java -Xms512M -Xmx1024M -jar /root/minecraft/server.jar nogui
```

The command displays the following output:
```
[Server thread/INFO]: Starting minecraft server version 1.14.4
[Server thread/INFO]: Loading properties
[Server thread/INFO]: Default game type: SURVIVAL
[Server thread/INFO]: Generating keypair
[Server thread/INFO]: Starting Minecraft server on *:25565
[Server thread/INFO]: Using default channel type
[Server thread/INFO]: Preparing level "world"
[Server thread/INFO]: Found new data pack vanilla, loading it automatically
[Server thread/INFO]: Reloading ResourceManager: Default
[Server thread/INFO]: Loaded 6 recipes
[Server thread/INFO]: Loaded 811 advancements
[Server thread/INFO]: Preparing start region for dimension minecraft:overworld
[Server-Worker-6/INFO]: Preparing spawn area: 0%
[Server-Worker-5/INFO]: Preparing spawn area: 1%
[Server-Worker-3/INFO]: Preparing spawn area: 2%
...
...
[Server-Worker-7/INFO]: Preparing spawn area: 92%
[Server-Worker-2/INFO]: Preparing spawn area: 97%
[Server thread/INFO]: Done! For help, type "help"
```

In the previous example, the server starts on port `25565`, which is the default
port of the Minecraft server. You'll need this information in the next section.
Your server is ready and waiting for clients to connect.

## Connecting a Minecraft client

To test the Minecraft server, connect a Minecraft: Java Edition client. Run the
following procedure from your client computer:

1. Download and install the Minecraft: Java Edition client by following the
   instructions on the [Minecraft download page][6]{: target="external"}.
1. Start Minecraft: Java Edition and click **Play**.
1. In the start screen, click **Multiplayer**.
1. In the **Play Multiplayer** screen, click **Add Server**.
1. In the **Edit Server Info** screen, type the IP address and port of your
   server in the **Server Address** field. For example, our instructions use the
   address `192.168.1.123:25565`.
1. Back in the **Play Multiplayer** screen, select the new server and click
   **Join Server**.

Your client joins the game and the server displays the following output:

```
[User Authenticator #1/INFO]: UUID of player USERNAME is ...
...
[Server thread/INFO]: USERNAME joined the game
```

You have successfully installed a Minecraft server on FreeNAS. However, the
server is running in an interactive session on the jail. The next section
explains how to run the server without an interactive session.

## Configuring the server as a daemon service

You can configure the Minecraft server to run as a service on the jail, which
allows you to start the service whenever the jail starts. You don't need to open
a session nor run the Java command.

To configure the service:

1. Open a session on the jail:
   ```shell
   iocage console minecraft
   ```
1. If it doesn't exist, create the `rc.d` folder to store the service script:
   ```shell
   mkdir -p /usr/local/etc/rc.d
   ```
1. Save the following script in the `/usr/local/etc/rc.d/minecraftd` file:
   ```shell
   #!/bin/sh
   #
   # PROVIDE: minecraftd
   # REQUIRE: LOGIN DAEMON NETWORKING mountcritlocal
   # KEYWORD: shutdown
   #
   # Use the following variables to configure the minecraft server. For example, to
   # configure the ON/OFF knob variable:
   # sysrc minecraftd_enable="YES"
   #
   # minecraftd_enable="YES"
   # minecraftd_user_dir="/root/minecraft"
   # minecraftd_jar_path="/root/minecraft/server.jar"
   # minecraftd_java_opts="-Xms512M -Xmx1024M"

   . /etc/rc.subr

   name=minecraftd
   rcvar=`set_rcvar`
   pidfile=/var/run/minecraftd.pid

   load_rc_config $name

   start_cmd="${name}_start"
   stop_cmd="${name}_stop"
   status_cmd="${name}_status"

   : ${minecraftd_enable="NO"}
   : ${minecraftd_user_dir="/root/minecraft"}
   : ${minecraftd_jar_path="/root/minecraft/server.jar"}
   : ${minecraftd_java_opts="-Xms512M -Xmx1024M"}

   minecraftd_start() {
       if [ -e $pidfile ]; then
           echo "$name already running."
       else
           echo "Starting $name..."
           /usr/sbin/daemon -f -p $pidfile \
               /usr/local/bin/java -Duser.dir=$minecraftd_user_dir \
               $minecraftd_java_opts \
               -jar $minecraftd_jar_path nogui
           echo "$name started."
       fi
   }

   minecraftd_stop() {
       if [ -e $pidfile ]; then
           echo "Stopping $name..."
           cat $pidfile | xargs kill
           echo "Stopped."
       else
           echo "$name is not running."
       fi
   }

   minecraftd_status() {
       if [ -e $pidfile ]; then
           echo "$name is running."
       else
           echo "$name is not running."
       fi
   }

   run_rc_command $1
   ```
1. Provide execute permissions to the script:
   ```shell
   chmod +x /usr/local/etc/rc.d/minecraftd
   ```
1. Configure the service to start when the jail starts:
   ```shell
   sysrc minecraftd_enable="YES"
   ```
1. Exit the jail session:
   ```shell
   exit
   ```
1. Restart the jail:
   ```shell
   iocage restart minecraft
   ```

Connect your Minecraft client again. You can use the same server configuration
from the [Connecting a Minecraft client](#connecting-a-minecraft-client)
section.


[0]: https://www.ixsystems.com/documentation/freenas/11.2-U4.1/shell.html
[1]: https://iocage.readthedocs.io/en/latest/
[2]: /self-hosted-architecture/#version-control-system
[3]: /self-hosted-architecture/#application-server
[4]: https://www.minecraft.net/download/server/
[5]: https://account.mojang.com/documents/minecraft_eula
[6]: https://www.minecraft.net/download/
[10]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
