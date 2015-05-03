---
layout: post
title: "Covariance and contravariance of generic parameter"
tag: c# .net
---

It is a popular topic at C#/.NET job interviews or performance reviews. And while many show a general understanding, fewer can write some code examples with those `in` and `out` modifiers. In this post I'll try to give those examples that I believe help in understanding the concept of the topic.

The idea about the variance of the generic parameter is about the ability to use base and derived classes as a generic parameter. The variance of the parameter can be applied to interfaces and delegates that have generic type parameters. it is supported since .NET 4.

Let's start with **invariance**. This is the default behavior. Say we have a hierarchy of 2 classes, and we use them in an interface as a generic parameter:

{% highlight c# %}
class Shape{ }
class Circle: Shape { }
interface IFoo<T> {}
class Foo<T> : IFoo<T>{}
{% endhighlight %}

Now we'll create some instances and see what are our restrictions:
{% highlight c# %}
IFoo<Shape> a = new Foo<Shape>();
/*does not compile, `IFoo` - intravariant
IFoo<Shape> b = new Foo<Circle>();
IFoo<Circle> c = new Foo<Shape>();
*/
IFoo<Circle> d = new Foo<Circle>();
{% endhighlight %}

As we see there are no 'cross-assignments' allowed. I we comment off lines above there will be a compile time error *Cannot implicitly convert type*.

Next is **covariant** type parameter, and it is declared with the keyword `out`.
{% highlight c# %}
interface IFooCo<out T> {}
class FooCo<T>: IFooCo<T> {}
{% endhighlight %}

With the covariance we can assign instances with a derived parameter to an instance with a base parameter. So it should be possible to assign instance of `IFooCo<Circle>` to one of type `IFooCo<Shape>`:

{% highlight c# %}
IFooCo<Shape> m = new FooCo<Shape>();
IFooCo<Shape> n = new FooCo<Circle>();
/* does not compile - covariant
IFooCo<Circle> o = new FooCo<Shape>();
*/
IFooCo<Circle> p = new FooCo<Circle>();
{% endhighlight %}

But we still cannot assign Shape - to Circle. In the framework the example of such type is `IEnumerable`. That is why we can do `IEnumerable<Shape> f = new List<Circle>()`, but not the other way. Looking at meta-data shows us that it is declared with `out` keyword:

{% highlight c# %}
public interface IEnumerable<out T> : IEnumerable
{% endhighlight %}

**Contravariant** type argument allows us assigning object with a base class parameter to an variable with the derived class. Let's create another interface and use the remaining `in` modifier:

{% highlight c# %}
interface IFooContra<in T> {}
class FooContra<T>: IFooContra<T> {}
{% endhighlight %}

And a usage where we can only use a type of 'cross' assignment that is opposite to the previous covariant case:
{% highlight c# %}
IFooContra<Shape> q = new FooContra<Shape>();
/* does not compile - contravariant
IFooContra<Shape> r = new FooContra<Circle>();
*/
IFooContra<Circle> s = new FooContra<Shape>();
IFooContra<Circle> t = new FooContra<Circle>();		
{% endhighlight %}

In .NET framework `Action` and `Func` are contravariant:
{% highlight c# %}
public delegate void Action<in T>(T obj)
{% endhighlight %}

This allows us following:

{% highlight c# %}
static void PrintShape(Shape x) {Console.WriteLine("ShapePrinter: {0}", x);}
...
Action<Circle> k = PrintShape;
{% endhighlight %}

It is possible to be co- and contra- variant at the same time. But only one modifier per type, it is not possible to have one type parameter `in` and `out` at the same time. The `Func` delegate has both `in` and `out` parameters:

{% highlight c# %}
public delegate TResult Func<in T, out TResult>(T arg);
{% endhighlight %}



More reading on the topic - <https://msdn.microsoft.com/en-us/library/dd799517(v=vs.110).aspx>

