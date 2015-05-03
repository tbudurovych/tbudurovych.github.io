---
layout: post
title:  "Parsing JSON in IronPython"
tag: ironpython .net
---

I was working with TIBCO Spotfire that has hosted IronPython scripting. And I had to process some JSON data, which is easy in Python by doing `import json`. But that construct gives an import error in IronPython "No module named json".

So I decided to use JavaScriptSerializer .NET class that I have been using in C#:

{% highlight python %}
peopleJson = '''[
	{"name":"Jon", "age": "29", canSing:false, favColors:["red", "white"]},
	{"name":"Lilly", "age": "55", "canSing": true}
]'''
#convert json to string
import clr
clr.AddReference('System.Web.Extensions')
from System.Web.Script.Serialization import JavaScriptSerializer
people = JavaScriptSerializer().DeserializeObject(peopleJson)

for person in people:
	# each line is a dictionary
	print person['name'] + ' ' + person['age']


{% endhighlight %}

The output is:

```
Jon 29
Lilly 55
```

This is not the only way of course. It is also possible to achieve similar thing using `eval`, or by importing other more specific libraries.