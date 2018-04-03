---
layout: post
title: Struct vs. Class in .NET
---
## Some markdown tests after a long time without writin anything.

Lately, I've made a very important mistake when using structs in our persistency system(save/load game).
So I wanted to write this post, which may help to others especially Unity developers. Since they mostly use .NET.

In their games.

So what is the difference between a _struct_ and a _class_.

Everybody should and probably do know by heart that; struct is *value type* and class is a *reference type*.
The things is most people do not what this means and what are the effects.
I am going to give some real-life examples to explain these effects.


