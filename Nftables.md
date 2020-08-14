nftables
========

- https://netfilter.org/projects/nftables/
- https://wiki.nftables.org/
- https://wiki.nftables.org/wiki-nftables/index.php/Moving_from_iptables_to_nftables
- https://wiki.debian.org/nftables


Installation
------------

Install the `nft` command:

```shell
$ sudo apt install nftables
```

nftables is the default firewall in debian 10


Basic commands
--------------

nft list tables

nft flush ruleset

nft add table nat

nft add chain nat prerouting { type nat hook prerouting priority 0 \; }

nft add chain nat postrouting { type nat hook postrouting priority 0 \; }

nft add rule nat postrouting ip saddr 1.2.3.4/24 oif eth0snat 5.6.7.8 

