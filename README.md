## Test 0

* => Boot kernel without patches/quirks:
* card reader exists
* => Suspend #1:
* suspend/resume ok
* card reader missing
* => Suspend #2:
* suspend fails
* card reader missing
* => Rebind xhci_hcd
* card reader missing
* => Suspend #3:
* suspend fails
* card reader missing

## Test 1

* => Boot kernel without patches/quirks:
* card reader exists
* => Remove card reader device:
* remove ok
* card reader missing
* => Suspend #1:
* suspend/resume ok
* card reader missing
* => Suspend #2:
* suspend/resume ok
* card reader missing
* => Rebind xhci_hcd
* card reader missing
* => Suspend #3:
* suspend/resume ok
* card reader missing
* => Suspend #3:
* suspend fails
* card reader missing

## Test 2

* => Boot kernel without patches/quirks:
* card reader exists
* => Remove card reader device:
* remove ok
* card reader missing
* => Suspend #1:
* suspend/resume ok
* card reader missing
* => Suspend #2:
* suspend/resume ok
* card reader missing
* => Suspend #3:
* suspend/resume ok
* card reader missing

## Test 3

* => Boot kernel without patches, cmdline == `usbcore.quirks=05ac:8406:b` (USB_QUIRK_RESET_RESUME)
* card reader exists
* => Suspend #1:
* suspend/resume ok
* card reader missing
* => Suspend #2:
* suspend/resume ok (probably because I waited for warm reset cycle to fail)
* card reader missing
* => Suspend #2:
* suspend fails
* card reader missing
