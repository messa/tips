
Tips for SSH
============

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

- [SSH ControlMaster: The Good, The Bad, The Ugly (anchor.com.au)]
  (http://www.anchor.com.au/blog/2010/02/ssh-controlmaster-the-good-the-bad-the-ugly/)


