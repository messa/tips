

Docker - Installation on Debian
===============================

Docs: https://docs.docker.com/installation/debian/

    apt-get purge lxc-docker*
    apt-get purge docker.io*
    apt-get install apt-transport-https
    apt-key adv --keyserver hkp://pgp.mit.edu:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
    echo "deb https://apt.dockerproject.org/repo debian-jessie main" > /etc/apt/sources.list.d/docker.list
    apt-get update
    # verify that apt is pulling from the right repository:
    apt-cache policy docker-engine
    sudo apt-get install docker-engine
    sudo service docker start
    # verify docker is installed correctly:
    sudo docker run --rm hello-world
