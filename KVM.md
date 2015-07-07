
KVM
===

KVM is virtualization tool for Linux, built on top of Qemu.
Or something like that.
Anyway, it uses hardware virtualiztion (Intel VT, AMD-V).

CPU support
-----------

How can I tell if I have Intel VT or AMD-V?

    egrep '^flags.*(vmx|svm)' /proc/cpuinfo

If something shows up, you have VT.

Prepare disk
------------

You can just create some big file (`dd if=/dev/zero of=disk.img bs=1M count=10240`).
Another option is qcow2 file format:

    qemu-img create -f qcow2 disk.img 10G

Run
---

It's simple:

    kvm -m 2048 -cdrom some.iso -hda disk.img

Parameter `-m 2048` means 2 GB of RAM.

It's not necessary to run kvm under sudo, but you have to be able to use device `/dev/kvm` - its group should be `kvm`, so just add yourself to group `kvm`.

__How to release mouse:__ press both Ctrl Alt :)

### Faster disk writes

Only for non-important data.

    kvm -drive disk.img,cache=unsafe


Networking
----------

By default the virtual machine is connected to some user mode internal network. It's easy to use (no setup needed), VM can reach outside network, but nobody can connect to the VM.

My favorite setup is __public virtual bridge__ - VM connects to the same network as your computer is connected and acts like another computer there. (One problem: it doesn't work with most wireless drivers.)

    sudo apt install iproute2 bridge-utils

__Setup `br0`__ in `/etc/network/interfaces` (this is for Debian):

    # Replace old eth0 config with br0
    auto eth0 br0
    # Use old eth0 config for br0, plus bridge stuff
    iface br0 inet dhcp
        bridge_ports    eth0
        bridge_stp      off
        bridge_maxwait  0
        bridge_fd       0

Run `/etc/init.d/networking restart`.
Of course do not do this over SSH without physical access to the computer so you are not going to be locked out if anything goes wrong.

__Add `tap0`:__

    sudo ip tuntap add dev tap0 mode tap user $USER
    sudo ip link set dev tap0 up
    sudo brctl addif br0 tap0

_Alternative:_
instead of adding `ta0` to `br0` manually,
it can be included in `/etc/network/interfaces` too.
See
[Debian QEMU wiki](https://wiki.debian.org/QEMU#Host_and_guests_on_same_network).

__Run kvm:__

    kvm  -file ...etc. \
        -net nic,macaddr=de:ad:be:ef:11:22 \
        -net tap,ifname=tap0,script=no,downscript=no

If you run multiple kvm instances at once, be sure to assign a different macaddr to each one.


Links
-----

- https://wiki.debian.org/KVM
- http://www.linux-kvm.org/page/RunningKVM
- http://www.linux-kvm.org/page/Networking
- http://wiki.qemu.org/Qemu-doc.html
