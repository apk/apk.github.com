---
title: ssh as proxy command
layout: post
excerpt: ssh via ssh
---

# ssh as proxy command

With the `proxy-command` configuration option you can tell ssh
not to connect iself, but instead to use the given command to
connect to the ssh server.

A trivial example is

    ssh -o "ProxyCommand nc --proxy 127.0.0.1:9150 \
                            --proxy-type socks5 \
                            apkx44pmf7fyd63e.onion 22"

to access onion services, because ssh itself does not allow to use a proxy
for the outgoing connection.

## Hopping

This gets more interesting when the target machine isn't directly
reachable from 'here', and you need to go through another host.
Of course, you can log into the other machine, and ssh on from there,
but ProxyCommand makes it possible to do something different:

    ssh -o 'ProxyCommand ssh other nc target 22'

i.e. we tell ssh to start another ssh to execute a netcat on the
intermediate machine. That will connect to the target ssh server,
and the data stream of the outer ssh is carried through the inner
ssh and the netcat.

Now, when you set up a `.ssh/config` entry for the target machine
and place the `ProxyCommand` there, you can just do a 'ssh target'
to get there (although you will be asked for two passwords/phrases
unless you use `ssh-agent`).

This now also means that you can do `scp file target:.` which will
go directly there. No more need to copy a file to the intermediate
host first.

## Reverse-hopping

This can also work in the other direction. The target host might
not be reachable at all from the outside (e.g. behind a NAT). Then,
instead, let the target host run

    ssh jumphost -R 6622:localhost:22

that, the sshd daemon on the jumphost will open a listener socket on
port 6622, and anything coming in there will be forwarded to the
ssh daemon on the target host. (Since these listener sockets are only
bound to localhost, only people able to get onto the jumphost are
able to access the target.)

Now you, on the other end, can do

    ssh -o 'ProxyCommand ssh jumphost nc localhost 6622'

and in that way your ssh accesses the 6622 port on the jumphost
locally, and via that the ssh daemon on the target host. Connection
established, and again you can `scp` as well, do X forwarding etc.

This obviously has security implications. First: Anybody with an account
on both the target and the jumphost can use that setup. Keep in mind
that the target's security (password choices etc.)  are set up under the
assumpion that the machine isn't externally reachable, and running the
`ssh -R` violates that assumption.  Depending on the security policy
and the jurisdiction you're in this easily can get you fired.

It is also a bit of work to set up the `ssh -R` so that it runs reliably,
actually restarts when necessary ('half-open TCP connections'), and that
the authentication used for automatic reconnect cannot be used for doing
anything else on the jumphost.
