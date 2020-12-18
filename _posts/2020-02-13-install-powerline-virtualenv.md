---
title: Installing Powerline in a Python virtual environment
excerpt: >
    Learn how to install Powerline in a Python virtual environment. Installing
    on a virtual environment allows you to keep your system Python free of user
    packages.
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
    src="{{ site.baseurl }}/assets/powerline-demo.cast"
    rows="11"
    font-size="medium"
    idle-time-limit="1"
    speed="1.5"
    title="Powerline demo"
    autoplay="true" >
</asciinema-player>

**Note:** It's relatively easier to install Powerline to use the system Python
by using APT packages. For this alternative, check our guide to [install
Powerline on Ubuntu][0].
{: .notice }


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


## Install Powerline in a virtual environment

1. Create a Python virtual environment in the `~/.venv` folder:
   ```shell
   python3 -m venv ~/.venv
   ```
1. Activate the virtual environment:
   ```shell
   source ~/.venv/bin/activate
   ```
1. Install the _wheel_ package:
   ```shell
   pip install wheel
   ```
1. Install _Powerline_:
   ```shell
   pip install powerline-status
   ```


{% include devpromedia/configure-powerline-bash.md
   powerline-root="$HOME/.venv/lib/python3.6/site-packages/powerline" %}

{% include devpromedia/configure-powerline-vim.md
   powerline-root="$HOME/.venv/lib/python3.6/site-packages/powerline" %}

{% include devpromedia/configure-powerline-tmux.md
   powerline-root="$HOME/.venv/lib/python3.6/site-packages/powerline" %}

{% include devpromedia/edit-powerline-configuration.md
   powerline-root="$HOME/.venv/lib/python3.6/site-packages/powerline" %}


[0]: /blog/install-powerline-ubuntu/
[1]: /blog/install-powerline-windows/
