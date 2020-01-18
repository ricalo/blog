## Configure Bash

To configure Powerline for bash, add the following lines to your `$HOME/.bashrc`
file:

```shell
# Powerline configuration
if [ -f /usr/share/powerline/bindings/bash/powerline.sh ]; then
  powerline-daemon -q
  POWERLINE_BASH_CONTINUATION=1
  POWERLINE_BASH_SELECT=1
  source /usr/share/powerline/bindings/bash/powerline.sh
fi
```

To apply the changes to your current terminal:
```shell
source ~/.bashrc
```

After running the previous command or restarting your terminal, the Powerline
segments appear in your prompt.
