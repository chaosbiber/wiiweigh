# wiiweigh
Use your Wii balance board as a scale, working with auto-reconnect on a raspberry pi 3, using bluez 5, xwiimote and some python.

## about
This is essentially a mashup of existing projects in a try to get a connection and data from the Wii Balance Board without having to resync (red button) on each connection with newer bluez versions. After the non-recurring syncronization you can connect the board by a single push of its button, step on the board, wait for a stable average weight and step off, the board disconnects and turns of, ready for the next connection.

## credits
- irq0 for his working code using xwiimote to connect to the balance board https://github.com/irq0/wiiscale
- Arch wiki for instructions how to enable auto-reconnect on the Wii controllers using bluez 5+ and bluetoothctl https://wiki.archlinux.org/index.php/XWiimote
- bluez example/test files in https://git.kernel.org/cgit/bluetooth/bluez.git/tree/test, nominally test-adapter, test-device and bluezutils.py
- libs: https://github.com/dvdhrm/xwiimote and https://github.com/dvdhrm/xwiimote-bindings

## main requirements

- hid_wiimote kernel module with xwiimote and xwiimote-bindings
- python-dbus
- bluez 5

## caveats
Adopted and tested on a Raspberry with Python 2. If Python 3 is the default, either the scripts need an update, or the installation process is slightly different.

On a test with Arch Linux and a newer Bluez stack I couldn't sucessfully disconnect the Balance Board from the PC-side. On the latest Raspbian version that's not a problem (yet).

## howto
Install bluez, bluez-firmware and python-bluez.
I've built xwiimote and bindings from source, because the debian xwiimote package did not harmonize with the bindings-source.

Step-by-step procedure on Raspbian (tested on a Raspberry Pi 2 with Bluetooth dongle, Raspian lite from 04/2018)

```bash
sudo apt-get update && sudo apt-get upgrade
# on raspbian already installed packages:
# sudo apt-get install build-essential bluez
sudo apt-get install python-dbus git autoconf libtool libudev-dev \
                     libncurses5-dev swig python-dev python-numpy
python -V
# should return Python 2.7.x

mkdir src && cd src
git clone https://github.com/dvdhrm/xwiimote.git
git clone https://github.com/dvdhrm/xwiimote-bindings.git
git clone https://github.com/chaosbiber/wiiweigh.git
cd xwiimote
./autogen.sh
make
sudo make install
cd ../xwiimote-bindings
./autogen.sh
make
sudo make install
cd ../wiiweigh

sudo gpasswd -a pi bluetooth # add user pi to bluetooth group
# reboot or the wiiweight script will throw an exception
# when trying to disconnect as normal user
sudo bluetoothctl
# continue with bluetooth setup below
```

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
scan off
exit
```
(From https://wiki.archlinux.org/index.php/XWiimote)
exit with `exit` or Ctrl+D

Manual disconnect (if needed during tests) with
```
echo "disconnect <MAC of the wiimote>" | bluetoothctl
```
This might throw warnings, but works. Alternatively remove the batteries.