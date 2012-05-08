---
title: the hardest bug
layout: post
excerpt: Testing a file system and finding a completely unrelated bug.
---

# the hardest bug

I once wrote a file system. Not the kind with hierarchical directories,
but a simple, special-purpose one. It just stores a set of
`id:int, meta:string, data:blob` tuples; file creating returns an id
instead of being given a name or id, and the data can be written
contiguously and read in any sequence. The internal design is another
story to be told another time. It used a log-style structure to avoid
the `fsck` times at startup, as is semi-customary these days. The point
of making our own was that there was no file system big enough or fast
enough available for the realtime platform we needed for other reasons.

Now, for the question whether it actually works I deemed sample tests
not being good enough because there are simply too many ways there
could be bugs that only showed under load or after some time of operation,
or after more data was cumulatively written and deleted than the disk can
hold etc. pp.

So I wrote an application that performed the expected operations (serial
writes, slow piecewise serial reads, fast sequential reads, deletes) in
many parallel instances. For one this was the stress test whether it
actually performed according to requirements, and for the other, it was
to test whether the data was actually stored and *correctly* retrieved.
For the latter, specific peudorandom data was written, and the readers
checked that against their expectations, and logged errors when those
weren't met. Some bugs were weeded out, both because of instabilities
and because of minor bugs in the actual block management routines. (There
were no previous unit test, not at that time. The file system *was* the unit,
basically.)

But the strange thing was that there were totally random
misoperations/assertion/crashes about once a day. Occasionally
random values appeared in function parameters, as I filtered out
by adding assertions. The strange thing was that the assert would
fire in a function receiving a parameter, but *not* the same assertion
on the values being passed into the function before the invocation.

That lead to some head scratching and to a dive into the assembly
(and of course, given the fact that it took a day or longer to
happen the next time, some wait). The final cause was not in the
file system code, but somewhere unexpected. The questionable parameter
is an eight byte structure (in the firm days of 32 bits), and the
compiler thought it reasonable to load it into a floating point
register to then store it on the stack as function parameter.

The operating system (a realtime system) wasn't told that this is
a floating point task (and it isn't), and thus didn't bother to
save the floating point registers on a task switch, and about once
a day, under full load, a context switch would just happen to occur
between the fp load and the fp store, and the data would get mangled,
and the file system code would crash.

The vendor later acknowledged this as a bug in his tool chain, and
delivered a fix. But it sure took some time to find and diagnose.
