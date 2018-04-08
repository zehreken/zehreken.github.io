---
layout: post
title: Struct vs. Class in .NET
---
Some markdown tests after a long time without writing anything.

Lately, I've made a very important mistake when using structs in our persistency system(save/load game).
So I wanted to write this post, which may help to others especially Unity developers. Since they mostly use .NET.
In their games.
So what is the difference between a _struct_ and a _class_.
Everybody should and probably do know by heart that; struct is *value type* and class is a *reference type*.
The things is most people do not what this means and what are the effects.
I am going to give some real-life examples to explain these effects.

1. new keyword
You can create a struct without using the **new** keyword, but then you need to assign all of the fields manually
```
struct MyStruct
{
	public int a;
}
MyStruct myStruct;
myStruct.a = 1; // Don't do this and you are going to get a compile error
```
If you use the new keyword, then the fields will be assigned to their default values. (e.g. 0 for int)

2. Extension methods for struct won't modify the actual object
```
public static void UpdateA(this MyStruct ms)
{
	ms.a += 1; // The passed ms objects field a is still 1, remember passed by value
}
```
