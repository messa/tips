
libvirt
=======

How to create VM
----------------

Example:

    lvcreate --size 20G --name lv_vm_example_01 vg1

    virt-install \
        --connect qemu:///system \
        --name example-01 \
        --ram 2048 \
        --vcpus 1 \
        --location http://http.debian.net/debian/dists/stable/main/installer-amd64/ \
        --file /dev/mapper/vg1-lv_vm_example_01 \
        --virt-type kvm \
        --os-type linux --os-variant debianwheezy \
        --network default \
        --graphics vnc,port=5901,listen=127.0.0.1,password=xxx \
        --noautoconsole

    virsh autostart example-01

    virt-viewer -c qemu:///system example-01
    virt-viewer -c qemu+ssh://root@example.com/system example-01



