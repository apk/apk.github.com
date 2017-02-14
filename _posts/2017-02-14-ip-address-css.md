---
title: css ip address
layout: post
excerpt: IP address via css.
---

# IP address via css.

<style type='text/css'>@import url("//apk.li/ip.css");</style>

Your <a href="//apk.li/ip">ip address</a>. But I don't want to modify
the entire page on the fly, so instead let's use css. Also, do
the background analogously to the previous
<a href="/2016/02/28/color-clock.html">color clock</a>, this time
using the first three IP(v4) tuples as the RGB values:

<table class='ipbox-bg' id='clockframe'
       style='margin-left: auto; margin-right: auto; border: 1px solid #080;
              padding: 0.5em; font-size:240%; border-spacing: 0px; '><tr>
  <td class="ipbox-text"></td>
</tr></table>
The text is either white or black, depending on an estimate which
gives the better contrast.
