---
layout: post
title: Cache Miss and Data Locality
---
**Programmers** working in the game industry are proud people, even the gameplay programmers like me. And the good ones are right about it. It is a hard job. We care about performance, we want our code to be fast, sometimes more than necessary. But we usually look for the problem in the wrong place, we try to optimize our code by using faster algorithms but never check or care about where we put our data.

In the past years, the increase in CPU speed compared to Memory speed is enormous. As good as it sounds, it is actually a bad thing. In the past, CPU speed and Memory speed were close and they worked in harmony. But today, because of the speed difference, CPU waits for the Memory and wastes our precious cycles on doing nothing. This is called a cache miss. Whenever the CPU wants some piece of data and can't find it, it is a cache miss. And it looks to a higher level cache, if it can't find it there it is another cache miss and it looks to a higher level cache and so on till it can find it.

## Why
The reason for cache misses is bad data locality. Let's examine this simple code piece written in c++.

A class when instanced creates an object which is 1MB in size
{% highlight ruby %}
class Big
{
	private:
		int actor; // 32b primitive type
		int clutter[262144]; // (1MB - 32b) array makes sure that sizeof big is exatly 1MB
	public:
		void setActor(int size);
		int getActor();
		float getSizeInKB();
		float getSizeInMB();
};
{% endhighlight %}

A class when instanced creates an object which is 32b in size
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

Here, we loop through the arrays
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

On average the second loop completes ~360 times faster than the first loop. The process is the same, which is setting an int field of an object. Why is that? Because the _clutter_ in the Big object causes the cpu to miss the cached data. Because the L1 and L2 caches are full of unnecessary data.



Nice start

https://ark.intel.com/products/52224/Intel-Core-i5-2410M-Processor-3M-Cache-up-to-2_90-GHz
https://en.0wikipedia.org/index.php?q=aHR0cHM6Ly9lbi53aWtpcGVkaWEub3JnL3dpa2kvTGlzdF9vZl9JbnRlbF9Db3JlX2k1X21pY3JvcHJvY2Vzc29ycw

L2 cache = 2 × 256 KB
L3 cache = 3 MB

## Observations
