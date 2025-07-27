# ELAN 04f3:0c4c communication PoC

This repository hosts a PoC implementing the communication for the fingerprint sensor found in
the 2020 HP Spectre x360 14".

The protocol so far seems to be very similar to that of 04f4:0c7e, already supported by
libfprint: [`elanmoc.c`](https://gitlab.freedesktop.org/libfprint/libfprint/-/blob/master/libfprint/drivers/elanmoc/elanmoc.c)

However there seem to be some differences.

A merge request has been opened to add support for this sensor to libfprint:
https://gitlab.freedesktop.org/libfprint/libfprint/-/merge_requests/330

## Dissection parser

`wireshark_parser.py` parses a WireShark dissection of the communication with the sensor,
output as JSON.

In case there are any differences between platforms, since I was not able to make the sensor
run in a VM, I collected all captures [from Windows with USBPcap](https://wiki.wireshark.org/CaptureSetup/USB#Windows).

To output the dissection, first make sure there are no `URB_BULK` packets that are reported as
"malformed". Disable all incriminated dissector as needed from `Analyze` > `Enabled Protocols...`.
`IPPUSB` seems to be the culprit.

Then, after making sure **all** packets referring to the device are **not** filtered, export
the JSON from `File` > `Export Packet Dissections` > `As JSON...`.

## Wireshark dissector

A simple WireShark dissector is also provided. It's very basic.

It can be dropped into the Lua plugins directory, then you can right-click a packet and select
"Decode as..." > ELANMOC.

## PoC

You can use [`uv`](https://docs.astral.sh/uv/) to run the PoC:

```bash
git clone https://github.com/depau/elanpoc.git
cd elanpoc
uv run elanfp --help
```

```
ELAN 04F3:0C4C Match-on-Chip fingerprint reader driver PoC.

Usage:
    elanfp -h | --help
    elanfp info
    elanfp soft_reset
    elanfp hard_reset
    elanfp finger_info <id>
    elanfp verify
    elanfp enrolled_count
    elanfp enroll (-u UD)
    elanfp delete <id>
    elanfp finger_info_all
    elanfp delete_all
    elanfp wipe_all
    elanfp fw_ver
    elanfp capture <png>
    elanfp read_reg <reg>
    elanfp dump_regs
    elanfp raw (-e EP) <hex>...

Options:
-h, --help         Show help
-e EP, --ep-in EP  Input endpoint for raw commands
-u UD. --user UD   User data for enroll command

Commands:
info               Get device info
soft_reset         Reset sensor (via libusb)
hard_reset         Reset sensor (via hardware - disconnect and reconnect)
finger_info <id>   Get finger info
verify             Verify finger
enrolled_count     Get number of fingers currently enrolled
enroll             Enroll a new finger
delete <id>        Delete finger
delete_all         Delete all enrolled fingers (one by one)
wipe_all           Wipe all enrolled fingers (using special command)
fw_ver             Get firmware version
capture            Capture image into a PNG file
read_reg <reg>     Read register
dump_regs          Read all registers
raw                Send raw command
```

## License

MIT
