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

Host is the computer where VirtualBox is running.


### NAT

The default network settings is **NAT**.
That enables the VM to access the internet.
But you cannot connect to that VM from your host.

So, for example if you want to access the VM via SSH from your terminal rather than use the VirtualBox window,
you cannot do it via **NAT** network.

But the **NAT** network is good enough for installation of the OS in the VM.


### Bridged

What about the **Bridged** network? It looks perfect, it enables all possible scenarios!

What **Bridged** does is it connects the VM directly tou your internet connection as if it was another computer sitting next to yours.
So other computers have the same connectivity to the VM as you do.

But I don't like to have the VM so much exposed in public or school networks.


### Best strategy: NAT (or NAT Network) + Host-only

You know, you can create more than one network interface in your VM settings :)

**Just add second interface - Host-only**.

<img alt='screenshot' src='https://s3-eu-west-1.amazonaws.com/messa-shared-files/2018/07/virtualbox_hostonly_second_interface.png' width='75%'>

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


### NAT Network

If you have multiple VM and they need to be interconnected, switch also first interface from NAT to NAT Network.

If you do this first time you also need to create NAT Network first - you need only one, it can be shared by multiple VMs.

```
$ VBoxManage natnetwork add --netname natnet1 --network "192.168.15.0/24" --enable --dhcp on
```

Enable NAT on the _NAT Network_.

```
$ VBoxManage natnetwork start --netname natnet1
```

This _NAT Network_ enables communication between VMs, from VM to internet, and (with port forwarding) from internet to VM.
