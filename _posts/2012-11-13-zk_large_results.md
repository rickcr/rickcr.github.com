---
title: ZK Paging Large Results
layout: article
---
Update 10/3/2013 - When I originally wrote this, the pagination examples using servers-side sorting (without pulling everything back where sort of weak.)
There is now this post however <a href="http://fadishei.wordpress.com/2012/03/22/zk-mvvm-design-pattern-and-server-side-paging/">http://fadishei.wordpress.com/2012/03/22/zk-mvvm-design-pattern-and-server-side-paging/</a>
which is similar to the approach I initially had demonstrated here. My initial approach was handling the onPaging event and it worked well,
but the approach in the above link is slightly cleaner since you don't need the onPaging
declaration event in your zul and instead you handle things in setActivePage. So I've now updated this example.
 It's really illustrating the same thing as the link above, but having
two links out there with the same approach is not a bad thing:) This example shows a bit more of the MyBatis code as well, but that's not super critical. I also
use a 'searchFilter' since in my real code that I'm using I send a lot more to the cut back than just the start/end rows (eg search by name, etc.)

NOTE: At the time of this writing, one of my complaints is that ZK now allows an MVVM approach, which I love, but when you view forum posts
 or google solutions they are often using an older approach and trying to mix the two isn't as simple as I would like (probably mostly due to my ignorance.)
 I do wish someone would write some best practice documents of how to blend the two.  What I've ended up deciding to do in most cases, is to just do the things 
 I want within my ViewModel directly. I'm sure this isn't the "pure" approach, and it certainly means the ViewModel has to know a bit more than it should about 
 the UI, but I find it working for me and it certainly means the ViewModel has to know a bit more than it should about the UI. (Possible a SelectComposer would
 be better, but that has its own issues when you need to notify it from a ViewModel - having to listen on queue vs just listening for a GlobalCommand? Clean up 
 issues then arise using listeners on queues.)
 


##The View Model
 
{% highlight java %}
public class ExecutionsVM {
	private final static Logger logger = LoggerFactory.getLogger(ExecutionsVM.class);
	
	private ExecutionSearchFilter searchFilter = new ExecutionSearchFilter();
	private long executionsTotalSize = 0l;
	private int executionsPageSize = 25;
	private int activePage = 0;
	
	@WireVariable
	private ExecutionService executionService;

	@NotifyChange({"executions","executionsTotalSize"})
	public void setActivePage(int activePage)  {
		this.activePage = activePage;
		populateSearchFilter();
	}

	private void populateSearchFilter() {
		int base = activePage * executionsPageSize;
		int firstRow = base + 1;
		int lastRow = base + executionsPageSize;
		searchFilter.setLowerRowLimit(firstRow);
		searchFilter.setUpperRowLimit(lastRow);
	}

	public List<Execution> getExecutions() {
		return executionService.getExecutions(searchFilter);
	}

	public int getExecutionsTotalSize() {
	 	executionsTotalSize = executionService.getExecutionsSize(searchFilter);
	}

	//obvious other getters and setters not shown
	//getActivePage, etc

}
{% endhighlight %}
 

##The zul
 
{% highlight xml %}
<paging id="executionsPaging"
  onCreate="executionsList.setPaginal(self)"
  totalSize="@bind(vm.executionsTotalSize)"
  pageSize="@bind(vm.executionsPageSize)"
  activePage="@bind(vm.activePage)"
  /> 


 <listbox id="executionsList" model="@load(vm.executions)"
       selectedItem="@bind(vm.selectedExecution)".... />
{% endhighlight %}
 

##Execution Service

   
{% highlight java %} 
@Service
public class ExecutionService {
	@Resource
	private ExecutionMapper executionMapper;

	public List<execution> featchExecutions(ExecutionSearchFilter searchFilter) {
		return executionMapper.getExecutions(searchFilter);
	}
}
{% endhighlight %}
  

##Search Filter POJO
 
{% highlight java %}
public class ExecutionSearchFilter {
 private int upperRowLimit;
 private int lowerRowLimit;
    //setters/getters
}
{% endhighlight %}
 

##Execution Mapper

{% highlight java %}
public interface ExecutionMapper {
 List<execution> featchExecutions(ExecutionSearchFilter searchFilter) 
}
{% endhighlight %}

##Execution mapper xml

{% highlight text %}
<select id="getExecutions" resultMap="executionResultMap" parameterType="ExecutionSearchFilter">
  select
   EXECUTION_ID,
   EXECUTION_NM,
   ROW_NUM
  FROM
   (
    SELECT
     tempRow.EXECUTION_ID,
     tempRow.EXECUTION_NM,
     rownum ROW_NUM
    from
    (
     SELECT
      e.EXECUTION_ID,
      e.EXECUTION_NM
     FROM
      EXECUTION_T e
    ) tempRow
    WHERE
     rownum <![CDATA[ <= ]]> #{upperRowLimit}
   )
  WHERE
   ROW_NUM >= #{lowerRowLimit}
</select>
{% endhighlight %}
