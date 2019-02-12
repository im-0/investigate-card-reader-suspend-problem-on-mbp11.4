# Problem

There is a known problem with suspend and resume on MacBook Pro Retina
mid-2015 (`MacBookPro11,4`) under Linux: internal SD Card Reader (USB,
05ac:8406) disappears after first suspend/resume cycle, and then all
subsequent tries to suspend are failing.

Links:

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

## Test 0

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

## Test 1

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

## Test 2

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

## Test 3

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

## Test 4

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

## Test 5

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

## Test 6

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

## Test 7

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

## Test 8

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

## Test 9

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
