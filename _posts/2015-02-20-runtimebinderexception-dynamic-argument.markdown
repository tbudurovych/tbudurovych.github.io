---
layout: post
title:  "Dynamic parameter type and RuntimeBinderException"
tag: c# .net
---

I have once found that it may be quite surprising behavior of your program when using dynamic typing. Let's look at the following C# code that turns a string into a list of strings. No dynamic typing so far, all static:

{% highlight c# %}

private static IList<string> Do(string p)
{
    return new List<string> { p };
}

{% endhighlight %}

And then a program that calls that method, passing a dynamic variable in it, and removes one element from the result list:

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

The code above compiles and runs just as expected. Note that a variable `str` is of type dynamic. It happens that you are getting a dynamic from someone else's code, right? The method `Do` has a completely static typing.

Now we change `myList.RemoveAt(0);` to be `myList.MyRemoveAt(0);`. Like this:

{% highlight c# %}
static void Main()
{
    dynamic str = "hello world";
    var myList = Do(str);
    myList.MyRemoveAt(0); // changed to non-existing method
}
{% endhighlight %}

Obviously `IList` does not have a method `MyRemoveAt` but the code still compiles. And only at runtime there will be caught `'Microsoft.CSharp.RuntimeBinder.RuntimeBinderException' occurred in System.Core.dll, Additional information: 'System.Collections.Generic.List<string>' does not contain a definition for 'MyRemoveAt'`.

That all happens because of the late binding and the variable `myList` is treated as a dynamic. More about this at [MSDN article](https://msdn.microsoft.com/en-us/library/dd264736.aspx) in Overload Resolution with Arguments of Type Dynamic.

So it may be very dangerous of using dynamics without proper care, since the error will only pop-up at runtime. Note that in the code above we still can get a compile error if we have `IList<string> myList` instead of implicit `var`.