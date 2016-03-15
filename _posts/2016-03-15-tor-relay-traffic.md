---
title: tor relay traffic
layout: post
excerpt: The shape of tor relay traffic.
---

# tor relay traffic

Some people seem to assume that tor relays push a constant amount
of traffic, but this is far from the truth. Each client individually
decides which relays to use, and the sum is something pretty noisy.

<div align='center'><img src='/images/f-trfl-345600s.png'></div>

This is the traffic on my faster relay (set to 7MB), and the
actual traffic is still ragged, and has some longer-term
excursions. It is new, and just became a guard node.

<div align='center'><img src='/images/q-trfl-345600s.png'></div>

This is the other one, with a traffic limit five times lower,
more that correspondingly less traffic, and because there are
fewer circuits running through it, the traffic is even more
ragged and bursty. This relay also slowly loses traffic;
this seems to be a trend these days for lower-traffic relays.

Note the logarithmic Y scale that makes the fluctuation less
prominent. In linear scale the one peak in this graph would
scale the entire rest down far to the bottom.
