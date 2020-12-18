---
title: Installing Powerline on WSL Windows 10
excerpt: >
    Install the Powerline status plugin on the Windows Subsystem for Linux
    (WSL). Get useful information on the shell prompt and provide a new look to
    your terminal on Windows 10.
date: 2019-12-28
categories:
tags:
  - powerline
  - vim
  - bash
  - wsl
  - windows
toc: true
---

Powerline is a text-based tool that provides useful information in a variety of
contexts. The following screenshot shows the Powerline status plugin on the
Windows 10 Ubuntu app displaying information about a Git repository:

![Powerline demo][demo]


This guide requires that you install the Ubuntu app on Windows 10. For more
information, check [WSL][0]{: target="external"} on
the Ubuntu wiki.

**Note:** We also have a guide that shows how to install Powerline on
[Linux][1].
{: .notice }

## Install and configure Powerline fonts

To install the Powerline fonts:

1. Open a Powershell session as administrator.
1. Download and expand the Powerline fonts repository:
   ```powershell
   powershell -command "& { iwr https://github.com/powerline/fonts/archive/master.zip -OutFile ~\fonts.zip }"
   Expand-Archive -Path ~\fonts.zip -DestinationPath ~
   ```
1. Update the execution policy to allow the installation of the fonts:
   ```powershell
   Set-ExecutionPolicy Bypass
   ```
1. Run the installation script:
   ```powershell
   ~\fonts-master\install.ps1
   ```
1. Revert the execution policy back the default value:
   ```powershell
   Set-ExecutionPolicy Default
   ```

To configure the fonts:

1. Open the Ubuntu app.
1. Open the **Properties** dialog.
1. From the **Font** tab, select one of the Powerline fonts, such as _ProFont
   for Powerline_.
1. Click **OK**.

Run the rest of this tutorial from within the Ubuntu app.

{% include devpromedia/install-apt-powerline.md %}

{% include devpromedia/configure-powerline-bash.md %}

{% include devpromedia/configure-powerline-vim.md %}

{% include devpromedia/configure-powerline-tmux.md %}

{% include devpromedia/edit-powerline-configuration.md %}

{% include devpromedia/install-pip-powerline.md %}


[demo]: /blog/assets/images/powerline-wsl.png
[0]: https://wiki.ubuntu.com/WSL
[1]: /blog/install-powerline-ubuntu/
