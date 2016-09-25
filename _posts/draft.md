---
layout: post
title: A Performance Comparison of OOD and ECS Patterns within Unity
---
**Nowadays** at the office, we programmers talked, discussed and thought a lot about the **Entity-Component-System**. Entity-Component-System will be referred as ECS from now on.
So what is an ECS?
An ECS is an architectural pattern which follows **composition over inheritance** to support flexibility.
In short,

* Entity: A general purpose object, typically a single integer.
* Component: Just data.
* System: Runs the logic related to the component of an entity.

Since Wikipedia has lots of information about what an ECS is, I will leave researching job to the reader.

ECS is very popular in game development and there are a bunch of ECS frameworks around.
Some of the ones that I can find are (they have fancy names btw):

* Entitas (C#) [link](https://github.com/sschmid/Entitas-CSharp)
* Artemis (C#) [link](https://github.com/thelinuxlich/artemis_CSharp)
* EgoCS (C#) [link](https://github.com/andoowhy/EgoCS)
* EntityX (C++) [link](https://github.com/alecthomas/entityx)
* Anax (C++) [link](https://github.com/miguelmartin75/anax)

Entitas is the most popular one and since we use Unity as our engine, we have decided to use Entitas. I have decided to create a simple project that we can test the performance of several development approaches.
Unity at its core is also a entity-component based engine. The most advised programming approach with Unity is writing simple MonoBehaviours and attach them to GameObjects, either through the editor or using the AddComponent method.
But is there something wrong with this approach? Yes, it is slow.
I have compared 4 different development approaches in the project.

1. Using simple and small MonoBehavior extended classes.
2. Using a single MonoBehavior extended class that has all the data and logic in it.
3. Using Entitas.
4. Using plain OOD and [Update Method Pattern](http://gameprogrammingpatterns.com/update-method.html) which I used to use before Entitas.

The test project has 10000 cubes. All of the cubes executes these two simple logic,

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
{% highlight c linenos %}
void Update()
		{
			_transform.Rotate(_xVel * Time.deltaTime, _yVel * Time.deltaTime, _zVel * Time.deltaTime);
		}
{% endhighlight %}
