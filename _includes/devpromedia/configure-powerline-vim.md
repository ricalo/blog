{% assign powerline-root = include.powerline-root %}

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
{% if powerline-root %}
set rtp+={{ powerline-root }}/bindings/vim/
{% else %}python3 from powerline.vim import setup as powerline_setup
python3 powerline_setup()
python3 del powerline_setup{% endif %}

set laststatus=2
```

The `laststatus` setting displays the status bar in Vim and makes Powerline
visible by default.
