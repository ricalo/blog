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
    src="{{ site.baseurl }}/assets/powerline-demo.cast"
    rows="11"
    font-size="medium"
    idle-time-limit="1"
    speed="1.5"
    title="Powerline demo"
    autoplay="true" >
</asciinema-player>

**Note:** We also have a guide that shows how to install Powerline on
[Windows 10][1] or using a [Python virtual environment][0].
{: .notice }


{% include devpromedia/install-apt-powerline.md %}

{% include devpromedia/configure-powerline-bash.md %}

{% include devpromedia/configure-powerline-vim.md %}

{% include devpromedia/configure-powerline-tmux.md %}

{% include devpromedia/edit-powerline-configuration.md %}


[0]: /blog/install-powerline-virtualenv/
[1]: /blog/install-powerline-windows/
