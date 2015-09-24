<!--
.. title: Don't expose the Docker socket (not even to a container)
.. slug: dont-expose-the-docker-socket-not-even-to-a-container
.. date: 2015-09-23 14:54:24 UTC-07:00
.. tags: docker, security
.. category:
.. link:
.. description:
.. type: text
-->

Docker primarily works as a client that communicates with a daemon
process (`dockerd`). Typically that socket is a UNIX domain socket
called `/var/run/docker.sock`. That daemon is highly privileged;
effectively having root access. Any process that can write to the
`dockerd` socket *also* effectively has root access.

This is no big secret. Docker clearly documents this in a bunch of
places, including the introductory documentation. It's an excellent
reason to use Docker Machine for development purposes, even on
Linux. If your regular user can write to the `dockerd` socket, then
every code execution vulnerability comes with a free privilege
escalation.

The warnings around the Docker socket typically come with a (sometimes
implicit) context of being on the host to begin with. Write access to
the socket as an unprivileged user on the host may mean privileged
access to the host, but there seems to be some confusion about what
happens when you get write access to the socket *from a
container*.

The two most common misconceptions seem to be that it either doesn't
grant elevated privileges at all, or that it grants you privileged
access within the container (and without a way to break out). This is
false; write access to the Docker socket is root on the host,
regardless on where that write comes from. This is different from
[Jerome Pettazoni's `dind`][dind], which gives you Docker-in-Docker;
we're talking about access to the host's Docker socket.

The process works like this:

1. The Docker container gets a `docker` client of its own, pointed at
   the `/var/run/docker.sock`.
2. The Docker container launches a new container mounting `/` on
   `/host`. This is the *host* root filesystem, not the first
   container.
3. The second container chroots to `/host`, and is now effectively
   root on the host. (There are a few differences between this and a
   clean login shell; for example, `/proc/self/cgroups` will still show
   Docker cgroups. However, the attacker has all of the permissions
   necessary to work around this.)

This is identical to the process you'd use to escalate from outside of
a container. Write access to the Docker socket is root on the host,
full stop; who's writing, or where they're writing from, doesn't
matter.

Unfortunately, there are plenty of development teams unaware of this
property. I recently came across one, and ended up making a screencast
to unambiguously demonstrate the flaw in their setup (which involved a
container with write access to the Docker socket). You can see the

<iframe width="560" height="315" src="https://www.youtube.com/embed/CB9Aa6QeRaI" frameborder="0" allowfullscreen></iframe>

This isn't new; it's been a known property of the way Docker works
ever since the (unfortunately trivially cross-site scriptable) REST
API listening on a local TCP port was replaced with the
`/var/run/docker.sock` UNIX domain socket.

[dind]: https://github.com/jpetazzo/dind
[ditka]: https://en.wikipedia.org/wiki/Mike_Ditka
