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


Overview
--------

table? chain? rule? hook?


Basic commands
--------------

nft list tables

nft flush ruleset

nft add table nat

nft add chain nat prerouting { type nat hook prerouting priority 0 \; }

nft add chain nat postrouting { type nat hook postrouting priority 0 \; }

nft add rule nat postrouting ip saddr 1.2.3.4/24 oif eth0 snat 5.6.7.8 

nft add rule inet filter input protocol vmap { 22 : accept, 23 : drop, 25 : jump mailing }

nft add rule ip filter prerouting dnat set tcp dport map { 80 : 192.168.1.101, 8888 : 182,168.1.102 }


Sources
-------

https://youtu.be/BLh29Gz9Sac

