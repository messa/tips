
Salt
====

[SaltStack.com](http://saltstack.com/community/)

Installation
------------

### Debian

[docs.saltstack.com - Debian installation](http://docs.saltstack.com/en/latest/topics/installation/debian.html)

    wget -q -O- "http://debian.saltstack.com/debian-salt-team-joehealy.gpg.key" \
        | sudo apt-key add -

    echo deb http://debian.saltstack.com/debian wheezy-saltstack main \
        | sudo tee /etc/apt/sources.list.d/salt.list

    sudo aptitude update
    sudo aptitude install salt-minion salt-master salt-syndic


Hello World!
------------

    $ sudo su -
    # salt-key
    Accepted Keys:
    Denied Keys:
    Unaccepted Keys:
    debvm
    Rejected Keys:
    # salt-key --accept-all
    ...
    # salt '*' test.ping
    debvm:
        True
    # salt debvm cmd.run uname
    debvm:
        Linux

Minion configuration - address of master is stored in file `/etc/salt/minion`.

More examples of execution modules:

    # salt '*' disk.usage
    # salt '*' pkg.install vim
    # salt '*' network.interfaces
    # salt '*' sys.doc

States
------

Save a file as `/srv/salt/basic_pkgs.sls`:

    install_network_packages:
      pkg.installed:
        - pkgs:
          - rsync
          - lftp
          - curl

Now you can run it:

    salt '*' state.apply basic_pkgs

(That `.sls` suffix means Salt State)

Top file
--------

Create file `/srv/salt/top.sls`:

    base:
      '*':
        - basic_pkgs

Now you can run:

    # salt '*' state.apply

That `base:` thing is an environment.
More environments can be defined, for example `dev`, `qa`, or `prod`.
More about this in
[the top file documentation](http://docs.saltstack.com/en/latest/ref/states/top.html).

Links
-----

- http://docs.saltstack.com/en/latest/
- http://docs.saltstack.com/en/latest/topics/tutorials/walkthrough.html

Czech articles about Salt:

- http://www.zdrojak.cz/clanky/saltstack-sul-pro-vase-servery/ (2013)
