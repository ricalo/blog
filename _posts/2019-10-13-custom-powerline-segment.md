---
title: Creating a custom Powerline segment
excerpt: >
    The Powerline status line provides useful information in your terminal
    prompt. In this post, learn how to create your custom Powerline segment
    using Python.
date: 2019-10-13
categories:
tags:
  - Powerline
  - vim
  - bash
  - tmux
  - Ubuntu
  - Python
toc: true
---

The Powerline status line provides useful information in your terminal prompt.
In this post, learn how to create your custom Powerline segment using Python.

You can create your own Powerline segments that display information tailored to
your workflow. The following screenshot shows a custom Powerline segment
displaying _hello_:

![Custom Powerline segment][screenshot]

The instructions in this post assume that you have installed Powerline and
copied the configuration files that allow you to customize the installation. To
meet these prerequisites, follow the instructions in [Installing Powerline on
Ubuntu Linux][0] including the _Edit your Powerline configuration_ section.

## Configuring the Python path

To configure your development environment, update the `PYTHONPATH` environment
variable to include your current working directory. This allows Powerline to
find your custom segment.

To configure the `PYTHONPATH` variable:

```shell
PYTHONPATH=$PYTHONPATH:/current_dir
```

## Creating the custom segment

Powerline segments are defined in Python classes. You must create the
appropriate filesystem structure to host the class that defines your custom
segment. The filesystem structure should look similar to the following:

```
current_dir/
└── my_segments
    ├── __init__.py
    └── custom.py
```

To create the filesystem structure:

1. Create the `my_segments` directory:
   ```shell
   mkdir my_segments
   ```
1. Create the `my_segments/__init__.py` file, which is just an empty file:
   ```shell
   touch my_segments/__init__.py
   ```
1. Create the `my_segments/custom.py` file, which defines the `CustomSegment`
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

## Configuring the custom segment

Powerline themes define the segments used in different applications, such as
vim, tmux, and shell. You can configure your custom segment in one of the
Powerline themes. For the purposes of this post, we'll add the custom segment to
the shell theme. The shell theme is declared in the
`$HOME/.config/powerline/themes/shell/default.json` file.

> **Note:** These instructions assume that you have copied the Powerline
> configuration files that allow you to customize your installation. For more
> information, check [Edit your Powerline configuration][1].

To add the custom theme to the shell theme, add the following segment
declaration to the list of segments in the `default.json` file:

```json
{
  "function": "my_segments.custom.hello",
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
class and the changes apply immediately. Other changes, such as changes to the
`PYTHONPATH` environment variable, require the daemon to be restarted.

[screenshot]: /assets/images/custom-powerline-screenshot.png
[0]: /install-powerline-ubuntu/
[1]: /install-powerline-ubuntu/#optional-edit-your-powerline-configuration
