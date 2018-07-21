VirtualBox
==========

[www.virtualbox.org](https://www.virtualbox.org/)

Network with host, other VMs and access to the internet
-------------------------------------------------------

[VirtualBox User Manual: Virtual networking](https://www.virtualbox.org/manual/ch06.html)

See table 6.1 in the documentation (link above):

| Mode        | VM ↔ Host | VM1 ↔ VM2 | VM → Internet | Internet → VM  |
|-------------|-----------|-----------|---------------|-----------------|
| Host-only   | ✅        | ✅       |               |                 |
| Internal    |           | ✅       |               |                 |
| Bridged     | ✅        | ✅       | ✅           | ✅              |
| NAT         |           |           | ✅           | Port forwarding |
| NAT Network |           | ✅       | ✅            | Port forwarding |



Create VM with default network settings (which is _NAT_), install operating system...

Create _NAT Network_ - you need only one, it can be shared by multiple VMs.

```
$ VBoxManage natnetwork add --netname natnet1 --network "192.168.15.0/24" --enable --dhcp on
```

Enable NAT on the _NAT Network_.

```
$ VBoxManage natnetwork start --netname natnet1
```

This _NAT Network_ enables communication between VMs, from VM to internet, and (with port forwarding) from internet to VM. This will be the primary interface.

In VM settings enable the **second interface** and set it up as _Host-only network_. This will enable connecting from host to VM.

This second interface must be configured inside VM to retrieve IP address:

```
# /etc/network/interfaces

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug enp0s3
iface enp0s3 inet dhcp

# The secondary network interface
allow-hotplug enp0s8
iface enp0s8 inet dhcp
```


