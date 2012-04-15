---
title: method_missing, C style
layout: default
excerpt: Doing an analogy of ruby's `method_missing` in C, by extracting identifiers.
---

# `method_missing`, C style

Given: We need to interface to communication in S-expressions. In
C. We are given a library that does the communication and S-expr
representation in some C data structure. We expect commands like

    (get "file" 2500 250)

that is, read from a given file at given position and length, and
return the data.

Now, doing this based on elementary functions like `sexprIsCons()`
and `sexprGetCar()` is pretty unwieldy. It would be much nicer to
simply do

{% highlight c %}
const char *file;
int pos, len;
if (fs_ps_get_SII (sx, file, pos, len)) {
  unsigned char *bp;
  int blen;
  get_data (file, pos, len, &bp, &blen); // TODO: error handling
  return fs_mk_SBy (file, bp, blen);
}
{% endhighlight %}

We'd like to have one function that parses the incoming S-expression
into some variables, and another to compose the reply. (The `fs_`
prefix is used since this is C and its usual method of namespacing.)

Now, we could go along and implement those functions by hand. But,
since we even encoded the exact data types we expect into the
function names, we could just as well generate them. The namespace
prefix and the `_mk_` (make) and `_ps_` (parse) prefixes suffice as a
discriminator against catching other names, and thus we can just
scan the whole source file(s) for these patterns and generate the
according functions. Voilà, auto-generation of missing functions.

Note: The `_ps_` functions are actually macros; otherwise we'd need
to pass in the addresses instead of the variables themselves.

Also, instead of scanning the program for magic identifiers beforehand,
it would be possible to try to compile the program, and then scan the
list of undefined symbols. That would be much closer to `method_missing`
but is also a lot more dependent on compiler and build process.

Exercise for the reader: Also implement command collection so that
all functions named `fs_cmd_get_XXX` are directly put into a command
table. Then a server would need only the command implementations
itself and the global setup code.
