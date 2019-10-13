---
title: Installing Powerline on Ubuntu Linux
excerpt: >
    Learn how to install the Powerline status plugin for Vim, Bash, and tmux on
    Ubuntu Linux. Get useful information on your shell prompt and make your
    terminal look beautiful.
date: 2019-09-21
categories:
tags:
  - powerline
  - vim
  - bash
  - tmux
  - ubuntu
toc: true
---

Learn how to install the Powerline status plugin for Vim, Bash, and tmux on
Ubuntu Linux. Get useful information on your shell prompt and make your terminal
look beautiful.

Powerline is a text-based tool that provides useful information in a variety of
contexts. The following screenshot shows Powerline displaying information about
a Git repository:

![Powerline demo][demo]

## Install and configure Powerline fonts

To install the Powerline fonts:

1. Clone the Powerline fonts repository to the `fonts` folder in your computer:
   ```shell
   git clone https://github.com/powerline/fonts.git --depth=1 fonts
   ```
1. Run the installation script:
   ```shell
   ./fonts/install.sh
   ```

To configure the fonts in the Ubuntu Terminal app:

1. Open Terminal.
1. Go to **Edit** > **Preferences**.
1. In the **Editing Profile** dialog, choose the **General** tab, enable
   **Custom font**, and click the button to select a font.
1. From the **Choose a Terminal Font** dialog, select one of the Powerline
   fonts, such as **Source Code Pro for Powerline Regular**.

While you are in the **Editing Profile** dialog, check the different color
schemes in the **Colors** tab. You might find one that works better for you than
the default.

## Install pip and Powerline

Powerline is available in pip, which is a package manager for Python. To install
pip:

```sh
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python get-pip.py --user
rm get-pip.py
```

To install Powerline:

```sh
$HOME/.local/bin/pip install --user powerline-status
```

The `--user` flag indicates that pip installs the package following the _user
scheme_. In other words, the package is installed in a location specific to the
current user.

## Configure Bash

To configure Powerline for bash, add the following lines to your `$HOME/.bashrc`
file:

```sh
# Add this to your PATH if it’s not already declared
export PATH=$PATH:$HOME/.local/bin

# Powerline configuration
if [ -f $HOME/.local/lib/python2.7/site-packages/powerline/bindings/bash/powerline.sh ]; then
    $HOME/.local/bin/powerline-daemon -q
    POWERLINE_BASH_CONTINUATION=1
    POWERLINE_BASH_SELECT=1
    source $HOME/.local/lib/python2.7/site-packages/powerline/bindings/bash/powerline.sh
fi
```

## (Optional) Configure Vim

To configure Powerline for Vim, add the following lines to your `$HOME/.vimrc` file:

```vim
set rtp+=$HOME/.local/lib/python2.7/site-packages/powerline/bindings/vim/
```

Optionally, add the following line to `$HOME/.vimrc` to display the status bar in
Vim and make the Powerline status line visible by default:

```vim
set laststatus=2
```

## (Optional) Edit your Powerline configuration

You can customize what segments appear in Powerline, or even the behavior of
specific segments. The summarized instructions to customize your Powerline
installation is the following:

1. Copy the `$HOME/.local/lib/python2.7/site-packages/powerline/config_files/`
   folder to `$HOME/.config/powerline`:
   ```sh
   mkdir -p $HOME/.config/powerline
   cp -R $HOME/.local/lib/python2.7/site-packages/powerline/config_files/* \
         $HOME/.config/powerline/
   ```
1. Edit the files there according to your needs. A good starting point is the
   `$HOME/.config/powerline/config.json` file.
1. Reload the Powerline daemon:
   ```sh
   powerline-daemon --kill
   ```

You can also create custom Powerline segments that display information tailored
to your workflow. For more information, check [Creating a custom Powerline
segment][0].

[demo]: /assets/images/powerline-demo.gif
[0]: /custom-powerline-segment/
