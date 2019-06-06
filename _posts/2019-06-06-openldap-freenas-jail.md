---
title: Installing OpenLDAP in a jail on FreeNAS
excerpt: >
    Learn how to install the Powerline status plugin for vim, bash, tmux, and
    others.
date: 2019-03-16
categories:
  - tools
tags:
  - Powerline
  - vim
  - bash
  - tmux
---

Learn how to install the Powerline status plugin for vim, bash, tmux, and
others.

Powerline is a text-based tool that provides useful information in a variety of
contexts. The following screenshot shows Powerline displaying information about
a Git repository:

![Powerline demo][demo]

## Install and configure Powerline fonts

To install the Powerline fonts:

```sh
git clone https://github.com/powerline/fonts.git --depth=1
cd fonts
./install.sh
cd ..
rm -rf fonts
```

To configure the Powerline fonts in Terminal:

Go to **Edit** > **Preferences**. In the **Editing Profile “Your Profile”**
dialog, choose the **General** tab, select the Custom font checkbox, and click
the button to select a custom font. From the **Choose a Terminal Font** dialog,
select one of the Powerline fonts, for example **Source Code Pro for Powerline
Regular**.

You probably want to use a color scheme that works better than the standard
white on black, but that’s a personal preference.

## Install Pip and Powerline

Powerline is published in Pip, which is a package manager for Python. To install
the Pip package manager:

```sh
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python get-pip.py --user
rm get-pip.py
```

To install Powerline to the user install directory:

```sh
~/.local/bin/pip install --user powerline-status
```

## Configure Bash

To configure Powerline for bash, add the following lines to your `~/.bashrc`
file:

```sh
# Add this to your PATH if it’s not already declared
export PATH=$PATH:$HOME/.local/bin

# Powerline configuration
if [ -f ~/.local/lib/python2.7/site-packages/powerline/bindings/bash/powerline.sh ]; then
    ~/.local/bin/powerline-daemon -q
    POWERLINE_BASH_CONTINUATION=1
    POWERLINE_BASH_SELECT=1
    source ~/.local/lib/python2.7/site-packages/powerline/bindings/bash/powerline.sh
fi
```

## (Optional) Configure Vim

To configure Powerline for Vim, add the following lines to your `~/.vimrc` file:

```vim
set rtp+=$HOME/.local/lib/python2.7/site-packages/powerline/bindings/vim/
```

Optionally, add the following line to `~/.vimrc` to display the status bar in Vim
(and make the Powerline status line visible by default):

```vim
set laststatus=2
```

## (Optional) Edit your Powerline configuration

You can customize what segments appear in Powerline, or even the behavior of
specific segments. The summarized instructions to customize your Powerline
installation is the following:

1. Copy the `~/.local/lib/python2.7/site-packages/powerline/config_files/`
   folder to `~/.config/powerline`:
   ```sh
   mkdir -p ~/.config/powerline
   cp -R ~/.local/lib/python2.7/site-packages/powerline/config_files/* \
         ~/.config/powerline/
   ```
1. Edit the files there according to your needs. A good starting point is the
   `~/.config/powerline/config.json` file.
1. Reload the Powerline daemon:
   ```sh
   powerline-daemon --kill
   ```

For more information about how to customize your Powerline installation, see the
[Quick setup guide][0] in the Powerline documentation.

[0]:https://powerline.readthedocs.io/en/master/configuration.html#quick-guide
[demo]:/assets/images/powerline-demo.gif
