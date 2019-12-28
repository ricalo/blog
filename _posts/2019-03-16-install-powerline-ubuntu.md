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
contexts. The following screenshot shows Powerline displaying information about
a Git repository:

![Powerline demo][demo]


## Install and configure Powerline fonts

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


## Install pip and Powerline

Powerline is available in pip, which is a package manager for Python. To install
pip and Powerline:

1. Install the curl and distutils packages:
   ```shell
   sudo apt update
   sudo apt install --yes curl python3-distutils
   ```
1. Get the pip installer:
```shell
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
```
1. Install pip:
   ```shell
   python3 get-pip.py --user
   ```
1. Optionally, remove the pip installer:
   ```shell
   rm get-pip.py
   ```
1. To install Powerline:
   ```shell
   ~/.local/bin/pip3 install --user powerline-status
   ```
   The `--user` flag indicates that pip installs the package following the _user
   scheme_. In other words, the package is installed in a location specific to the
   current user.


You might see the following message in the output of the previous command:
> ERROR: launchpadlib 1.10.6 requires testresources, which is not installed.

The installation can continue without further issues.


## Configure Bash

To configure Powerline for bash, add the following lines to your `$HOME/.bashrc`
file:

```shell
# Add this to your PATH if it’s not already declared
export PATH=$PATH:$HOME/.local/bin

# Powerline configuration
if [ -f $HOME/.local/lib/python3.6/site-packages/powerline/bindings/bash/powerline.sh ]; then
  $HOME/.local/bin/powerline-daemon -q
  POWERLINE_BASH_CONTINUATION=1
  POWERLINE_BASH_SELECT=1
  source $HOME/.local/lib/python3.6/site-packages/powerline/bindings/bash/powerline.sh
fi
```

To apply the changes to your current terminal:
```shell
source ~/.bashrc
```

After running the previous command or restarting your terminal, the Powerline
segments appear in your prompt.


## Configure Vim

To use Powerline, Vim requires support for Python 3. You can check the features
included in your installation with the following command:
```
vim --version
```
Look for the **python3** entry, which must have a plus sign (`+`) next to it to
indicate that the feature is available. If your version of vim doesn't include
support for Python 3, you can install the latest vim package, which at the time
of this writing includes support for Python 3:
```shell
sudo apt update
sudo apt install --yes vim
```

To configure Powerline for Vim, add the following lines to your `$HOME/.vimrc`
file:

```vim
set rtp+=$HOME/.local/lib/python3.6/site-packages/powerline/bindings/vim/
set laststatus=2
```

The `laststatus` setting displays the status bar in Vim and makes Powerline
visible by default.


## Edit your Powerline configuration

You can customize what segments appear in Powerline, or even the behavior of
specific segments. The summarized instructions to customize your Powerline
installation is the following:

1. Copy the `$HOME/.local/lib/python3.6/site-packages/powerline/config_files/`
   folder to `$HOME/.config/powerline`:
   ```shell
   mkdir -p $HOME/.config/powerline
   cp -R $HOME/.local/lib/python3.6/site-packages/powerline/config_files/* \
         $HOME/.config/powerline/
   ```
1. Edit the files there according to your needs. A good starting point is the
   `$HOME/.config/powerline/config.json` file.
1. Reload the Powerline daemon:
   ```shell
   powerline-daemon --replace
   ```

The following example shows how to configure Powerline on the shell to behave as
the animation shown in the introduction of this guide:

1. To show the vcs segment—which displays information about git repositories—on
   the left side of the shell, replace the **default** theme with the
   **default_leftonly** theme in the `$HOME/.config/powerline/config.json` file:
   ```json
   ···
   "shell": {
     "colorscheme": "default",
     "theme": "default_leftonly",
     "local_themes": {
       "continuation": "continuation",
       "select": "select"
     }
   },
   ···
   ```
1. To make the vcs segment display in a different color when the git repository
   is dirty, add the `status_colors` attribute to the vcs segment in the
   `$HOME/.config/powerline/themes/shell/default_leftonly.json` file:
   ```json
   ···
   {
     "function": "powerline.segments.common.vcs.branch",
     "priority": 40,
     "args": { "status_colors": true }
   }
   ···
   ```

You can also create custom Powerline segments that display information tailored
to your workflow. For more information, check [Creating a custom Powerline
segment][0].

[demo]: /assets/images/powerline-demo.gif
[0]: /custom-powerline-segment/
