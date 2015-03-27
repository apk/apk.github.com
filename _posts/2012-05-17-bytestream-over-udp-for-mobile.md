---
title: bytestream over udp for mobile
layout: post
excerpt: Transporting a TCP stream (like ssh) via UDP to deal with mobile flakiness.
---

# bytestream over udp for mobile

TCP is the standard internet protocol to achieve reliable transport
over the stateless and lossy packet transport that the internet IP
layer provides. It achieves this goal by sending the data to be
transported, waiting for the other side to acknowlede that it has been
reveiced, and, if deemed necessary, retransmit data for which it does
not receive an acknowledgement in time.

The algorithms that govern the retransmission are well-honed for the
usual case where all transmissions go over links that are, by
themselves, reliable and where loss and/or delay is caused by
overloaded links, and scarcely by data actually getting lost on a
link.

Mobile IP is different. The actual link capacity varies wildly,
including going to to zero occasionally. This is still working pretty
well when the mobile station is rather stationary. But when you start
moving around like in a train it gets pretty horrible. You have loss
of connectivity every dozen seconds, and you are pretty lucky when
a TCP retransmit just falls into the window of accessibility so that
the connection will resume to work. This becomes obvious when you run
a ping in parallel to an ssh session. The ping will resume much faster
than a ssh connection.

As an additional complication the possible distance of roaming may be
limited. For one, LTE/4G implementations do not always allow handover
to 2G/3G at all, and there also seem to be limitations to the distance
roaming works before the session is terminated. You need to reconnect,
and will get a different IP address then, and this will break existing
TCP connections. This is not that bad with plain HTTP, but web socket
connections will suffer, as will any ssh session.

# Protocol data format

Thus I started to design an implement a simple UDP-based reliable
transport protocol. It is based on numbering data packets and basing
acknowledments on packet number (TCP does number individual bytes),
but the important parts are the retransmission algorithms.

The client first establishes a connection identity with the server,
and this identity is subsequently used by the server to identify the
client. This allows the connection to resume even after a change of
address; the server will simply notice that the addresse the client's
packets come from has changed, and will send its packets to that
address subsequently.

Client and server also regularly send beacon packets to each other,
and will only send data packets when they recently received something
from the other side. This avoids sending data and thus filling the
bloat buffers when the channel isn't open at the moment. While the
client continues indefinitely to send beacons the server only does so
for a short time when not hearing from the client. This avoids sending
packets to a client address when the client has lost its network
session and the address may already have been reassigned to someone
else.

The protocol is quite simple-minded. It has a twelve-byte header
containing three numbers and a flag field:

    3 bytes: Connection identity
    1 byte: Flags
    4 bytes: Acknowledgement number
    4 bytes: Packet number

For data packets, this is followed by the actual data. There is no length
field as this can be deduced from the length of the whole UDP packet.

Only the fields that are needed for a given packet type are included
in the actual UDP packet. There are three kinds of packets: The initial
request, acknowledgement-only packets, and data packets.

The initial request is an empty UDP frame sent from client to server,
and server responds to it with an ack-only packet that contains the
connection identifier assigned to that client, and and acknowledgement
number of zero (as the server hasn't received any data yet).

Acknowledgement-only packets are eight bytes in size, and contain the
session identifier (plus flags) and the acknowledgement number which is
the number of data frames the reveived has got so far. Ack-only packets
are also used as the beacon packets mentioned above.

Data packets also contain the session identifier and the current
acknowledgement number, and in addition the contain the data frame
number and the associated data. Sending an empty data frame indicated
the end of stream in that direction.

# Algorithms

The client initially sends the empty initial packet to the server, and
does so every few seconds until the server responds. From then on the
client knows its session identity and can send acknowledgement or data
packets. The server must take care that duplicate initial packets from
the same source only establish a single session.

Both sides are only allowed to send data packets under two conditions:
It must have received some packets from the other side recently (the
channel is open), and it must only have sent a limited number of
unacknowledged frames before. This limit, the window, is initially
one, and increases up to eight with received acknowledgements.

If either side is not eligible for sending data packets they instead
send ack-only packets. When a new data packet is received, an ack for
that is sent immediately. Otherwise ack packets are sent every few
seconds; the server stops doing so after a minute of not receiving any
packets from the client, which the client continues sending
indefinitely.

Data frame are retransmitted only when after a few seconds
acknowledgements received still do not acknowledge the frame in
question. There is no selective retransmission; the oldest outstanding
frame is retransmitted, and the window is shrunk to one, expecting it
to grow again when transport resumes.

# Considerations

There is no security involved here; it is assumed that the byte stream
that we reliably transport is itself secured, like an ssh or https
connection. It may be interesting to use a simple scrambling algorithm
to make the packet contents look more random, although the first
priority is to get it tuned to work efficiently with fast recovery
from channel outages.

Also it is desirable to put all traffic from a mobile station onto a
single instance of this protocol, to reduce the number of beacons sent
in idle, and to generally reduce the traffic. The most easy way would
be, for instance, use this protocol for a single ssh connection which
port forwards a browser's many proxy connection to a proxy on the ssh
target machine.

The server needs to discard idle sessions after some time, but that
time should be on the order of an hour. It is possible to disconnect a
laptop from the mobile network, suspend it, and later reconnect
(after, for instance, changing trains), and the session will still
work.

Many details are yet missing here.
