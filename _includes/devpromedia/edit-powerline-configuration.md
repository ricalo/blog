## Edit your Powerline configuration

You can customize what segments appear in Powerline, or even the behavior of
specific segments. The summarized instructions to customize your Powerline
installation is the following:

1. Copy the `/usr/share/powerline/config_files/` folder to
   `$HOME/.config/powerline`:
   ```shell
   mkdir -p $HOME/.config/powerline
   cp -R /usr/share/powerline/config_files/* \
         $HOME/.config/powerline/
   ```
1. Edit the files there according to your needs. A good starting point is the
   `$HOME/.config/powerline/config.json` file.
1. Reload the Powerline daemon:
   ```shell
   powerline-daemon --replace
   ```

The following example shows how to configure Powerline on the shell to behave as
the image shown in the introduction of this guide:

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
segment](/custom-powerline-segment/).
