---
title: OSWorkflow+MySQL工作流示例
date: 2014-04-18 16:19
categories: 工作流
tags: osworkflow
---
<p>OSWorkflow是一个比较老的工作流框架，官方早就已经停止了对它的支持。但是用来学习工作流的基本思想，用于简单的工作流业务还是可以的。官方文档中部署示例是针对tomcat4来说明的，在这里进行了调整，使其能够跑在tomcat7和MySQL数据库上。</p> 
<p>下载地址：<a href="https://java.net/downloads/osworkflow/" target="_blank">https://java.net/downloads/osworkflow/</a></p> 
<p>解压下载的压缩包，其中的osworkflow-2.8.0-example.war是可以直接跑在任何servlet服务器中的。但是默认情况下，这个示例程序是使用的xml作为来做存储的。现在需要将其调整到数据库中。</p> 
<p>示例程序中有几个比较重要的配置文件：</p> 
<p><strong>osworkflow.xml</strong>：主要配置文件，系统主要通过该文件配置数据库或者流程定义文件。</p> 
<p></p> 
<pre name="code" class="html">&lt;osworkflow&gt;
    &lt;!-- xml的存储方式
    &lt;persistence class="com.opensymphony.workflow.spi.memory.MemoryWorkflowStore"/&gt;
    &lt;factory class="com.opensymphony.workflow.loader.XMLWorkflowFactory"&gt;
    	&lt;property key="resource" value="workflows.xml" /&gt;  只对XMLWorkflowFactory
    &lt;/factory&gt;
     --&gt;
	&lt;!-- 数据库的存储方式 --&gt;
	&lt;persistence class="com.opensymphony.workflow.spi.jdbc.JDBCWorkflowStore"&gt;
			 &lt;property key="datasource" value="java:comp/env/jdbc/osworkflow"/&gt;
			 &lt;property key="entry.sequence" value="select case when a.rowcount=0 then 0 else a.maxId+1 end from (select count(*) as rowcount,max(id) as maxId from os_wfentry)as a"/&gt; 
			 &lt;property key="entry.table" value="OS_WFENTRY"/&gt; 
			 &lt;property key="entry.id" value="ID"/&gt; 
			 &lt;property key="entry.name" value="NAME"/&gt; 
			 &lt;property key="entry.state" value="STATE"/&gt; 
			 &lt;property key="step.sequence" value="select sum(c1) from (select 1 tb, count(*) c1 from os_currentstep union select 2 tb, count(*) c1 from os_historystep) as a"/&gt; 
			 &lt;property key="history.table" value="OS_HISTORYSTEP"/&gt; 
			 &lt;property key="current.table" value="OS_CURRENTSTEP"/&gt; 
			 &lt;property key="historyPrev.table" value="OS_HISTORYSTEP_PREV"/&gt; 
			 &lt;property key="currentPrev.table" value="OS_CURRENTSTEP_PREV"/&gt;
			 &lt;property key="step.id" value="ID"/&gt; 
			 &lt;property key="step.entryId" value="ENTRY_ID"/&gt; 
			 &lt;property key="step.stepId" value="STEP_ID"/&gt; 
			 &lt;property key="step.actionId" value="ACTION_ID"/&gt; 
			 &lt;property key="step.owner" value="OWNER"/&gt; 
			 &lt;property key="step.caller" value="CALLER"/&gt; 
			 &lt;property key="step.startDate" value="START_DATE"/&gt; 
			 &lt;property key="step.finishDate" value="FINISH_DATE"/&gt; 
			 &lt;property key="step.dueDate" value="DUE_DATE"/&gt; 
			 &lt;property key="step.status" value="STATUS"/&gt; 
			 &lt;property key="step.previousId" value="PREVIOUS_ID"/&gt;  
	&lt;/persistence&gt;
	 &lt;factory class="com.opensymphony.workflow.loader.JDBCWorkflowFactory"&gt;
			 &lt;property key="datasource" value="java:comp/env/jdbc/osworkflow"/&gt;
	 &lt;/factory&gt;
&lt;/osworkflow&gt;</pre>其中主要需要注意的地方是 
<p></p> 
<p>datasource： 这里配置了jndi数据源，名称是jdbc/osworkflow，你需要在tomcat的context.xml中进行相应的配置。数据库文件可以在下载包的src\etc\deployment\jdbc中找到。官方原配置中没有"java:comp/env/"，导致在tomcat中始终报“找不到数据源”的错误。</p> 
<p>entry.sequence和step.sequence：这两个是用来配置os_wfentry（流程实例）表和os_currentstep(当前步骤或节点)表的主键生成策略的。官方原配置中没有考虑MySQL，而是使用Oracle的nextVal数据库函数，要在MySQL上使用，我把改成了上述配置。</p> 
<p>persistence：配置存储方式，WorkflowStore有很多种，默认配置的是MemoryWorkflowStore，这里改成了JDBCWorkflowStore。</p> 
<p>factory：也是配置存储方式，两个基本差不多，但是factory中不需要配置项persistent中那么多的property，只需要配置一个datasource就可以了。至于为什么非得配置两个就不得而知了，也许作者是准备将流程定义和流程执行的数据放在不同的数据库中，甚至用不同的存储方式来存储。因为从源码中可以看出，factory负责的是流行定义方面的信息处理，额persistent则主要负责管理流程的执行过程。</p> 
<p><br></p> 
<p><strong>propertyset.xml</strong>：如果你选择了JDBCWorkflowStore，那么还需要配置这个文件：</p> 
<p></p> 
<pre name="code" class="html">&lt;propertysets&gt;
    &lt;propertyset name="jdbc" class="com.opensymphony.module.propertyset.database.JDBCPropertySet"&gt;
        &lt;arg name="datasource" value="jdbc/osworkflow"/&gt;
        &lt;arg name="table.name" value="OS_PROPERTYENTRY"/&gt;
        &lt;arg name="col.globalKey" value="GLOBAL_KEY"/&gt;
        &lt;arg name="col.itemKey" value="ITEM_KEY"/&gt;
        &lt;arg name="col.itemType" value="ITEM_TYPE"/&gt;
        &lt;arg name="col.string" value="STRING_VALUE"/&gt;
        &lt;arg name="col.date" value="DATE_VALUE"/&gt;
        &lt;arg name="col.data" value="DATA_VALUE"/&gt;
        &lt;arg name="col.float" value="FLOAT_VALUE"/&gt;
        &lt;arg name="col.number" value="NUMBER_VALUE"/&gt;
    &lt;/propertyset&gt;
&lt;/propertysets&gt;</pre>这里又配置了一遍数据源。 
<p></p> 
<p><br></p> 
<p><strong>workflows.xml</strong>：看名称就知道，它是用来配置流行定义文件的。如果你在osworkflow.xml中配置XMLWorkflowFactory，那么你还需要为这个factory配置一个resouce来引用下面的配置：</p> 
<p></p> 
<pre name="code" class="html">&lt;workflows&gt;
    &lt;workflow name="example" type="resource" location="example.xml"/&gt;
&lt;/workflows&gt;</pre> 
<br> 这个文件中可以定义多个流行定义，每个流行定义对应一个文件。如上所示定义了一个example流行，对应的文件是example.xml。 
<p><br></p>