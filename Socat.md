socat – multipurpose relay for bidirectional data transfer
==========================================================

Homepage: http://www.dest-unreach.org/socat/

Installation on Debian:

```shell
$ sudo apt install socat
```

Expose localhost-only things on network
---------------------------------------

For example if some service is bound only to the localhost interface:

```
$ sudo netstat -ntlp | grep 5900
tcp        0      0 127.0.0.1:5900          0.0.0.0:*               NASLOUCHÁ  119584/qemu-system- 
```

And you want to make it possible to connect to it from the network your computer is connected to:

```shell
$ socat TCP-LISTEN:5910,reuseaddr,fork TCP4:127.0.0.1:590
```
