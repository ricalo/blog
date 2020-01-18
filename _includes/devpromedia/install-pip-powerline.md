## Install Powerline using pip

Powerline is also available in pip, which is a package manager for Python. If
you prefer to install Powerline using pip:

```shell
~/.local/bin/pip3 install --user powerline-status
```

The `--user` flag indicates that pip installs the package following the _user
scheme_. In other words, the package is installed in a location specific to the
current user.

You might see the following message in the output of the previous command, which
doesn't affect the Powerline installation:
> ERROR: launchpadlib 1.10.6 requires testresources, which is not installed.

Make sure to replace `/usr/share/powerline` with
`$HOME/.local/lib/python3.6/site-packages/powerline` in the instructions of this
tutorial if you install Powerline using pip.
