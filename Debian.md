

Debian Notes
============


First login
-----------

`sudo aptitude update` :)

Install basic packages: `sudo aptitude install apt-file bind9-host colordiff dnsutils dstat git htop iotop less mlocate nginx ntp openssl python python3 python3-venv rsync screen strace sudo tree vim`

Hostname
--------

[Debian Wiki: How to change hostname](https://wiki.debian.org/HowTo/ChangeHostname)

Change files: `/etc/hostname`, `/etc/hosts`, `/etc/mailname`, `/etc/resolv.conf`

You can grep for occurrences of the old hostname in entire `/etc` directory:

    grep -Ri <old hostname> /etc


DNS local cache
---------------

For faster DNS queries, install __[pdnsd](https://en.wikipedia.org/wiki/Pdnsd)__: `sudo aptitude install pdnsd`

- [debian-administration.org: Speedup DNS requests with a local cache](https://www.debian-administration.org/article/390/Speedup_DNS_requests_with_a_local_cache)

Configuration file is `/etc/pdnsd.conf`. For most setups I recommend to set:

- set `uptest = none` - server(s) will not be checked whether they are live
- set `preset = on` - it means that the default state of given server is `on`; if it was `off` and `uptest` was `none`, then the server state would never switch to `on` and every query would fail with debug message `No server is marked up and allowed for this domain`.

If there are any problems, you can set `debug = on` and watch `/var/cache/pdnsd.debug`.

If you want to have a "local domain", for example _.ldev_, so _anything.ldev_ resolves to `127.0.0.1`:

    rr {
        name = ldev;
        owner = localhost;
        a = 127.0.0.1;
        soa = localhost, root.localhost, 42, 86400, 900, 86400, 86400;
    }

    rr {
        name = *.ldev;
        a = 127.0.0.1;
    }



Debian versions
---------------

Just for reference... I'm always confused which Debian version has which name :)

Last updated: 2015-10-29

- next
    - __stretch__ (_testing_)
- current
    - __jessie__ (8; _stable_)
        - __8.2__ released 2015-09-05 - [CD .iso](http://cdimage.debian.org/debian-cd/8.2.0/amd64/iso-cd/)
        - 8.1 released 2015-06-06
        - 8.0 released 2015-04-26
            - systemd is default (sysvinit still available), UEFI sopport
            - Linux 3.16.7-ckt9,
              Python 2.7.9 and 3.4.2,
              PHP 5.6.7,
              Apache 2.4.10,
              PostgreSQL 9.4.1,
              GCC 4.9.2
- oldstable
    - __wheezy__ (7; _oldstable_)
        - 7.8 released 2015-01-10
        - 7.0 released 2013-05-04
            - multiarch support, UEFI
            - Linux 3.2,
              Python 2.7.3 and 3.2.3,
              PHP 5.4.4,
              Apache 2.2.22,
              PostgreSQL 9.1,
              GCC 4.7.2
- old :)
    - __squeeze__ (6)
        - 6.0.10 released 2014-07-19
        - 6.0.0 released 2011-02-06
            - GNU/kFreeBSD alternative, KDE Plasma Desktop
            - Linux 2.6.32,
              Python 2.6.6, 2.5.5 and 3.1.3,
              PHP 5.3.3,
              Apache 2.2.16,
              PostgreSQL 8.4.6,
              GCC 4.4.5
    - __lenny__ (5)
        - 5.0.10 released 2012-03-10
        - 5.0.0 released 2009-02-14
        - Linux 2.6.26,
          Python 2.5.2 and 2.4.6,
          PHP 5.2.6,
          Apache 2.2.9,
          PostgreSQL 8.3.6,
          GCC 4.3.2
    - __etch__ (4)
    - __sarge__ (3.1)
    - __woody__ (3.0)
    - __potato__ (2.2)
    - __slink__ (2.1)
    - __hamm__ (2.0)




