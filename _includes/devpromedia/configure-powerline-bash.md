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
