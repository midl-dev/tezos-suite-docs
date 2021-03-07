# Setup remote signer

Bill of materials:

* one Raspberry Pi 4 with a Geekworm hat
* two batteries
* one LTE dongle
* one USB-to-micro-USB adapter
* one USB-to-USB-C cable

Not included:

* USB power adapter of at least 3A (3,000 mA) supply
* Ethernet cable

## Assembly

Insert the batteries in the Pi Hat. Make sure to respect polarity.

Plug the 4G dongle to a USB 2 port (these are black in color).

Plug the USB-to-micro-USB adapter to a USB 3 port. These are blue in color. **This is important!** The Ledger will be connected to the adapter, and it must be connected to a USB 3 port.

Plug the Ethernet cable to an Internet-connected device.

Plug the USB-to-USB C cable to the USB hat. **Important!** Do not connect the USB-C port to the Raspberry Pi directly, otherwise the battery will not charge.

On the other end, connect the USB cable to the power adapter.

## Visual checks

Check that the battery status LEDs indicate that the device is either charging or fully charged. In steady state, the device should be indicating a full charge. If that is not the case, your adapter may not be powerful enough.

Check that the LED on the 4G LTE dongle shows a solid blue for Huawei (blinking blue for ZTE). If that is not the case, please move your setup to an area of cell phone coverage.

## Connect Ledger

Connect the Baking Ledger Nano S to the Micro-USB end of the adapter. Unlock it and launch the baking app.
