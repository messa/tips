
Dockerfile
==========

### Useful or interesting snippets

    FROM debian:jessie
    ENV DEBIAN_FRONTEND noninteractive
    RUN apt-get update && \
        apt-get install -y --no-install-recommends \
	        build-essential \
	        ca-certificates \
	        python3 \
	        python3-dev \
	        vim \
	        ...

Cleanup for smaller images:

    RUN apt-get ... && \
        apt-get clean && \
        rm -rf /var/lib/apt/lists/*

For more inspiration, see official Dockerfiles at [github.com/docker-library](https://github.com/docker-library).


### Private NPM repository

If you use private NPM repository (for example [Sinopia](https://www.npmjs.com/package/sinopia)) and want to run `npm install` as part of `docker build`:

First, prepare `npmrc` file:

    egrep '^//npm.example.com/:_authToken=' ~/.npmrc > npmrc
    echo 'registry=https://npm.example.com' >> npmrc

`Dockerfile` snippet:

    COPY npmrc /root/.npmrc
    WORKDIR my-web-app
    RUN npm install --unsafe-perm


### Signals

Proper handling of signals because of init etc. so Ctr-C and `docker stop` works - download [this signal_wrapper.py](https://github.com/messa/este-docker/blob/master/signal_wrapper.py) and run commands this way:

    ADD signal_wrapper.py /
    CMD ["/signal_wrapper.py", "gulp", "-p"]

This is my "invention" - I couldn't find anything like this (just some hints that this is the way). But it is possible that there is more correct way how to deal with signals.
