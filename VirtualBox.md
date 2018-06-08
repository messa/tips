VirtualBox
==========

[www.virtualbox.org](https://www.virtualbox.org/)

Network with host, other VMs and access to the internet
-------------------------------------------------------

[VirtualBox User Manual: Virtual networking](https://www.virtualbox.org/manual/ch06.html)

Create VM with default network settings (which is _NAT_), install operating system...

Create _NAT Network_ - you need only one, it can be shared by multiple VMs.

```
$ VBoxManage natnetwork add --netname natnet1 --network "192.168.15.0/24" --enable --dhcp on
```

Enable NAT on the _NAT Network_.

```
$ VBoxManage natnetwork start --netname natnet1
```

Configure the VM you have created to use this _NAT Network_.

Now you can ping host from VM, VM from host, VM from other VM, and internet from VM.
