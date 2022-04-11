
Debian Notes
============


First login
-----------

`sudo apt update` :)

Install basic packages - these are my favorite: `sudo apt install apt-file bind9-host colordiff dnsutils dstat git htop iotop less mlocate nginx ntp openssl python python3 python3-venv rsync screen strace sudo tree vim`


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


/tmp on tmpfs
-------------

```shell
echo 'tmpfs /tmp tmpfs noatime,nosuid,nodev,noexec,mode=1777 0 0' | sudo tee -a /etc/fstab
```

Alternative solution – I'm not sure whether it works on newest Debian versions:

```shell
$ cd /etc/systemd/system
$ sudo ln  -s /usr/share/systemd/tmp.mount .
$ sudo systemctl enable tmp.mount
```
