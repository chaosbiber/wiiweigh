# wiiweigh
Use your Wii balance board as a scale, working with auto-reconnect on a raspberry pi 3, using bluez 5, xwiimote and some python.

## about
This is essentially a mashup of existing projects in a try to get a connection and data from the Wii Balance Board without having to resync (red button) on each connection with newer bluez versions. After the non-recurring syncronization you can connect the board by a single push of its button, step on the board, wait for a stable average weight and step off, the board disconnects and turns of, ready for the next connection.

## credits
- irq0 for his working code using xwiimote to connect to the balance board https://github.com/irq0/wiiscale
- Arch wiki for instructions how to enable auto-reconnect on the Wii controllers using bluez 5+ and bluetoothctl https://wiki.archlinux.org/index.php/XWiimote
- bluez example/test files in https://git.kernel.org/cgit/bluetooth/bluez.git/tree/test, nominally test-adapter, test-device and bluezutils.py
- libs: https://github.com/dvdhrm/xwiimote and https://github.com/dvdhrm/xwiimote-bindings

## requirements

- hid_wiimote kernel module with xwiimote and xwiimote-bindings
- bus-python
- bluez 5

## howto
Install bluez, bluez-firmware and python-bluez.
I've built xwiimote and bindings from source and installed both to /usr/local, because the debian xwiimote package did not harmonize with the bindings-source.

The following procedure should only be required once, the sync survives host reboots. Just try if after the disconnect command you can connect by simply pushing the front button of the board, if not, delete the pairing and try again. The hid_wiimote module lights the blue led of the button when loaded, when it's not the led keeps blinking.

Start `bluetoothctl`
```
power on
agent on
<press red sync button>
scan on
pair <MAC of the found wiimote, use TAB for autocompletion>
# note: we do not explicitly connect, we just pair!
connect <MAC of the wiimote>
# there seems to be a pretty short timeout, so execute this immediately after the pairing command

trust <MAC of the wiimote>
disconnect <MAC of the wiimote>
```
(From https://wiki.archlinux.org/index.php/XWiimote)
exit with `exit` or Ctrl+D

Manual disconnect (if needed during tests) with
```
echo "disconnect <MAC of the wiimote>" | bluetoothctl
```
This might throw warnings, but works. Alternatively remove the batteries.