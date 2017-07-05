---
layout: post
title: Cache Miss and Data Locality
---
**Programmers** working in the game industry are proud people, even the gameplay programmers like me. And the good ones are right about it. It is a hard job. We care about performance, we want our code to be fast, sometimes more than necessary. But we usually look for the problem in the wrong place, we try to optimize our code by using faster algorithms but never check or care about where we put our data.

In the past years, the increase in CPU speed compared to memory speed is enormous. As good as it sounds, it is not actually a good thing. In the past, CPU speed and Memory speed were close and they worked in harmony. But today, because of the speed difference, CPU waits for the Memory for data and wastes those precious cycles on doing nothing.

This is called a **cache miss**. Whenever the CPU wants some piece of data and can't find it, it is a cache miss. And it looks to a higher level cache, if it can't find it there it is another cache miss and it looks to a higher level cache and so on till it can find it.

## Cache Miss
### Cache Miss
The reason for a cache miss is bad data locality. Let's examine this simple code piece written in c++.

{% highlight ruby %}
class Big
{
	private:
		int actor; // 32b primitive type
		int clutter[262144]; // (1MB - 32b) Size of big is exactly 1MB
	public:
		void setActor(int size);
		int getActor();
		float getSizeInKB();
		float getSizeInMB();
};
{% endhighlight %}
###### A class when instanced creates an object which is 1MB in size


{% highlight ruby %}
class Small
{
	private:
		int actor;
	public:
		void setActor(int size);
		int getActor();
		float getSizeInKB();
		float getSizeInMB();
};
{% endhighlight %}
###### A class when instanced creates an object which is 32b in size


{% highlight ruby %}
void loop()
{
	const int SIZE = 50000;

	Big* bigs = new Big[SIZE];
	for (int i = 0; i < SIZE; i++)
	{
		bigs[i].setActor(i);
	}

	Small* smalls = new Small[SIZE];
	for (int i = 0; i < SIZE; i++)
	{
		smalls[i].setActor(i);
	}
}
{% endhighlight %}
###### Here, we loop through the arrays


On average the second loop completes ~360 times faster than the first loop. The process is the same, which is setting an int field of an object. Why is that? Because the **clutter** in the Big object causes the cpu to miss the cached data. Because the L1 and L2 caches are full of unnecessary data.

Let's examine how our data is placed on the actual memory using lldb. You can also use gdb, they are very similar by the way.

I have compiled the script above using clang++ with -g flag to enable debugging with extra information. Here is the simple compile command.
{% highlight ruby %}
clang++ -g -o p main.cpp
{% endhighlight %}

I load the program to lldb using
{% highlight ruby %}
lldb p
{% endhighlight %}

and then add a simple breakpoint to pause the process without terminating it.
{% highlight ruby %}
b main.cpp: 110
{% endhighlight %}

at some point lldb show us the addresses of our two arrays, using those addresses we can examine the memory and see what they have, it may show different addresses of course.
{% highlight ruby %}
Address of bigs: 0x101000000
Address of smalls: 0x1000c4000
{% endhighlight %}

lets examine the first 32 elements of the _smalls_ array using
{% highlight ruby %}
memory read -fx 0x1000c4000 0x1000c4000+128
{% endhighlight %}

{% highlight ruby %}
0x1000c4000: 0x00000000 0x00000001 0x00000002 0x00000003
0x1000c4010: 0x00000004 0x00000005 0x00000006 0x00000007
0x1000c4020: 0x00000008 0x00000009 0x0000000a 0x0000000b
0x1000c4030: 0x0000000c 0x0000000d 0x0000000e 0x0000000f
0x1000c4040: 0x00000010 0x00000011 0x00000012 0x00000013
0x1000c4050: 0x00000014 0x00000015 0x00000016 0x00000017
0x1000c4060: 0x00000018 0x00000019 0x0000001a 0x0000001b
0x1000c4070: 0x0000001c 0x0000001d 0x0000001e 0x0000001f
...
{% endhighlight %}

Look at how nicely the elements are stacked on the memory. The first element is 0, the second is 1, the third is 2 and so on as expected.

Not let's examine the first 32 elements of the _bigs_ array using
{% highlight ruby %}
memory read -fx 0x100200000 0x100200000+128
{% endhighlight %}

{% highlight ruby %}
0x100200000: 0x00000000 0xffffffff 0xffffffff 0xffffffff
0x100200010: 0xffffffff 0xffffffff 0xffffffff 0xffffffff
0x100200020: 0xffffffff 0xffffffff 0xffffffff 0xffffffff
0x100200030: 0xffffffff 0xffffffff 0xffffffff 0xffffffff
0x100200040: 0xffffffff 0xffffffff 0xffffffff 0xffffffff
0x100200050: 0xffffffff 0xffffffff 0xffffffff 0xffffffff
0x100200060: 0xffffffff 0xffffffff 0xffffffff 0xffffffff
0x100200070: 0xffffffff 0xffffffff 0xffffffff 0xffffffff
...
{% endhighlight %}
See, it's full of unnecessary data that causes the CPU to miss the cache.

## What is the solution
It has been a very long time since the hype of Data Oriented Programming but it seems like DOD is the solution. Frankly, I don't think how CPU and Memory manage data will change ever. It is very difficult and also illogical.

I have been programming data oriented for almost a year now. At first (like most of the things I first encounter) I thought it was the holy grail and will solve every problem. Right now I think there are places for OOD and DOD(and in that sense other programming paradigms) in the same software. Both of them have their strengths and weaknesses. Till memory speeds catch up with CPU speeds, using DOD is a very good idea.

Nice start

https://ark.intel.com/products/52224/Intel-Core-i5-2410M-Processor-3M-Cache-up-to-2_90-GHz
https://en.0wikipedia.org/index.php?q=aHR0cHM6Ly9lbi53aWtpcGVkaWEub3JnL3dpa2kvTGlzdF9vZl9JbnRlbF9Db3JlX2k1X21pY3JvcHJvY2Vzc29ycw

g flag
https://stackoverflow.com/questions/10475040/gcc-g-vs-g3-gdb-flag-what-is-the-difference

L2 cache = 2 × 256 KB
L3 cache = 3 MB

## Observations
