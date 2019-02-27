# Problem

There is a known problem with suspend and resume on MacBook Pro Retina
mid-2015 (`MacBookPro11,4`) under Linux: internal SD Card Reader (USB,
05ac:8406) disappears after first suspend/resume cycle, and then all
subsequent tries to suspend are failing.

Links:

* https://bugzilla.kernel.org/show_bug.cgi?id=111201
* https://bugzilla.kernel.org/show_bug.cgi?id=202509
* https://www.spinics.net/lists/linux-usb/msg164261.html
* https://www.spinics.net/lists/linux-usb/msg176258.html

This repository contains:

* [various information about my MacBook, where this problem
appears](hwinfo);
* [patched uhubctl](uhubctl) (patch disables check for power switching
support);
* kernel logs of various manually performed tests, with kernel
configuration (see `test-*` directories).

Descriptions of individual tests, with results:

## Test 0 (FAIL)

* **=> Boot kernel without patches/quirks:**
    * card reader exists
* **=> Suspend #1:**
    * suspend/resume ok
    * *card reader missing*
* **=> Suspend #2:**
    * *suspend fails*
    * *card reader missing*
* **=> Rebind xhci_hcd**
    * *card reader missing*
* **=> Suspend #3:**
    * *suspend fails*
    * *card reader missing*

## Test 1 (FAIL)

* **=> Boot kernel without patches/quirks:**
    * card reader exists
* **=> Remove card reader device:**
    * remove ok
    * card reader missing
* **=> Suspend #1:**
    * suspend/resume ok
    * *card reader missing*
* **=> Suspend #2:**
    * suspend/resume ok
    * *card reader missing*
* **=> Rebind xhci_hcd**
    * *card reader missing*
* **=> Suspend #3:**
    * suspend/resume ok
    * *card reader missing*
* **=> Suspend #3:**
    * *suspend fails*
    * *card reader missing*

## Test 2 (FAIL)

* **=> Boot kernel without patches/quirks:**
    * card reader exists
* **=> Remove card reader device:**
    * remove ok
    * card reader missing
* **=> Suspend #1:**
    * suspend/resume ok
    * *card reader missing*
* **=> Suspend #2:**
    * suspend/resume ok
    * *card reader missing*
* **=> Suspend #3:**
    * suspend/resume ok
    * *card reader missing*

Removing device using `/sys/devices/*/*/usb*/*-*/remove` unbreaks the
suspend, but not fixes card reader itself.

## Test 3 (FAIL)

* **=> Boot kernel without patches, cmdline ==
`usbcore.quirks=05ac:8406:b` (USB_QUIRK_RESET_RESUME)**
    * card reader exists
* **=> Suspend #1:**
    * suspend/resume ok
    * *card reader missing*
* **=> Suspend #2:**
    * suspend/resume ok
    * *card reader missing*
* **=> Suspend #2:**
    * *suspend fails*
    * *card reader missing*

## Test 4 (FAIL)

* **=> Boot kernel without patches, cmdline ==
`usbcore.quirks=05ac:8406:m` (USB_QUIRK_DISCONNECT_SUSPEND)**
    * card reader exists
* **=> Suspend #1:**
    * suspend/resume ok
    * *card reader missing*
* **=> Suspend #2:**
    * suspend/resume ok
    * *card reader missing*
* **=> Suspend #3:**
    * *suspend fails*
    * *card reader missing*

## Test 5 (FAIL)

* **=> Boot kernel without patches, cmdline == `xhci_hcd.quirks=0x80`
(XHCI_RESET_ON_RESUME)**
    * card reader exists
* **=> Suspend #1:**
    * suspend/resume ok
    * *card reader missing*
* **=> Suspend #2:**
    * suspend/resume ok
    * *card reader missing*
* **=> Suspend #3:**
    * *suspend fails*
    * *card reader missing*

## Test 6 (FAIL)

* **=> Boot kernel without patches/quirks:**
    * card reader exists
* **=> Suspend #1:**
    * suspend/resume ok
    * *card reader missing*
* **=> Turn off port using patched hubctl:**
    * card reader missing
* **=> Turn on port using patched hubctl:**
    * *card reader missing*
* **=> Rebind xhci_hcd**
    * *card reader missing*
* **=> Suspend #2:**
    * suspend/resume ok
    * *card reader missing*
* **=> Suspend #3:**
    * suspend/resume ok
    * *card reader missing*

## Test 7 (SUCCESS)

* **=> Boot kernel without patches/quirks:**
    * card reader exists
* **=> Turn off port using patched hubctl:**
    * card reader still exists in lsusb
* **=> Suspend #1:**
    * suspend/resume ok
    * *card reader missing*
* **=> Turn on port using patched hubctl:**
    * *card reader missing*
* **=> Rebind xhci_hcd**
    * card reader exists
* **=> Turn off port using patched hubctl:**
    * card reader still exists in lsusb
* **=> Suspend #2:**
    * suspend/resume ok
    * *card reader missing*
* **=> Rebind xhci_hcd**
    * card reader exists
* **=> Turn off port using patched hubctl:**
    * card reader still exists in lsusb
* **=> Suspend #3:**
    * suspend/resume ok
    * *card reader missing*
* **=> Rebind xhci_hcd**
    * card reader exists
* **=> Turn off port using patched hubctl:**
    * card reader still exists in lsusb
* **=> Suspend #4:**
    * suspend/resume ok
    * *card reader missing*
* **=> Rebind xhci_hcd**
    * card reader exists

This is the first test where both suspend/resume and SD card reader are
working.

(WiFi broke during the last suspend/resume cycle, this is a different
story, and it is fixable by reloading module brcmfmac)

## Test 8 (SUCCESS)

* **=> Boot kernel without patches, cmdline == `xhci_hcd.quirks=0x80`
(XHCI_RESET_ON_RESUME)**
    * card reader exists
* **=> Turn off port using patched hubctl:**
    * card reader still exists in lsusb
* **=> Suspend #1:**
    * suspend/resume ok
    * card reader exists
* **=> Turn off port using patched hubctl:**
    * card reader still exists in lsusb
* **=> Suspend #2:**
    * suspend/resume ok
    * card reader exists
* **=> Turn off port using patched hubctl:**
    * card reader still exists in lsusb
* **=> Suspend #3:**
    * suspend/resume ok
    * card reader exists
* **=> Turn off port using patched hubctl:**
    * card reader still exists in lsusb
* **=> Suspend #4:**
    * suspend/resume ok
    * card reader exists

Everything is the same as in Test 7, but reuires less manual
interaction.

(again, unrelated WiFi problems after suspend/resume)

## Test 9 (FAIL)

This repeats Test 2, but with USB 3.0 flash attached to one of the
external ports:

	Bus 002 Device 003: ID 05ac:8406 Apple, Inc.
	Bus 002 Device 002: ID 1b1c:1a15 Corsair Voyager Slider Flash Drive
	Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub

	/:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/6p, 5000M
	    |__ Port 2: Dev 2, If 0, Class=Mass Storage, Driver=usb-storage, 5000M
	    |__ Port 4: Dev 3, If 0, Class=Mass Storage, Driver=usb-storage, 5000M

* **=> Boot kernel without patches/quirks:**
    * card reader exists
    * usb flash exists
* **=> Remove card reader device:**
    * remove ok
    * card reader missing
    * usb flash exists
* **=> Suspend #1:**
    * suspend/resume ok
    * *card reader missing*
    * usb flash exists
* **=> Suspend #2:**
    * suspend/resume ok
    * *card reader missing*
    * usb flash exists
* **=> Suspend #3:**
    * suspend/resume ok
    * *card reader missing*
    * usb flash exists
* **=> Suspend #4:**
    * suspend/resume ok
    * *card reader missing*
    * usb flash exists

Suspend/resume with device attached to external USB 3.0 port works as
expected. It looks like only the internal port (or SD card reader device attached to it) is problematic.

## Test 10 (SUCCESS)

Test kernel patch which turns off power on port before switching link state
to U3. XHCI_RESET_ON_RESUME enables host controller reset on resume, so
port is powered on automatically.

* **=> Boot patched kernel, cmdline ==
`xhci_hcd.quirks=0x80 usbcore.quirks=05ac:8406:p` (XHCI_RESET_ON_RESUME,
USB_QUIRK_PWR_CYCLE_ON_SUSPEND):**
    * card reader exists
* **=> Suspend #1:**
    * suspend/resume ok
    * card reader exists
* **=> Suspend #2:**
    * suspend/resume ok
    * card reader exists
* **=> Suspend #3:**
    * suspend/resume ok
    * card reader exists
* **=> Suspend #4:**
    * suspend/resume ok
    * card reader exists

## Test 11 (FAIL)

Similar to Test 10, but port power is turned off **after** switching link
state to U3.

* **=> Boot patched kernel, cmdline ==
`xhci_hcd.quirks=0x80 usbcore.quirks=05ac:8406:p` (XHCI_RESET_ON_RESUME,
USB_QUIRK_PWR_CYCLE_ON_SUSPEND):**
    * card reader exists
* **=> Suspend #1:**
    * suspend/resume ok
    * *card reader missing*
* **=> Suspend #2:**
    * suspend/resume ok
    * *card reader missing*
* **=> Suspend #3:**
    * *suspend fails*
    * *card reader missing*

## Test 12 (FAIL)

Similar to Test 10, but using only *usb_acpi_set_power_state* in
*xhci_set_port_power*.

* **=> Boot patched kernel, cmdline ==
`xhci_hcd.quirks=0x80 usbcore.quirks=05ac:8406:p` (XHCI_RESET_ON_RESUME,
USB_QUIRK_PWR_CYCLE_ON_SUSPEND):**
    * card reader exists
* **=> Suspend #1:**
    * suspend/resume ok
    * *card reader missing*
* **=> Suspend #2:**
    * suspend/resume ok
    * *card reader missing*
* **=> Suspend #3:**
    * *suspend fails*
    * *card reader missing*

## Test 13 (SUCCESS)

Test kernel patch which disables link on port instead of switching link state
to U3. XHCI_RESET_ON_RESUME enables host controller reset on resume, so
link is enabled automatically.

* **=> Boot patched kernel, cmdline ==
`xhci_hcd.quirks=0x80 usbcore.quirks=05ac:8406:p` (XHCI_RESET_ON_RESUME,
USB_QUIRK_DISABLE_DURING_SUSPEND):**
    * card reader exists
* **=> Suspend #1:**
    * suspend/resume ok
    * card reader exists
* **=> Suspend #2:**
    * suspend/resume ok
    * card reader exists
* **=> Suspend #3:**
    * suspend/resume ok
    * card reader exists
* **=> Suspend #4:**
    * suspend/resume ok
    * card reader exists

## Test 14 (FAIL)

Same as Test 13, but without XHCI_RESET_ON_RESUME.

* **=> Boot patched kernel, cmdline ==
`usbcore.quirks=05ac:8406:p` (USB_QUIRK_DISABLE_DURING_SUSPEND):**
    * card reader exists
* **=> Suspend #1:**
    * suspend/resume ok
    * *card reader missing*
* **=> Suspend #2:**
    * suspend/resume ok
    * *card reader missing*

## Test 15 (SUCCESS)

Test new patch that should disable link before resume, re-enable link
after resume and reset the device.

* **=> Boot patched kernel, cmdline ==
`usbcore.quirks=05ac:8406:p` (USB_QUIRK_DISABLE_LINK_ON_SUSPEND):**
    * card reader exists
* **=> Suspend #1:**
    * suspend/resume ok
    * card reader exists
* **=> Suspend #2:**
    * suspend/resume ok
    * card reader exists
* **=> Suspend #3:**
    * suspend/resume ok
    * card reader exists
* **=> Suspend #4:**
    * suspend/resume ok
    * card reader exists
* **=> Suspend #5:**
    * suspend/resume ok
    * card reader exists
* **=> Suspend #6:**
    * suspend/resume ok
    * card reader exists
* **=> Suspend #7:**
    * suspend/resume ok
    * card reader exists

## Test 16 (FAIL)

This happened during normal use of laptop.

It looks like with my patches system survives only short suspend but not
the long one. This happened on latest kernel from Fedora 29 with
patches applied. Unfortunately, debug logging was disabled.

## Test 17 (SUCCESS)

Trying to reproduce what happended in Test 16, but with debug enabled.
Suspend length seems does not matter. This time everything worked as
expected.

## Test 18 (FAIL)

Trying patch from Mathias Nyman:
https://marc.info/?l=linux-usb&m=155015657209860&w=2

* **=> Boot patched kernel without quirks:**
    * card reader exists

* **=> Suspend #1:**
    * suspend/resume ok
    * *card reader missing*

* **=> Suspend #2:**
    * *suspend fails*
    * *card reader missing*

## Test 19 (FAIL)

Trying to reproduce what happended in Test 16, but with debug enabled.
Problem appeared again, with debug enabled. Kernel log is huge (~5G).

Potentially interesting lines:

```
...
[60683.714182] PM: suspend devices took 0.934 seconds
[60686.645612] xhci_hcd 0000:00:14.0: clear port connect change, actual port 3 status  = 0x280
[60686.752372] xhci_hcd 0000:00:14.0: get port status, actual port 3 status  = 0x280
[60686.752378] xhci_hcd 0000:00:14.0: Get port status returned 0x280
[60686.970893] PM: resume devices took 0.326 seconds
[89295.118057] PM: suspend devices took 0.891 seconds
[89297.993768] xhci_hcd 0000:00:14.0: clear port connect change, actual port 3 status  = 0x280
[89298.100722] xhci_hcd 0000:00:14.0: get port status, actual port 3 status  = 0x280
[89298.100726] xhci_hcd 0000:00:14.0: Get port status returned 0x280
[89298.669905] PM: resume devices took 0.677 seconds
[89341.471200] PM: suspend devices took 0.935 seconds
[89344.397226] xhci_hcd 0000:00:14.0: clear port connect change, actual port 3 status  = 0x280
[89344.503937] xhci_hcd 0000:00:14.0: get port status, actual port 3 status  = 0x280
[89344.503941] xhci_hcd 0000:00:14.0: Get port status returned 0x280
[89345.073397] PM: resume devices took 0.677 seconds
[89478.005991] PM: suspend devices took 0.930 seconds
[89480.938092] xhci_hcd 0000:00:14.0: clear port connect change, actual port 3 status  = 0x280
[89481.045154] xhci_hcd 0000:00:14.0: get port status, actual port 3 status  = 0x280
[89481.045159] xhci_hcd 0000:00:14.0: Get port status returned 0x280
[89482.948524] PM: resume devices took 2.011 seconds
[90527.837722] PM: suspend devices took 0.884 seconds
[90530.759223] xhci_hcd 0000:00:14.0: clear port connect change, actual port 3 status  = 0x280
[90530.866108] xhci_hcd 0000:00:14.0: get port status, actual port 3 status  = 0x280
[90530.866115] xhci_hcd 0000:00:14.0: Get port status returned 0x280
[90531.435176] PM: resume devices took 0.677 seconds
[114123.965070] PM: suspend devices took 0.929 seconds
[114126.880362] xhci_hcd 0000:00:14.0: clear port connect change, actual port 3 status  = 0x280
[114126.987251] xhci_hcd 0000:00:14.0: get port status, actual port 3 status  = 0x280
[114126.987256] xhci_hcd 0000:00:14.0: Get port status returned 0x280
[114127.205603] PM: resume devices took 0.326 seconds
[129910.663807] PM: suspend devices took 0.932 seconds
[129913.557532] xhci_hcd 0000:00:14.0: clear port connect change, actual port 3 status  = 0x280
[129913.664178] xhci_hcd 0000:00:14.0: get port status, actual port 3 status  = 0x280
[129913.664184] xhci_hcd 0000:00:14.0: Get port status returned 0x280
[129914.565688] PM: resume devices took 1.009 seconds
[143995.901665] PM: suspend devices took 0.931 seconds
[143998.799691] xhci_hcd 0000:00:14.0: clear port connect change, actual port 3 status  = 0x280
[143998.906982] xhci_hcd 0000:00:14.0: get port status, actual port 3 status  = 0x280
[143998.906986] xhci_hcd 0000:00:14.0: Get port status returned 0x280
[143999.124351] PM: resume devices took 0.325 seconds
[152414.724917] PM: suspend devices took 0.935 seconds
[152417.618568] xhci_hcd 0000:00:14.0: clear port connect change, actual port 3 status  = 0x280
[152417.725533] xhci_hcd 0000:00:14.0: get port status, actual port 3 status  = 0x280
[152417.725538] xhci_hcd 0000:00:14.0: Get port status returned 0x280
[152418.294834] PM: resume devices took 0.677 seconds
[175739.696508] PM: suspend devices took 0.931 seconds
[175742.648687] xhci_hcd 0000:00:14.0: clear port connect change, actual port 3 status  = 0x280
[175742.755327] xhci_hcd 0000:00:14.0: get port status, actual port 3 status  = 0x280
[175742.755332] xhci_hcd 0000:00:14.0: Get port status returned 0x280
[175742.973617] PM: resume devices took 0.326 seconds
[199327.543423] PM: suspend devices took 0.914 seconds
[199330.456007] xhci_hcd 0000:00:14.0: clear port connect change, actual port 3 status  = 0x280
[199330.562787] xhci_hcd 0000:00:14.0: get port status, actual port 3 status  = 0x280
[199330.562794] xhci_hcd 0000:00:14.0: Get port status returned 0x280
[199330.781033] PM: resume devices took 0.326 seconds
[231745.097397] PM: suspend devices took 1.015 seconds
[231747.999063] xhci_hcd 0000:00:14.0: clear port connect change, actual port 3 status  = 0x280
[231748.105697] xhci_hcd 0000:00:14.0: get port status, actual port 3 status  = 0x280
[231748.105703] xhci_hcd 0000:00:14.0: Get port status returned 0x280
[231748.323231] PM: resume devices took 0.325 seconds
[232140.437522] PM: suspend devices took 0.929 seconds
[232143.371131] xhci_hcd 0000:00:14.0: clear port connect change, actual port 3 status  = 0x280
[232143.477828] xhci_hcd 0000:00:14.0: get port status, actual port 3 status  = 0x280
[232143.477833] xhci_hcd 0000:00:14.0: Get port status returned 0x280
[232143.696145] PM: resume devices took 0.326 seconds
[250619.048326] PM: suspend devices took 0.929 seconds
[250621.984990] xhci_hcd 0000:00:14.0: clear port connect change, actual port 3 status  = 0x280
[250622.091757] xhci_hcd 0000:00:14.0: get port status, actual port 3 status  = 0x280
[250622.091763] xhci_hcd 0000:00:14.0: Get port status returned 0x280
[250622.309781] PM: resume devices took 0.326 seconds
[301626.024791] PM: suspend devices took 0.932 seconds
>>> last successful resume (card reader is working):
[301628.927560] xhci_hcd 0000:00:14.0: clear port connect change, actual port 3 status  = 0x280
[301629.034327] xhci_hcd 0000:00:14.0: get port status, actual port 3 status  = 0x280
[301629.034334] xhci_hcd 0000:00:14.0: Get port status returned 0x280
[301629.251752] PM: resume devices took 0.325 seconds
>>> last successful suspend:
[351671.877535] PM: suspend devices took 0.932 seconds
>>> LS_U3 instead of LS_SS_DISABLED after resume?:
[351674.740952] xhci_hcd 0000:00:14.0: get port status, actual port 3 status  = 0x1263
[351674.740955] xhci_hcd 0000:00:14.0: Get port status returned 0x263
[351674.741044] xhci_hcd 0000:00:14.0: get port status, actual port 3 status  = 0x1263
[351674.741045] xhci_hcd 0000:00:14.0: Get port status returned 0x263
[351674.805034] xhci_hcd 0000:00:14.0: get port status, actual port 3 status  = 0x1263
[351674.805038] xhci_hcd 0000:00:14.0: Get port status returned 0x263
[351674.840496] PM: resume devices took 0.100 seconds
[351674.841313] xhci_hcd 0000:00:14.0: get port status, actual port 3 status  = 0x1263
[351674.841321] xhci_hcd 0000:00:14.0: Get port status returned 0x263
[351674.841366] usb 2-4: USB disconnect, device number 5
[351674.841373] usb 2-4: unregistering device
[351674.841381] usb 2-4: unregistering interface 2-4:1.0
[351674.841550] usb 2-4: usb_disable_device nuking all URBs
>>> trying to reset:
[351674.849603] xhci_hcd 0000:00:14.0: get port status, actual port 3 status  = 0x202e0
[351674.849606] xhci_hcd 0000:00:14.0: Get port status returned 0x102e0
[351674.849627] xhci_hcd 0000:00:14.0: set port reset, actual port 3 status  = 0x202f0
[351674.910992] usb usb2-port4: not reset yet, waiting 60ms
>>> first failing suspend:
[403139.301047] PM: Some devices failed to suspend, or early wake event detected
...
```

## Test 20 (SUCCESS)

Test new patch with slightly changed logic (waiting loops added).

* **=> Boot patched kernel, without additional quirks:**
    * card reader exists
* **=> Suspend #1:**
    * suspend/resume ok
    * card reader exists
* **=> Suspend #2:**
    * suspend/resume ok
    * card reader exists
* **=> Suspend #3:**
    * suspend/resume ok
    * card reader exists
* **=> Suspend #4:**
    * suspend/resume ok
    * card reader exists
* **=> Suspend #5:**
    * suspend/resume ok
    * card reader exists
* **=> Suspend #6:**
    * suspend/resume ok
    * card reader exists

## Test 21 (FAIL)

Same as in Test 20, but with additional patch which changes link
state to U3 before disabling link completely.

* **=> Boot patched kernel, without additional quirks:**
    * card reader exists
* **=> Suspend #1:**
    * suspend/resume ok
    * card reader missing

Obviously this does not work.

## Test 22 (FAIL)

Trying another patch from Mathias Nyman:
https://www.spinics.net/lists/linux-usb/msg177378.html

* **=> Boot patched kernel, without additional quirks:**
    * card reader exists
* **=> Suspend #1:**
    * suspend/resume ok
    * *card reader missing*
* **=> Suspend #2:**
    * suspend/resume ok
    * *card reader missing*
* **=> Suspend #3:**
    * *suspend fails (wakes up automatically with closed lid)*
    * *card reader missing*

## Test 23 (FAIL)

Same as Test 22, but trying to manually disable ACPI wakeup by
XHCI.

* **=> Boot patched kernel, without additional quirks:**
    * card reader exists
* **=> grep XHC1 /proc/acpi/wakeup**
    * `XHC1	  S3	*enabled   pci:0000:00:14.0`
* **=> Suspend #1:**
    * suspend/resume ok
    * *card reader missing*
* **=> grep XHC1 /proc/acpi/wakeup**
    * `XHC1	  S3	*enabled   pci:0000:00:14.0`
* **=> echo XHC1 >/proc/acpi/wakeup**
* **=> grep XHC1 /proc/acpi/wakeup**
    * `XHC1	  S3	*disabled  pci:0000:00:14.0`
* **=> Suspend #2:**
    * suspend/resume ok
    * *card reader missing*
* **=> grep XHC1 /proc/acpi/wakeup**
    * `XHC1	  S3	*disabled  pci:0000:00:14.0`
* **=> Suspend #3:**
    * suspend/resume ok
    * *card reader missing*
* **=> grep XHC1 /proc/acpi/wakeup**
    * `XHC1	  S3	*disabled  pci:0000:00:14.0`
* **=> Suspend #4:**
    * suspend/resume ok
    * *card reader missing*

As expected, combination of patch and `echo XHC1 >/proc/acpi/wakeup`
fixes suspend/resume. Card reader remains broken.

## Test 24 (FAIL)

Trying to reproduce the card reader problem without full system
suspend.

* **=> Boot kernel, without any patches or quirks:**
    * card reader exists
* **=> cat /sys/bus/usb/devices/2-4/power/runtime_suspended_time**
    * `0`
* **=> cat /sys/bus/usb/devices/2-4/power/control**
    * `on`
* **=> rmmod uas usb-storage**
* **=> echo auto >/sys/bus/usb/devices/2-4/power/control**
* **=> Wait few seconds...**
* **=> cat /sys/bus/usb/devices/2-4/power/runtime_suspended_time**
    * some non-zero value
    * card reader still exists in `lsusb` output
* **=> echo on >/sys/bus/usb/devices/2-4/power/control**
    * *card reader missing*

The same happens if both uas and usb-storage modules are blacklisted
from the beginning. And everything works fine with USB 3.0 flash drive.

It looks like Apple's card reader just breaks after changing the link
state to U3.

## Test 25 (FAIL)

Same as Test 24, but without hub autosuspend.

* **=> Boot kernel, without any patches or quirks:**
    * card reader exists
* **=> cat /sys/bus/usb/devices/2-4/power/runtime_suspended_time**
    * `0`
* **=> cat /sys/bus/usb/devices/2-4/power/control**
    * `on`
* **=> cat /sys/bus/usb/devices/usb2/power/runtime_suspended_time**
    * `0`
* **=> cat /sys/bus/usb/devices/usb2/power/control**
    * `auto`
* **=> echo on >/sys/bus/usb/devices/usb2/power/control**
* **=> rmmod uas usb-storage**
* **=> echo auto >/sys/bus/usb/devices/2-4/power/control**
* **=> Wait few seconds...**
* **=> cat /sys/bus/usb/devices/2-4/power/runtime_suspended_time**
    * some non-zero value
* **=> cat /sys/bus/usb/devices/usb2/power/runtime_suspended_time**
    * `0` (hub is not suspended)
    * card reader still exists in `lsusb` output
* **=> echo on >/sys/bus/usb/devices/2-4/power/control**
    * *card reader missing*

## Test 26 (FAIL)

Trying to reproduce problem from Test 16 and Test 19 using device
autosuspend. Reproduced successfully and much faster (less than 10
minutes).

This happens even faster if other USB-related activity happens during
the device resume. Continuously moving cursor using touchpad is enough.

Relevant parts of dmesg:

```
[   83.401198] usb usb2-port4: enabling link...
[   83.401211] xhci_hcd 0000:00:14.0: Enable port 3
[   83.401250] xhci_hcd 0000:00:14.0: get port status, actual port 3 status  = 0x2a0
[   83.401253] xhci_hcd 0000:00:14.0: Get port status returned 0x2a0
[   83.401276] usb 2-4: Waited 0ms for !LS_SS_DISABLED, status == 0
[   83.401279] usb 2-4: finish reset-resume
[   83.410404] xhci_hcd 0000:00:14.0: Port Status Change Event for port 19
[   83.410414] xhci_hcd 0000:00:14.0: handle_port_status: starting port polling.
[   83.410436] hub 2-0:1.0: state 7 ports 6 chg 0000 evt 0010
[   88.869292] xhci_hcd 0000:00:14.0: Cancel URB 000000005abf9045, dev 4, ep 0x0, starting at offset 0x459cf0cf0
[   88.869299] xhci_hcd 0000:00:14.0: // Ding dong!
[   88.869315] xhci_hcd 0000:00:14.0: Stopped on Transfer TRB for slot 2 ep 0
[   88.869322] xhci_hcd 0000:00:14.0: Removing canceled TD starting at 0x459cf0cf0 (dma).
[   88.869326] xhci_hcd 0000:00:14.0: Finding endpoint context
[   88.869328] xhci_hcd 0000:00:14.0: Cycle state = 0x1
[   88.869330] xhci_hcd 0000:00:14.0: New dequeue segment = 000000003b52395e (virtual)
[   88.869333] xhci_hcd 0000:00:14.0: New dequeue pointer = 0x459cf0d10 (DMA)
[   88.869336] xhci_hcd 0000:00:14.0: Set TR Deq Ptr cmd, new deq seg = 000000003b52395e (0x459cf0000 dma), new deq ptr = 000000000e061ffb (0x459cf0d10 dma), new cycle = 1
[   88.869340] xhci_hcd 0000:00:14.0: // Ding dong!
[   88.869358] usb 2-4: autosuspend-loo timed out on ep0out len=0/0
[   88.869359] xhci_hcd 0000:00:14.0: Successful Set TR Deq Ptr cmd, deq = @459cf0d10
[   88.869365] usb 2-4: Disable of device-initiated U1 failed.
[   93.988482] xhci_hcd 0000:00:14.0: Cancel URB 0000000050ae4319, dev 4, ep 0x0, starting at offset 0x459cf0d10
[   93.988489] xhci_hcd 0000:00:14.0: // Ding dong!
[   93.988523] xhci_hcd 0000:00:14.0: Stopped on Transfer TRB for slot 2 ep 0
[   93.988530] xhci_hcd 0000:00:14.0: Removing canceled TD starting at 0x459cf0d10 (dma).
[   93.988533] xhci_hcd 0000:00:14.0: Finding endpoint context
[   93.988535] xhci_hcd 0000:00:14.0: Cycle state = 0x1
[   93.988538] xhci_hcd 0000:00:14.0: New dequeue segment = 000000003b52395e (virtual)
[   93.988540] xhci_hcd 0000:00:14.0: New dequeue pointer = 0x459cf0d30 (DMA)
[   93.988543] xhci_hcd 0000:00:14.0: Set TR Deq Ptr cmd, new deq seg = 000000003b52395e (0x459cf0000 dma), new deq ptr = 0000000095c1d9f2 (0x459cf0d30 dma), new cycle = 1
[   93.988546] xhci_hcd 0000:00:14.0: // Ding dong!
[   93.988561] xhci_hcd 0000:00:14.0: Successful Set TR Deq Ptr cmd, deq = @459cf0d30
[   93.988569] usb 2-4: autosuspend-loo timed out on ep0out len=0/0
[   93.988572] usb 2-4: Disable of device-initiated U2 failed.
[   93.988575] xhci_hcd 0000:00:14.0: Set up evaluate context for LPM MEL change.
[   93.988579] xhci_hcd 0000:00:14.0: // Ding dong!
[   93.988600] xhci_hcd 0000:00:14.0: Successful evaluate context command
[   93.988608] xhci_hcd 0000:00:14.0: get port status, actual port 3 status  = 0x21203
[   93.988609] xhci_hcd 0000:00:14.0: Get port status returned 0x10203
[   93.988634] xhci_hcd 0000:00:14.0: set port reset, actual port 3 status  = 0x21311
[   94.050490] xhci_hcd 0000:00:14.0: get port status, actual port 3 status  = 0x221203
[   94.050492] xhci_hcd 0000:00:14.0: Get port status returned 0x110203
[   94.050522] xhci_hcd 0000:00:14.0: clear port reset change, actual port 3 status  = 0x21203
[   94.050544] xhci_hcd 0000:00:14.0: clear port warm(BH) reset change, actual port 3 status  = 0x21203
[   94.050566] xhci_hcd 0000:00:14.0: clear port link state change, actual port 3 status  = 0x21203
[   94.050584] xhci_hcd 0000:00:14.0: clear port connect change, actual port 3 status  = 0x1203
[   94.050597] xhci_hcd 0000:00:14.0: get port status, actual port 3 status  = 0x1203
[   94.050599] xhci_hcd 0000:00:14.0: Get port status returned 0x203
[   94.103486] xhci_hcd 0000:00:14.0: Resetting device with slot ID 2
[   94.103492] xhci_hcd 0000:00:14.0: // Ding dong!
[   94.103629] xhci_hcd 0000:00:14.0: Completed reset device command.
[   94.103641] xhci_hcd 0000:00:14.0: Successful reset device command.
[   94.103683] xhci_hcd 0000:00:14.0: // Ding dong!
[   94.285475] xhci_hcd 0000:00:14.0: xhci_hub_status_data: stopping port polling.
[   99.109726] xhci_hcd 0000:00:14.0: Command timeout
[   99.109729] xhci_hcd 0000:00:14.0: Abort command ring
[   99.109758] xhci_hcd 0000:00:14.0: Timeout while waiting for setup device command
[   99.316718] xhci_hcd 0000:00:14.0: // Ding dong!
[  104.740921] xhci_hcd 0000:00:14.0: Command timeout
[  104.740927] xhci_hcd 0000:00:14.0: Abort command ring
[  104.740974] xhci_hcd 0000:00:14.0: Timeout while waiting for setup device command
[  104.948910] usb 2-4: device not accepting address 2, error -62
```
