---
layout: post
title:  "Dynamic parameter type and RuntimeBinderException"
tag: c# .net
---

Let's look at the following C# code that calls method `Do` to get a list of strings and then removes one element from it.

{% highlight c# %}

	class Program
	{
	    static void Main()
	    {
	        dynamic str = "hello world";
	        var myList = Do(str);
	        myList.RemoveAt(0);
	    }

	    private static IList<string> Do(string p)
	    {
	        return new List<string> { p };
	    }
	}

{% endhighlight %}

The code above compiles and runs just as expected. Note that a variable `str` is of type dynamic. It can happen that you are getting a dynamic from someone else's code. The method `Do` has a completely static typing.

Now we change `myList.RemoveAt(0);` to be `myList.MyRemoveAt(0);`.

Like this:

{% highlight c# %}
static void Main()
{
    dynamic str = "hello world";
    var myList = Do(str);
    myList.MyRemoveAt(0); // changed to non-existing method
}
{% endhighlight %}

Obviously `IList` does not have a method `MyRemoveAt` but the code still compiles. And only at runtime there will be caught `'Microsoft.CSharp.RuntimeBinder.RuntimeBinderException' occurred in System.Core.dll, Additional information: 'System.Collections.Generic.List<string>' does not contain a definition for 'MyRemoveAt'`.

That all happens because of late binding and the variable `myList` is treated as a dynamic. More about this at [MSDN article](https://msdn.microsoft.com/en-us/library/dd264736.aspx) in Overload Resolution with Arguments of Type Dynamic.

So it may be very dangerous of using dynamics without proper care, since the error will only pop-up at runtime. Note that in the code above we still can get a compile error if we have `IList<string> myList` instead of implicit `var`.