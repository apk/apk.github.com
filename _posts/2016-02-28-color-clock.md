---
title: color clock
layout: post
excerpt: A color clock.
---

# color clock

A <a href="//apk.li/c">clock</a>. Instead of using numeric
digits, this one uses the standard color-to-number association
used in electronics, for
<a href="https://en.wikipedia.org/wiki/Electronic_color_code#Resistor_color-coding">resistor
labeling</a> or ribbon cable coloring.

<table id='clockframe'
       style='margin-left: auto; margin-right: auto; border: 1px solid #080;
              padding: 1em; width: 20em; height: 5em; border-spacing: 0px; '><tr>
  <td id="ht"></td>
  <td id="ho"></td>
  <td id="mt"></td>
  <td id="mo"></td>
  <td id="st"></td>
  <td id="so"></td>
</tr></table>

The one interesting thing is what many clocks that display seconds
get wrong: They just update about once in a second by they don't
even try to do the update on the second. Effect: multiple instances
drift with respect to each other (and to the actual seconds),
and occasionally wil do a two-second step (when slow) or not
step in a second (if fast, unusual).

The fix:
{% highlight javascript %}
  var d=new Date();
  var m=1050-d.getMilliseconds();
  if (m < 500) m += 1000;
  window.setTimeout(tick,m);
{% endhighlight %}
Compute the time to the next full second, and do a delay to that
point, instead of just `setTimeout(1000)`.

The background color around the 'digits' (and the entire background
on the original <a href="//apk.li/c">page</a> - fullscreen and enjoy)
is set as it is on
<a href="http://whatcolourisit.scn9a.org/">whatcolourisit.scn9a.org</a>
which inspired me to this clock in the first place. That page also displays
the second drift - just open both pages side by side to see that.

<script type="text/javascript" src="//apk.li/c.js"></script>

