---
title: Installing Powerline on Ubuntu Linux
excerpt: >
    Learn how to install the Powerline status plugin for Vim, Bash, and tmux on
    Ubuntu Linux. Get useful information on your shell prompt and make your
    terminal look beautiful.
date: 2019-12-28
categories:
tags:
  - powerline
  - vim
  - bash
  - tmux
  - ubuntu
toc: true
---

Powerline is a text-based tool that provides useful information in a variety of
contexts. The following demo shows Powerline displaying information about
a Git repository:

<asciinema-player
    src="/assets/powerline-demo.cast"
    rows="11"
    font-size="medium"
    idle-time-limit="1"
    speed="1.5"
    title="Powerline demo"
    autoplay="true" >
</asciinema-player>

**Note:** We also have a guide that shows how to install Powerline on
[Windows 10][1].
{: .notice }


{% include devpromedia/install-apt-powerline.md %}

{% include devpromedia/configure-powerline-bash.md %}

{% include devpromedia/configure-powerline-vim.md %}

{% include devpromedia/configure-powerline-tmux.md %}

{% include devpromedia/edit-powerline-configuration.md %}


## Manually install Powerline fonts

To install the Powerline fonts:

1. Install git:
   ```shell
   sudo apt update
   sudo apt install --yes git
   ```
1. Clone the Powerline fonts repository to the `fonts` folder in your computer:
   ```shell
   git clone https://github.com/powerline/fonts.git --depth=1 fonts
   ```
1. Run the installation script:
   ```shell
   ./fonts/install.sh
   ```
1. Optionally, you can remove your copy of the fonts repository. This guide
   doesn't need it anymore:
   ```shell
   rm -rf fonts/
   ```

To configure the fonts in the Ubuntu Terminal app:

1. Open Terminal.
1. Go to **Edit** > **Preferences**.
1. In the **Preferences - Profile** dialog, choose the **Text** tab, enable
   **Custom font**, and click the button right next to it to select a font.
1. From the **Choose a Terminal Font** dialog, select one of the Powerline
   fonts, such as _Source Code Pro for Powerline Regular_.


{% include devpromedia/install-pip-powerline.md %}


[demo]: /assets/images/powerline-demo.gif
[1]: /install-powerline-windows/
