
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

Minion configuration how to connect to master:




Links
-----
