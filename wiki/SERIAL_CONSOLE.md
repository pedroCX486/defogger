### Serial Console

Useful for fw greater than v2.02.02. The serial console is used to temporally 
enable the webservice of the camera. Then, the fw can be downloaded using defogging procedure 
and further flash the custom fw.tar firmware.

There is a 4 hole female header with 2 mm spacing in the bottom of the
camera. This header is easily accessible without opening the case at
all. But you will need to remove the bottom label to find it:
![label removed](https://www.mork.no/~bjorn/dcs8000lh/dcs8000lh-label-removed.jpg)

Take a picure of the lable or save the information somewhere else
first, in case you make the it unreadable in the process.

Mate with a 3 (or 4) pin male 2 mm connector, or use sufficiently
solid wires.  The pins need to be 6-10 mm long.  The pins will mess up the QR code, but the rest of the label can be left intact if you're careful:
![header with pins](https://www.mork.no/~bjorn/dcs8000lh/dcs8000lh-label-with-serial-pins.jpg)

The pinout seen from egde to center of camera are:


| 1    | 2  | 3  | 4   |
|------|----|----|-----|
| 3.3V | TX | RX | GND |

And the serial port baudrate is 57600 8N1.


You obviously need a 3.3V TTL adapter for this, look for example
at the generic OpenWrt console instructions if you need guidance.
![USB ttl adapter connected](https://www.mork.no/~bjorn/dcs8000lh/dcs8000lh-serial-connected.jpg)


Do not connect the 3.3V pin.  All USB TTL adapters are powered by the
USB bus.

### Opening the case

Remove the top and bottom parts of the sylinder.  I assume the two
remaning halves of the sylinder are simple held together by clips, but
I did not verify this after discovering the easily accessible console
header.

The top lid is clipped on:
![top lid](https://www.mork.no/~bjorn/dcs8000lh/dcs8000lh-top-lid.jpg)

The bottom cover is held in place by two screws under the label:
![bottom cover](https://www.mork.no/~bjorn/dcs8000lh/dcs8000lh-label-removed.jpg)


Removing the bottom cover reveals the reset button and the console header:
![bottom removed](https://www.mork.no/~bjorn/dcs8000lh/dcs8000lh-bottom-without-cover.jpg)

After having access to a serial console, you may see [U-BOOT](U_BOOT.md) show up, go there
for more info on how to proceed.