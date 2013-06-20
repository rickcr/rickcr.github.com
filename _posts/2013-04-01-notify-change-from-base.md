---
title: ZK Notify Change from Base ViewModels
layout: article
---
In annoying thing in ZK is that if your ViewModel extends another ViewModel and your base ViewModel 
decides that it needs to delegate a call to its subclass, the subclass @NotifyChange({"props}") 
will not automatically be triggered. As an example take:

{% highlight java %}
public abstract class BaseVM {

 public void doSomething() {
  //do Stuff
  //delegate to implementing class
  doSomeStuff();
 }
 public abstract void doSomeStuff();
}

public ChildVM extends BaseVM {
 private String someProperty;

 @NotifyChange("someProperty")
 public void doSomeStuff() {
  someProperty = "blah";
 }

 public String getSomeProperty() {
  return someProperty;
    }
}
{% endhighlight %}

It would be nice if when "doSomeStuff" is called that "someProperty" would be able to be notified so it could display properly in your zul. 
Matze2 on the forums helped provide the work around 
<a href="http://forum.zkoss.org/question/86637/notifychange-in-subclass-methods-called-from-parent-will-not-fire/">http://forum.zkoss.org/question/86637/notifychange-in-subclass-methods-called-from-parent-will-not-fire/</a>
As he mentioned, the solution is to use the "Binder" object. Illustrated as so...

{% highlight java %}
public abstract class BaseVM {
 
 protected Binder binder;

 @Init
 public final void init(@ContextParam(ContextType.BINDER) Binder _binder) {
  binder = _binder;
 }

 public void doSomething() {
  //do Stuff
  //delegate to implementing class
  doSomeStuff();
  binder.notifyChange(this, "someProperty");
 }
 public abstract void doSomeStuff();
}

public ChildVM extends BaseVM {
 private String someProperty;

 @Init(superclass=true)
 public void init() {
 }

 @NotifyChange("someProperty")
 public void doSomeStuff() {
  someProperty = "blah";
 }

 public String getSomeProperty() {
  return someProperty;
    }
}
{% endhighlight %}


Don't forget the @Init(superclass=true) in your subclass (I think you can add it to the Class 
annotation if you aren't providing an init method in your subclass.) The above is clunky but it works.


