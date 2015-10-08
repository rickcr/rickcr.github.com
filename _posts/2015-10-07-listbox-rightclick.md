---
title: ZK Handling Multi-Select Listbox and Right-Click Menupopup in VM
layout: article
---
Recently I had the following requirements for a ZK Listbox:

* Right click menu [ options dynamic based on properties of the object backing the row selected ]
* Be able to select multiple rows (for deletion) with checkboxes

The listbox started out without having the multi-select option. So initially I had a backing "selectedItem" property in my 
backing ViewModel. Since I built up my dynamic right-click menu as well in this ViewModel, in order to access properties of the
row selected, it was just a matter of reading them off the selectedItem property in my view model.

So for example my Listbox initially was set up with

{% highlight xml %}
<listbox id="executionsList" model="@load(vm.executions)"
	selectedItems="@bind(vm.selectedExecution)" ... />
{% endhighlight %}

When building my right-click menu in my ViewModel, I just read what I needed from the bound "selectedExecution"

However, once I needed to introduce the multi-select options, things became a bit uglier. To setup a listbox for multi-select (with checkboxes)
you add both the following properties to your listbox: checkmark="true" multiple="true"

But you'll also need to keep track of the multi select items so you'll need the selectedItems property as well. So something like:

{% highlight xml %}
<listbox id="executionsList" model="@load(vm.executions)"
	 selectedItems="@bind(vm.selectedExecutions)" 
	 checkmark="true" multiple="true">
{% endhighlight %}

Things became buggy, however, when I also wanted the 'selectedItem' property on the listbox for use with my right-click option. Without getting into
the details of what went wrong using both selectedItem and selectedItems, I opted to just get rid of having right-click trigger the row being selected
and handled retrieving the row directly in the right-click menupopup method defined in my ViewModel. This gave me a 'decent' approach: multi-select worked as expected,
and right-click ONLY handled the row that was being right-clicked on (and it did NOT trigger the row being selected for multi-select.)

So here is what was done from a code perspective:

The ZK listbox relevant info:

{% highlight xml %}
<listbox id="executionsList" model="@load(vm.executions)"
		 selectedItems="@bind(vm.selectedExecutions)" 
		 checkmark="true" multiple="true">
	<listhead sizable="true">
		 <!-- not shown for brevity -->
	</listhead> 
	<template name="model" var="item">
		<listitem  
			onRightClick="@command('getExecMenuPopup', target=event.target, execution=item)">
{% endhighlight %}

In order to disable having right-click trigger the row selection, add the following somewhere on the page before your listbox

{% highlight xml %}
<custom-attributes org.zkoss.zul.listbox.rightSelect="false"/>
{% endhighlight %}
 
 or if you want it global, in your zk.xml file add:

{% highlight xml %}
<library-property>
	<name>org.zkoss.zul.listbox.rightSelect</name>
	<value>false</value>
</library-property>
{% endhighlight %}

In my ViewModel I built up my menupopup...

{% highlight java %}
@Command
public void getExecMenuPopup(@BindingParam("target") Component target, @BindingParam("execution") final Execution content) {
	Menupopup menuPopup = new Menupopup(); 

	menuPopup.setPage(target.getPage());
	menuPopup.setWidth("150");
	 
	Menuitem viewSpec = new Menuitem("View Details [ "+execution.getId()+" ]");
	viewSpec.addEventListener("onClick", new EventListener() {
		public void onEvent(Event evt) throws Exception {
			viewDetails(content);
		}
	});
	menuPopup.appendChild(viewSpec);
	
	//other items
{% endhighlight %}