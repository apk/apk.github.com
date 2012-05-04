---
title: strange unit testing
layout: default
excerpt: Unit testing legacy code.
---

# strange unit testing

Recently I had to extend a load balancing algorithm to take into
account more variable (which they were isn't pertinent here).
Unfortunately the actual code to determine a candidate is in the
middle of a long function, and that function can hardly be exercised
without running the whole program *and* its communication relations.

I wasn't inclined to extract that part into a separate function, because
it would also needed to be in a separate file, and also I didn't want
to risk breaking anything in that step (and also I don't think that
testability should greatly affect program design).

So instead I went placing comments like

    /*# BEGIN test1 */
    /*# END test1 */

around the pieces of code that I needed to exercise and later modify,
and wrote a ruby script to extract exactly those pieces into a file
as named in the comment. The test code then includes those files
(one each) after setting up the necessary scaffolding (including
the variables used from other parts of that big function); the result
then is compiled and run to perform the tests. (Part of the testing
was to simulate the disk usage distribution caused by the load balancing
by simulating everything for some time. Doing load balancing by a number
of weighed factors can have counterintuitive results.)
