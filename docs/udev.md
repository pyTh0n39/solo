# tl;dr

Create `/etc/udev/fido.rules` and add the following.

```
# U2F Zero
KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="10c4", ATTRS{idProduct}=="8acf", TAG+="uaccess"
```

Then run

```
udevadm trigger
```

# How do udev rules work and why are they needed

In Linux, `udev` (part of `systemd`, read `man 7 udev`) handles "hot-pluggable" devices, of which Solo and U2F Zero are examples. In particular, it creates nodes in the `/dev` filesystem (in Linux, everything is a file), which allow accessing the device.

By default, for security reasons often only the `root` user can access these nodes, unless they are whitelisted using a so-called "udev rule". So depending on your system setup, such a udev rule may be necessary to allow non-root users access to the device, for instance yourself when using a browser to perform two-factor authentication.

## What does a udev rule do?
It matches events it receives (typically, comparing with the `==` operator), and performs actions (typically, setting attributes of the node with the `=` or `+=` operators).

## What is `hidraw`?
HID are human-interface devices (keyboards, mice, Solo keys), attached via USB. The `hidraw` system gives software direct ("raw") access to the device.

## Which node is my Solo or U2F Zero security key?
You can either compare `ls /dev` before and after inserting, or use the `udevadm` tool, e.g., by running
```
udevadm monitor --environment --udev | grep DEVNAME
```
Typically, you will detect `/dev/hidraw0`.

## How do you know if your system is configured correctly?
Try reading and writing to the device node you identified in the previous step. Assuming the node is called `/dev/hidraw0`:

* read: try `cat /dev/hidraw0`, if you don't get "permission denied", you can access.
* write: try `echo "hello, Solo" > /dev/hidraw0`. Again, if you don't get denied permission, you're OK.

## Which rule should I use, and how do I do it?
Simplest is probably to copy [Yubico's rule file](https://github.com/Yubico/libu2f-host/blob/master/70-u2f.rules) to `/etc/udev/rules.d/fido.rules` on your system. This contains rules for Yubico's keys, the U2F Zero, and many others. The relevant line for U2F Zero is:
```
KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="10c4", ATTRS{idProduct}=="8acf", TAG+="uaccess"
```
It matches on the correct vendor/product IDs of 10c4/8acf, and adds the TAG `uaccess`. Older versions of udev use rules such as 
```
KERNEL=="hidraw*", SUBSYSTEM=="hidraw", ATTRS{idVendor}=="10c4", MODE="0644", GROUP="plugdev"
```
which sets MODE of the device node to readable by anyone.

Now reload the device events.

```
udevadm trigger
```

## What about vendor and product ID for Solo?
Current prototypes reuse the IDs of the U2F Zero (10c4/8acf). The final Solo will probably be assigned new IDs; read about it here first :)

## You got this all wrong, I can't believe it!
Are you suffering from [us being wrong](https://xkcd.com/386/)? Please, send us a [pull request](https://github.com/SoloKeysSec/solo/pulls) and prove us wrong :D
