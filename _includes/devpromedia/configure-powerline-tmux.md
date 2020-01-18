## Configure tmux

To configure Powerline in tmux, add the following to your `~/.tmux.conf` file:

```shell
set -g default-terminal "screen-256color"
source "/usr/share/powerline/bindings/tmux/powerline.conf"
```

You must set the correct value for your terminal using the `default-terminal`
setting. If you don't configure the correct value for your terminal, you might
run into some issues. For example, Vim inside of tmux doesn't show colored
Powerline segments. The `screen-256color` or `xterm-256color` values work fine
in our environments.
