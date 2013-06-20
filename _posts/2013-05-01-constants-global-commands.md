---
title: ZK Using Constants for GlobalCommands
layout: article
---
Most of the documentation and examples demonstrate declaring GlobalCommmand annotations like: 

{% highlight java %}
@GlobalCommand  
public void someMethod() { ... }  
{% endhighlight %}

and calling it from some other ViewModel like so:

{% highlight java %}
BindUtils.postGlobalCommand(null, null, "someMethod", null);
{% endhighlight %}

The problem with this is it's extremely fragile. If you decide to refactor your method name, you end up then having to do a search and replace for everywhere you've called your Global command with "someMethod." Possibly more annoying is you do not get any IDE help when you want to post to this GlobalCommand from another ViewModel... you need to recall the exact name of the method and if you can't recall the name you have to remember where that GlobalCommand was defined which could take time in a large project. Instead, I recommend always getting in the habit of using a GlobalCommandValues class that declares the GlobalCommand as String constant and then using it in your annotation and of course in your postGlobalCommand calls. This seems obvious, but I wasn't even aware it was an option. 


## GlobalCommandValues class
{% highlight java %}
public static final String SOME_METHOD = "someMethod";
{% endhighlight %}

## ViewModel
{% highlight java %}
@GlobalCommand(GlobalCommandValues.SOME_METHOD)
public void someMethodDoesntHaveToMatchYourConstant() { ... }
{% endhighlight %}

## Usage
{% highlight java %}
BindUtils.postGlobalCommand(null, null, GlobalCommandValues.SOME_METHOD, null);
{% endhighlight %}

