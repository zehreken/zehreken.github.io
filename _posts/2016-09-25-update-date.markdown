---
layout: post
title: A Performance Comparison of OOD and ECS Patterns within Unity
---
**For** the last three months at the office, we programmers talked, discussed and thought a lot about the **Entity-Component-System** architectures. Entity-Component-System will be referred as ECS from now on.
So what is an ECS?
An ECS is an architectural pattern which follows **composition over inheritance** to support flexibility.
In short;

* Entity: A general purpose object, typically a single integer.
* Component: Just data.
* System: Runs the logic related to the component of an entity.

Since Wikipedia has lots of information about what an ECS is, I will leave researching job to the reader.

ECS is very popular in game development and there are a bunch of ECS frameworks around.
Some of the ones that I can find are (they have fancy names btw);

* Entitas (C#) [link](https://github.com/sschmid/Entitas-CSharp)
* Artemis (C#) [link](https://github.com/thelinuxlich/artemis_CSharp)
* EgoCS (C#) [link](https://github.com/andoowhy/EgoCS)
* EntityX (C++) [link](https://github.com/alecthomas/entityx)
* Anax (C++) [link](https://github.com/miguelmartin75/anax)

Entitas is the most popular one and since we use Unity as our engine, we have decided to use Entitas. I have decided to create a simple project that we can test the performance of several development approaches.
Unity at its core is also an entity-component based engine. The most advised programming approach with Unity is writing simple MonoBehaviours and attaching them to GameObjects, either through the editor or using the generic AddComponent method.
But is there something wrong with this approach? Yes, it is slow.
I have compared 4 different development approaches in the project.

1. Using simple and small MonoBehavior extended classes.
2. Using a single MonoBehavior extended class that has all the data and logic in it.
3. Using Entitas.
4. Using plain OOD and [Update Method Pattern](http://gameprogrammingpatterns.com/update-method.html) which I used to use before Entitas.

The test project has 10000 cubes. All of the cubes runs these two simple logic,

Move
{% highlight c linenos %}
void Update()
{
	_transform.Translate(_xVel * Time.deltaTime, _yVel * Time.deltaTime, _zVel * Time.deltaTime, Space.World);
	if (_transform.position.x < -TestConfig.BORDER || _transform.position.x > TestConfig.BORDER)
	{
		_transform.position = new Vector3(TestConfig.BORDER * Mathf.Sign(_transform.position.x), _transform.position.y, _transform.position.z);
		_xVel *= -1f;
	}
	if (_transform.position.y < -TestConfig.BORDER || _transform.position.y > TestConfig.BORDER)
	{
		_transform.position = new Vector3(_transform.position.x, TestConfig.BORDER * Mathf.Sign(_transform.position.y), _transform.position.z);
		_yVel *= -1f;
	}
	if (_transform.position.z < -TestConfig.BORDER || _transform.position.z > TestConfig.BORDER)
	{
		_transform.position = new Vector3(_transform.position.x, _transform.position.y, TestConfig.BORDER * Mathf.Sign(_transform.position.z));
		_zVel *= -1f;
	}
}
{% endhighlight %}

Rotate
{% highlight c# linenos %}
void Update()
{
	_transform.Rotate(_xVel * Time.deltaTime, _yVel * Time.deltaTime, _zVel * Time.deltaTime);
}
{% endhighlight %}

{% highlight ruby linenos %}
void Update()
{
	_transform.Rotate(_xVel * Time.deltaTime, _yVel * Time.deltaTime, _zVel * Time.deltaTime);
}
{% endhighlight %}

## Tell some about the initialization performance
Multiple monos take a lot of time to initialize because MonoBehaviour is huge. Single MonoBehaviour is also takes time because it is still MonoBehaviour. Entitas is a little bit slower probably because of the overhead of the systems. The fastest is Update Method Pattern.
## Tell some about the runtime performance
When it comes to runtime performance multiple MonoBehaviours and single MonoBehaviour is almost the same and worse than  the other two. Entitas is better but not as good as Update Method Pattern.

## Conclusion
At the end Entitas and Update Method Pattern is definitely better than using MonoBehaviours as components. Update Method Pattern is slightly faster when initializing and at runtime but having the flexibility of an ECS is definitely worth it.

It is a trade off, because with a big mono you lose the flexibility of an ECS architecture
