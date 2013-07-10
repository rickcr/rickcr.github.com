---
title: NotifyChange From Within onEvent
layout: article
---
A common UI situation is that you want to provide a confirmation box to a user. If the user clicks "yes" you want some process to take
place and possibly refresh some items on your page.

In my case I wanted to prompt the user if they wanted to delete an item, and if 'yes' proceed with the delete and then refresh the item list.

So at first one might thing something like this should work:

{% highlight java %}
@Command
public void delete() {
    Messagebox.show("Do you really want to Delete this Item?", "Delete Item", Messagebox.YES|Messagebox.NO, Messagebox.QUESTION,
        new EventListener() {
            public void onEvent(Event evt) {
                switch (((Integer)evt.getData()).intValue()) {
                    case Messagebox.YES:
                        tierService.deleteTierItem(selectedItem.getCode());
                        break;
                    case Messagebox.NO: break;
                }
            }
        }
    );
}
{% endhighlight %}


The problem here is that the modal messagebox interferes with the message queue and the items refresh BEFORE your confirmation box even takes place.
So, after the delete, the items aren't refreshed. On top of this, even if the above did work as expected
(notify of items change AFTER the method exits), you still have a wasted notify change if the user selected "NO."

The solution as pointed out by hswain and nsharma within this <a href="http://forum.zkoss.org/question/87884/notifychange-happening-async-when-messagebox-with-eventlistener/">post</a> is
to leverage BindUtils.postNotifyChange(...)

My modified code now looks like:

{% highlight java %}

@Command
public void delete() {
    Messagebox.show("Do you really want to Delete this Item?", "Delete Item", Messagebox.YES|Messagebox.NO, Messagebox.QUESTION,
        new EventListener() {
            public void onEvent(Event evt) {
                switch (((Integer)evt.getData()).intValue()) {
                    case Messagebox.YES:
                        tierService.deleteTierItem(selectedItem.getCode());
                        BindUtils.postNotifyChange(null, null, ItemsVM.this, "items");
                        break;
                    case Messagebox.NO: break;
                }
            }
        }
    );
}
{% endhighlight %}

