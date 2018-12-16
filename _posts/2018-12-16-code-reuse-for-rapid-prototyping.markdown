---
layout: post
title: Code Reuse for Rapid Prototyping
---
**Hyper casual games** are today's hottest topic in mobile game development.
First of all what is an hyper casual game? An hyper casual game is a super simple, easy to play and hard to master, instant play game.

To be successful in the hyper casual game market, you need to be fast at prototyping and creating the minimum viable product. In this post, I will not talk about the design process of MVPs but programming and technical aspects.
The fastest way to create an MVP is writing less code obviously, and to write less code you need to reuse the ones you have already written.

#### Code Reuse
In the software industry everybody, especially managers love the term _code reuse_. But it is always hard to reuse the code, no programmer wants to use others' code since she needs to understand what the code is doing first. And probably she finds it easier to write it from scratch.
So to promote code reuse, you need to write reusable code. How is this possible?

##### Service Locator Pattern
Some functionalities are needed in almost every game. For hyper casual games, the most important one is an advertisement service. Other systems may be audio, store, idle time calculation for idle games, logging, etc.
Some examples
* Advertisement service
* Audio service
* Score service
* Logging service
If you can implement these services robust and easy to use, then the developers won't worry about them, they will use them and it will really impact development speed.
##### Template
Having a prototype template is an important asset for faster prototyping as well. It gives developers a solid entrance point, also helps new developers understand the flow. 
It would be better if it has the services above already implemented. A simple UI layer is also very useful. For hyper casual games, it is a play button, some text fields for score and other simple instructions and maybe a settings button.
