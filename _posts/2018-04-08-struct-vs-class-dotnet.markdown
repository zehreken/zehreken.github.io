---
layout: post
title: Struct vs. Class in .NET
---
**Lately**, I've made a very important mistake while using structs in our persistency system(save/load game).
So I wanted to write this post, which may help others, especially **Unity** developers. Since they mostly use C#
in their programs.

So what is the difference between a **struct** and a **class**?
Everybody should and probably do know by heart that, struct is a *value type* and class is a *reference type* in C#.
The things is most people do not know what this means and what are the effects.
I am going to give some examples to explain these effects.

1.  The **new** keyword. You can create a struct without using the **new** keyword, but then you need to assign all of the fields manually
{% highlight ruby %}
struct MyStruct
{
	public int a;
}
MyStruct myStruct;
myStruct.a = 1; // Don't do this and you are going to get a compile error
{% endhighlight %}
If you use the new keyword, then the fields will be assigned to their default values. (e.g. 0 for int)

2. Extension methods for structs won't modify the actual object
{% highlight ruby %}
public static void UpdateA(this MyStruct ms)
{
	ms.a += 1; // The passed ms object's field 'a' is still 1, remember pass by value
}
{% endhighlight %}
3. Still a result of being a value type, you should be careful with recursion
{% highlight ruby %}
public static MyStruct RecursiveA(MyStruct ms)  
{  
	ms.a += 1;
	if (ms.a < 10)
	{
		RecursiveA(ms);
	}
	return ms;
}

ms = RecursiveA(ms);
{% endhighlight %}
The function above will still terminate when ms.a is 10 but the value of the actual ms.a will be 1.
You can solve this problem by using a class of course or you can assign ms inside the function like this.
{% highlight ruby %}
public static MyStruct RecursiveA(MyStruct ms)  
{  
	ms.a += 1;
	if (ms.a < 10)
	{
		ms = RecursiveA(ms);
	}
	return ms;
}
{% endhighlight %}

If you think that this blog post is wrong or missing, please send me a message.
