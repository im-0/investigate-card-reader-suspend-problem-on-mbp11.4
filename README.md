# Problem

There is a known problem with suspend and resume on MacBook Pro Retina
mid-2015 (`MacBookPro11,4`) under Linux: internal SD Card Reader (USB,
05ac:8406) disappears after first suspend/resume cycle, and then all
subsequent tries to suspend are failing.

Links:

* https://bugzilla.kernel.org/show_bug.cgi?id=111201
* https://bugzilla.kernel.org/show_bug.cgi?id=202509
* https://www.spinics.net/lists/linux-usb/msg164261.html

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
