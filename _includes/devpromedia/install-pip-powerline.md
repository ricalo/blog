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
