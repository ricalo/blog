---
title: Xbox 360 controller doesn't work on Ubuntu
excerpt: >
    Learn how to fix configuration an issue on Ubuntu where the Xbox button of a
    controller keeps blinking and Ubuntu doesn't detect it. Additionally, the
    `xboxdrv` tool throws a `LIBUSB_ERROR_NOT_FOUND` error.
date: 2019-08-24
categories:
tags:
  - xbox
  - madcatz
  - ubuntu
---

Learn how to fix configuration an issue on Ubuntu where the *Xbox button* of a
controller keeps blinking and Ubuntu doesn't detect it. Additionally, the
`xboxdrv` tool throws a `LIBUSB_ERROR_NOT_FOUND` error. This issue was
experienced with the Mad Catz Arcade FightStick KE for the Xbox 360 on Ubuntu
18.04.

Follow these steps to reproduce the error:

1. Connect the controller to an USB port in your computer.
1. The *Xbox button* lights start blinking non stop.
1. Type the following command `sudo xboxdrv`, the command throws the following
   error:
   > Error couldn't claim the USB interface: LIBUSB_ERROR_NOT_FOUND

**Note:** With [Proton 4.11-3][1]{: target="external"}, games can access
gamepads directly. You shouldn't need to use the solution explained below. For
example, our Mad Catz stick works okay in
[Samurai Shodown V Special][2]{: target="external"} withouth disabling the
autosuspend feature. However, we are still experiencing the issue with the wrong
mapping of buttons, which is resolved by the workaround explained in
[Additional steps](#additional-steps).
{: .notice }

## Solution

Disable the autosuspend feature of the usbcore module. For more information,
check [Power Management for USB][0]{: target="external"}:

1. Set the `usbcore.autosuspend` option to `-1` in the `GRUB_COMDLINE_LINUX`
   line in the Grub configuration file `/etc/default/grub`:
   ```json
   GRUB_CMDLINE_LINUX="usbcore.autosuspend=-1"
   ```
1. Update the grub configuration:
   ```shell
   $ sudo update-grub
   ```
1. Restart the computer.

Once you restart the computer, the *Xbox button* lights stop blinking.

## Additional steps

If the controller has the button mapping wrong—for example, the *Start button*
doesn't work but others do—run the following commands:

```shell
$ sudo apt install xboxdrv
$ sudo xboxdrv --mimic-xpad --detach-kernel-driver
```

The buttons should work appropriately.

[0]: https://www.kernel.org/doc/Documentation/usb/power-management.txt
[1]: https://github.com/ValveSoftware/Proton/releases/tag/proton-4.11-3
[2]: https://store.steampowered.com/app/1076550/SAMURAI_SHODOWN_V_SPECIAL/
