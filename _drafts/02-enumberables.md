---
layout: post
title:  "Ruby's Enumberable module is your best friend"
date:   2015-02-14 21:35:07
summary: "These handy mixin methods will help you when #each falls short"
---
As I move into the latter half of my time at Turing, I find myself writing more
and more Rails and less and less Ruby (ignoring the obvious fact that Rails is
written in Ruby). I jump at the opportunity to extract
logic out into a "regular old Ruby class", but from time to time I feel like my
Ruby chops are slipping.

This is more of a selfish post to get me back to writing
Ruby. One of my favorite things about programming is solving challenges like
[Exercisms](http://exercism.io). If you haven't seen these yet, I highly
recommend you start using them to sharpen your skills in your language of choice
as well as Test Driven Development. Exercisms have pushed me to the edge of my
Ruby knowledge and helped me to learn some handy tools in the Ruby toolkit. One
group of such tools is the [Enumberable
module](http://ruby-doc.org/core-2.2.0/Enumerable.html).

Enumerables are a collection of methods included with Ruby out of the box.
