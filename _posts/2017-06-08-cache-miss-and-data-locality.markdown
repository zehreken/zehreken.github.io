---
layout: post
title: Cache Miss and Data Locality
---
**Programmers** working in the game industry are proud people, even the gameplay programmers like me. And the good ones are right about it. It is a hard job. We care about performance, we want our code to be fast, sometimes more than necessary. But we usually look for the problem in the wrong place, we try to optimize our code by using faster algorithms but never check or care about where we put our data.

In the past years, the increase in CPU speed compared to Memory speed is enormous. As good as it sounds, it is actually a bad thing. In the past, CPU speed and Memory speed were close and they worked in harmony. But today, because of the speed difference, CPU waits for the Memory and wastes our precious cycles on doing nothing. This is called a cache miss. Whenever the CPU wants some piece of data and can't find it, it is a cache miss. And it looks to a higher level cache, if it can't find it there it is another cache miss and it looks to a higher level cache and so on till it can find it.

## Why
The reason for cache misses is bad data locality. Let's examine this simple code piece written in c++.

{% highlight ruby %}
#include <iostream>
#include <time.h>
#include <stack>

class Big
{
	private:
		int actor;
		int clutter[262144]; // (1MB - 32b) array makes sure that sizeof big is exatly 1MB
	public:
		void setActor(int size);
		int getActor();
		float getSizeInKB();
		float getSizeInMB();
};

void Big::setActor(int actor)
{
	this->actor = actor;
}

int Big::getActor()
{
	return this->actor;
}

float Big::getSizeInKB()
{
	return sizeof(Big) / 1024.f;
}

float Big::getSizeInMB()
{
	return sizeof(Big) / (1024.f * 1024.f);
}

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

void Small::setActor(int actor)
{
	this->actor = actor;
}

int Small::getActor()
{
	return this->actor;
}

float Small::getSizeInKB()
{
	return sizeof(Small) / 1024.f;
}

float Small::getSizeInMB()
{
	return sizeof(Small) / (1024.f * 1024.f);
}

int main()
{
	using namespace std;
	
	const int SIZE = 50000;
	
	Big* bigs = new Big[SIZE];
	clock_t t1, t2;
	t1 = clock();
	for (int i = 0; i < SIZE; i++)
	{
		bigs[i].setActor(i);
	}
	t2 = clock();
	float diffBig = ((float)t2 - (float)t1) / CLOCKS_PER_SEC;
	cout << "Running time for bigs: " << diffBig << endl;
	cout << "Size of big(KB): " << bigs[0].getSizeInKB() << endl;
	cout << "Size of bigs(KB): " << bigs[0].getSizeInKB() * SIZE << endl;
	cout << "Address of bigs: " << bigs << endl;
	cout << "Address of bigs: " << &bigs[0] << endl;
	
	// ==========
	
	Small* smalls = new Small[SIZE];
	clock_t t3, t4;
	t3 = clock();
	for (int i = 0; i < SIZE; i++)
	{
		smalls[i].setActor(i);
	}
	t4 = clock();
	float diffSmall = ((float)t4 - (float)t3) / CLOCKS_PER_SEC;
	cout << "\nRunning time for smalls: " << diffSmall << endl;
	cout << "Size of small(KB): " << smalls[0].getSizeInKB() << endl;
	cout << "Size of smalls(KB): " << smalls[0].getSizeInKB() * SIZE << endl;
	cout << "Address of smalls: " << smalls << endl;
	cout << "Address of smalls: " << &smalls[0] << endl;

	cout << "Result: " << (diffBig / diffSmall) << " times faster" << endl;

	cout << "End of program" << endl;
	
	return 0;
}
{% endhighlight %}





Nice start

https://ark.intel.com/products/52224/Intel-Core-i5-2410M-Processor-3M-Cache-up-to-2_90-GHz
https://en.0wikipedia.org/index.php?q=aHR0cHM6Ly9lbi53aWtpcGVkaWEub3JnL3dpa2kvTGlzdF9vZl9JbnRlbF9Db3JlX2k1X21pY3JvcHJvY2Vzc29ycw

L2 cache = 2 × 256 KB
L3 cache = 3 MB

## Observations
