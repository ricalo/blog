---
title: Creating a custom Powerline segment
excerpt: >
    The Powerline status line provides useful information in your terminal
    prompt. In this post, learn how to create your custom Powerline segment
    using Python.
date: 2019-10-13
categories:
tags:
  - powerline
  - vim
  - bash
  - tmux
  - ubuntu
  - python
toc: true
---

The Powerline status line provides useful information in your terminal prompt.
In this post, learn how to create your custom Powerline segment using Python.

You can create your own Powerline segments that display information tailored to
your workflow. The following screenshot shows a custom Powerline segment
displaying _hello_:

![Custom Powerline segment][screenshot]


## Prerequisites

This post assumes that you have installed Powerline and copied the configuration
files needed to customize the installation.  To meet these prerequisites, check
[Installing Powerline on Ubuntu Linux][0] including the _Edit your Powerline
configuration_ section.


## Creating the custom segment

Powerline segments are defined in Python classes. You must create a package to
host the class that defines your custom segment. The structure of the package
looks similar to the following:

```
current_dir/
├── mysegments
│   ├── __init__.py
│   └── custom.py
└── setup.py
```

To create the package:

1. Create the `setup.py` file, copy the following contents, and save the file:
   ```python
   import setuptools

   setuptools.setup(
       name="mysegments",
       version="0.0.1",
       author="Your name",
       author_email="username@example.org",
       description="My custom Powerline segments.",
       long_description="A long description for the " + \
           "package that includes the custom segments.",
       long_description_content_type="text/plain",
       packages=setuptools.find_packages(),
   )
   ```
   **Note:** You must append your username or organization name to the `name`
   field if you plan to upload your package to a central location, such as
   pypi.org. Such step is not required in this tutorial.
   {: .notice }
1. Create the `mysegments` directory:
   ```shell
   mkdir mysegments
   ```
1. Create the `mysegments/__init__.py` file, which is just an empty file:
   ```shell
   touch mysegments/__init__.py
   ```
1. Create the `mysegments/custom.py` file, which defines the `CustomSegment`
   class. Copy the following contents and save the file:
   ```python
   # vim:fileencoding=utf-8:noet
   from __future__ import (unicode_literals, division, absolute_import, print_function)

   from powerline.segments import Segment, with_docstring
   from powerline.theme import requires_segment_info, requires_filesystem_watcher

   @requires_filesystem_watcher
   @requires_segment_info
   class CustomSegment(Segment):
     divider_highlight_group = None

     def __call__(self, pl, segment_info, create_watcher):
       return [{
         'contents': 'hello',
         'highlight_groups': ['information:regular'],
         }]

   hello = with_docstring(CustomSegment(), '''Return a custom segment.''')
   ```
   This segment uses the `information:regular` highlight group, which defines
   the color and other appearance features. To check other highlight groups,
   browse the `$HOME/.config/powerline/colorschemes/default.json` file.


## Installing the package in editable mode

Installing the package in editable mode allows you to make changes to the code
without having to reinstall the package to see the changes.

To install the package in editable mode:

```shell
pip install --editable ./
```

To check that the package is installed:
```shell
pip list
```

You should see an entry similar to the following:
```
mysegments (0.0.1, /path_to_current_dir)
```


## Configuring the custom segment

Powerline themes define the segments used in different applications, such as
vim, tmux, and shell. You can configure your custom segment in one of the
Powerline themes. For the purposes of this post, we'll add the custom segment to
the shell theme. The shell theme is declared in the
`$HOME/.config/powerline/themes/shell/default.json` file.

**Note:** These instructions assume that you have copied the configuration files
that allow you to customize your Powerline installation. For more information,
check [Edit your Powerline configuration][1].
{: .notice }

To add the custom theme to the shell theme, add the following segment
declaration to the list of segments in the `default.json` file:

```json
{
  "function": "mysegments.custom.hello",
  "priority": 10
}
```


## Testing the custom segment

You must restart the Powerline daemon to apply the changes in the previous
sections:

```shell
powerline-daemon --kill
```

After you restart the daemon, the custom Powerline segment appears in your shell
when the prompt refreshed. You can keep making changes to the `CustomSegment`
class and the changes apply immediately.


[screenshot]: /assets/images/custom-powerline-screenshot.png
[0]: /install-powerline-ubuntu/
[1]: /install-powerline-ubuntu/#optional-edit-your-powerline-configuration
