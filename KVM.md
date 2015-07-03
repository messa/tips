
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


Networking
----------

TBD


Links
-----

- https://wiki.debian.org/KVM
- http://www.linux-kvm.org/page/RunningKVM
- http://wiki.qemu.org/Qemu-doc.html
