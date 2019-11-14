Debian - package pinning
========================


Set default release
-------------------

Save to `/etc/apt/apt.conf.d/00release`:

```
APT::Default-Release "stable";
```


Package pinning
---------------

Create a file in `/etc/apt/preferences.d/`:

```
Package: balik
Pin: version 0.1.3*
Pin-Priority: 990
```


Links
-----

- https://cs.wikibooks.org/wiki/Apt
