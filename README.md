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
