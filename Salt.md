
Salt
====

[SaltStack.com](http://saltstack.com/community/)

This text focuses on version 2015.5.2.


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
    # salt '*' sys.list_modules
    # salt '*' sys.list_functions
    # salt '*' sys.list_runner_functions
    # salt '*' sys.list_state_functions
    # salt '*' pillar.items

Salt states, modules... what is what?

- __State modules__  are the components that map to actual enforcement and management
  of Salt states.
  Like `pkg` in the example `basic_pkg.sls` below.

- __Execution modules__ are the functions called by the salt command
 (like `disk.usage` above).
 Cannot be called directly within state files.
 You must use the `module` state module to call execution modules within state runs.

- __Runners__ are convenience applications executed with the salt-run command.
  Salt runners work similarly to Salt execution modules however they execute on
  the Salt master itself instead of remote Salt minions.)


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

- [Salt docs](http://docs.saltstack.com/en/latest/)
- [Salt Walkthrough](http://docs.saltstack.com/en/latest/topics/tutorials/walkthrough.html)
- [Full list of builtin execution modules](http://docs.saltstack.com/en/latest/ref/modules/all/index.html)
- [Best Practices](https://docs.saltstack.com/en/latest/topics/best_practices.html)

Czech articles about Salt:

- http://www.zdrojak.cz/clanky/saltstack-sul-pro-vase-servery/ (2013)
