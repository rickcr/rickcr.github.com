---
title: ZK Paging Large Results
layout: article
---
#Introduction
One thing typically needed in any sort of web application is way to paginate a large list of data. On top of pagination, you also need a way to sort the data (typically be clicking a column header.) For small amounts of data the solution is always pretty simple, but when dealing with a large amount of data, it's a bit trickier since you rrarely want to pull back all the data at once and store it in memory.</p>
 
In regard to pagination large amounts of data in ZK, there are some solutions floating around out there. I found this solution proposed
<a href="http://books.zkoss.org/wiki/Small_Talks/2012/March/Handling_a_Trillion_Data_Using_ZK">http://books.zkoss.org/wiki/Small_Talks/2012/March/Handling_a_Trillion_Data_Using_ZK</a> but I couldn't really figure it out and, unless I'm missing something, it requires ALL your data to be stored server side (someone correct me if I'm wrong there?)  This seems sort of expensive. Most people are using something like Hibernate or MyBaitis (my choice) and I don't think it's worth populating EVERY object up front.    
 
 
Another approach found here <a href="http://books.zkoss.org/wiki/Small_Talks/2009/May/Paging_Sorting_with_a_filter_object">http://books.zkoss.org/wiki/Small_Talks/2009/May/Paging_Sorting_with_a_filter_object</a> is interesting as well but it confused me, since I started most of my learning of ZK recently with an MVVM approach and not dealing with the older controller based approach. This other approach is more like what I'm doing below <a href="http://books.zkoss.org/wiki/Small_Talks/2009/July/Handling_huge_data_using_ZK">http://books.zkoss.org/wiki/Small_Talks/2009/July/Handling_huge_data_using_ZK</a>, but I'm doing it all in my ViewModel. I'm sure purists won't like it but it seems like an easier approach than most of the other ones I've seen.
 
NOTE: At the time of this writing, one of my complaints is that ZK now allows an MVVM approach, which I love, but when you view forum posts
 or google solutions they are often using an older approach and trying to mix the two isn't as simple as I would like (probably mostly due to my ignorance.)
 I do wish someone would write some best practice documents of how to blend the two.  What I've ended up deciding to do in most cases, is to just do the things 
 I want within my ViewModel directly. I'm sure this isn't the "pure" approach, and it certainly means the ViewModel has to know a bit more than it should about 
 the UI, but I find it working for me and it certainly means the ViewModel has to know a bit more than it should about the UI. (Possible a SelectComposer would
 be better, but that has its own issues when you need to notify it from a ViewModel - having to listen on queue vs just listening for a GlobalCommand? Clean up 
 issues then arise using listeners on queues.) 
 
 
Anyway, here is what I came up with. For now I'm just showing the relevant code.  I think it'll make sense without a downloadable example.
 
  

##The View Model
 
{% highlight java %}
public class ExecutionsVM {
	private final static Logger logger = LoggerFactory.getLogger(ExecutionsVM.class);
	
	private List<execution> executions = new ArrayList<execution>();
	private ExecutionSearchFilter searchFilter = new ExecutionSearchFilter();
	private long executionsTotalSize = 0l;
	private int executionsPageSize = 25;
	private int currentPageNumber = 0;
	
	@WireVariable
	private ExecutionService executionService;
	
	@NotifyChange({"executions","executionsTotalSize"})
	@Command
	public void pageExecutions(@BindingParam("pageNum") int pageNum) {
		logger.debug("pageExecutions for pagNum {}", pageNum);
		this.currentPageNumber = pageNum;
		populateSearchFilter();
		executions = executionService.getExecutions(searchFilter);
		executionsTotalSize = executionService.getExecutionsSize(searchFilter);
	}

	private void populateSearchFilter() {
		int base = this.currentPageNumber * executionsPageSize;
		int firstRow = base + 1;
		int lastRow = base + executionsPageSize;
		searchFilter.setLowerRowLimit(firstRow);
		searchFilter.setUpperRowLimit(lastRow);
	}

}
{% endhighlight %}
 

##The zul
 
{% highlight java %}
<paging id="executionsPaging"
  onCreate="executionsList.setPaginal(self)"
  totalSize="@bind(vm.executionsTotalSize)"
  pageSize="@bind(vm.executionsPageSize)"
  activePage="@bind(vm.currentPageNumber)"
  onPaging="@command('pageExecutions', pageNum=event.activePage)"
  /> 


 <listbox id="executionsList" model="@load(vm.executions)"
       selectedItem="@bind(vm.selectedExecution)".... />
{% endhighlight %}
 

##Execution Service

{% highlight text %}
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
