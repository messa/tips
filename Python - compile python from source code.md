Compile Python from source code
===============================

For Debian. Tested with Python 3.8.3 on Debian 10.

https://www.python.org/downloads/source/

```shell
$ sudo apt-get build-dep python3.7  # or whatever is the default version in your system
```

Download and unzip source tarball. Enter the directory where the tarball was extracted to.

```shell
$ ./configure --prefix /opt/python3.8.3 --enable-optimizations
$ make
$ make install
```


Links
-----

https://devguide.python.org/setup/
