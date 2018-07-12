
Tips for SSH
============

SSH tunnels
-----------

Example: `ssh -S none -N -C -R 8000:127.0.0.1:80 server.example.com`

- `-S none` - do not use shared socket (see multiplexing below)
- `-N` - do not run any command, just do tunneling
- `-C` - compression
- `-R 8000:127.0.0.1:80` - any connections to TCP socket port 8000 at server.example.com will be forwarded to 127.0.0.1:80 (the machine you are running this command on)


ProxyJump
---------

Imagine you want to connect to guestvm.example.com that is accessible only from hostserver.example.com - you could do it this way:

```
ssh guestvm.example.com ssh hostserver.example.com
```

But this is inflexible. Better alternative is to use jump host option>

```
ssh -J guestvm.example.com hostserver.example.com
```

Or you can configure it in SSH config and use just shortcut:

```
# ~/.ssh/config
Host guestvm
    Hostname guestvm.example.com
    ProxyJump hostserver.example.com
```

```
ssh guestvm
```





SSH multiplexing
----------------

SSH can keep connections to remote servers open in the background and multiplex your SSH sessions into it.
It works also for rsync or **git** (when it uses SSH), so git fetches, pulls and pushes are faster.

Add this to `~/.ssh/config`:

    Host *
    ControlMaster auto
    ControlPath ~/.ssh/cm/%r@%h:%p
    ControlPersist 10m

Create the control socket directory:

    mkdir ~/.ssh/cm

But it's better to create a separate connection for high-volume transfers -
control master for a given connection can be disabled using the switch `-S none`.

Read more about SSH multiplexing here:

- [SSH ControlMaster: The Good, The Bad, The Ugly (anchor.com.au)](http://www.anchor.com.au/blog/2010/02/ssh-controlmaster-the-good-the-bad-the-ugly/)


