# Clemote - IR Remote Control for Clementine Player

<img align="right" width="40%" src="./assets/remote-labels.jpg">

This app reads button presses from a remote control, sends commands to the Clementine Player, deletes and renames files, and updates song ratings.

The app controls Clementine via D-Bus. File and tag operations are performed directly, after which the Clementine library is synchronized in order to make the changes visible in the tracklists.

**Warning: The RECORD button immediately deletes the currently playing file from disk.**

The blue button adds ".delete" to the filename of the currently playing file but does not delete the file.


### Usage

* Start the Clementine Player
* Start this app with:

```
$ clemote remote </dev/input/eventX>
```

Where `/dev/input/eventX` is the receiver for your remote control.

You may have to run the app as root and the device number may change when you plug or unplug devices, or reboot the computer. To fix these issue, see below.

If you need instructions for how to run the app automatically as a service, feel free to add a ticket.


### Finding your remote control receiver device

```
$ sudo apt install input-utils
$ sudo lsinput
```

Look for a device such as

```
/dev/input/event15
   bustype : BUS_USB
   vendor  : 0x1019
   product : 0xf38
   version : 0
   name    : "Media Center Ed. eHome Infrared "
   phys    : "usb-0000:29:00.3-3"
   bits ev : (null) (null) (null) (null)
```

You now have the device (`/dev/input/event15` in this case).


### Creating a persistent device and allow use by regular user

The device number may change when you plug or unplug devices, or reboot the computer. It may also be usable only by root. To fix these issues, add a udev file:

```
$ sudo editor /etc/udev/rules.d/10-clemote.rules
```

With contents:

```
ACTION=="add", ATTRS{idVendor}=="1019", ATTRS{idProduct}=="0f38", SYMLINK+="remote_control", MODE="0777"
```

Replace the `1019` and `0f38` values in this example with the values you found in the `vendor` and `product` sections of the output from `lsinput`. The values must be 4 digits in udev, so add `0`s if necessary. Remove `0x`. E.g.: `0xf38` -> `0f38`.

To activate the new rule, unplug and plug the remote control receiver or reboot the computer.

You should now be able to start the app as a regular user with:

```
$ clemote remote /dev/remote_control
```

### Setup

* Allow the app to trigger refresh in Clementine to show changes:


    Clementine > Tools > Preferences > Music Library
        > Monitor library for changes > Enable


### Troubleshooting

* This app does not need LIRC, and LIRC may interfere if installed. Try:

```
$ sudo apt remove lirc
```

### Technologies

* sdbus-cpp - High level D-Bus C++ bindings
* taglib - Solid, battle tested, audio metadata library


### Build on Linux

Tested on Linux Mint 19.

Should work on Ubuntu 18.04 and other distributions based on it.


Packaged dependencies:

```
$ sudo apt install build-essential cmake \
libboost-filesystem-dev libboost-system-dev \
libsystemd-dev libpulse-dev libevdev-dev
```

sdbus-cpp:

```
$ git clone https://github.com/Kistler-Group/sdbus-cpp.git
$ cd sdbus-cpp
$ ./autogen.sh --disable-tests
$ make -j$(nproc)
```

taglib:

```
$ git clone https://github.com/taglib/taglib.git
$ cd taglib
$ cmake -DCMAKE_BUILD_TYPE=Release .
$ make -j$(nproc)
$ sudo make install
```

Troubleshoot D-Bus issues:

```
$ sudo apt install d-feet
$ d-feet
```
