---
title: Activiti工作流实践
date: 2014-04-24 16:26
categories: 工作流 
tags: 
- Activiti
---

# 配置

首先，配置processEngineConfiguration：

 
```xml
    <bean id="processEngineConfiguration" class="org.activiti.spring.SpringProcessEngineConfiguration">
        <property name="dataSource" ref="dataSource"/>
        <property name="transactionManager" ref="transactionManager"/>
        <property name="databaseSchemaUpdate" value="true"/>
        <property name="jobExecutorActivate" value="false"/>
        <!--<property name="history" value="full"/>-->
        <property name="processDefinitionCacheLimit" value="10"/>

        <!-- UUID作为主键生成策略
        <property name="idGenerator" ref="uuidGenerator" />
        -->

        <!-- 生成流程图的字体 -->
        <property name="activityFontName" value="${diagram.activityFontName}"/>
        <property name="labelFontName" value="${diagram.labelFontName}"/>

        <!-- 缓存支持
        <property name="processDefinitionCache">
            <bean class="me.kafeitu.demo.activiti.util.cache.DistributedCache" />
        </property>-->

        <!-- 自动部署 -->
        <property name="deploymentResources">
            <list>
                <value>classpath*:/deployments/*</value>
            </list>
        </property>

        <!-- 自定义表单字段类型 -->
        <property name="customFormTypes">
            <list>
                <bean class="me.kafeitu.demo.activiti.activiti.form.UsersFormType"/>
            </list>
        </property>

        <!-- JPA -->
        <property name="jpaEntityManagerFactory" ref="entityManagerFactory" />
        <property name="jpaHandleTransaction" value="false" />
        <property name="jpaCloseEntityManager" value="false" />

        <!-- 全局事件 -->
        <property name="typedEventListeners">
            <map>
                <entry key="VARIABLE_CREATED" >
                    <list>
                        <ref bean="variableCreateListener"/>
                    </list>
                </entry>
            </map>
        </property>
    </bean>
```
然后，根据这个配置来生成流程引擎：
 
```xml
    <bean id="processEngine" class="org.activiti.spring.ProcessEngineFactoryBean">
        <property name="processEngineConfiguration" ref="processEngineConfiguration"/>
    </bean>
```
最后，通过流程引擎获取关于仓库、运行、表单、身份认证、任务、历史等相关操作服务：

```xml
    <bean id="repositoryService" factory-bean="processEngine" factory-method="getRepositoryService"/>
    <bean id="runtimeService" factory-bean="processEngine" factory-method="getRuntimeService"/>
    <bean id="formService" factory-bean="processEngine" factory-method="getFormService"/>
    <bean id="identityService" factory-bean="processEngine" factory-method="getIdentityService"/>
    <bean id="taskService" factory-bean="processEngine" factory-method="getTaskService"/>
    <bean id="historyService" factory-bean="processEngine" factory-method="getHistoryService"/>
    <bean id="managementService" factory-bean="processEngine" factory-method="getManagementService"/>
```

# 流程定义

## 添加

最多的部署方式为上传流程定义文件来部署，RepositoryService中提供了多种部署方式，其中包括从输入流、classpath资源文件等方式：

 
```java
         String fileName = file.getOriginalFilename();

        try {
            InputStream fileInputStream = file.getInputStream();
            Deployment deployment = null;

            String extension = FilenameUtils.getExtension(fileName);
            if (extension.equals("zip") || extension.equals("bar")) {
                ZipInputStream zip = new ZipInputStream(fileInputStream);
                deployment = repositoryService.createDeployment().addZipInputStream(zip).deploy();
            } else {
                deployment = repositoryService.createDeployment().addInputStream(fileName, fileInputStream).deploy();
            }
 
```
 

## 查询

Activityti提供链式api来进行查询操作，支持分页查询。一个对一个流程定义部署多次，会产生一个流程定义的多个版本而不是多个流程定义。可以看成流程定义和部署之间存在一对多的关联关系。

```java
        List<Object[]> objects = new ArrayList<Object[]>();
        Page<Object[]> page = new Page<Object[]>(PageUtil.PAGE_SIZE);
        int[] pageParams = PageUtil.init(page, request);
        ProcessDefinitionQuery processDefinitionQuery = repositoryService.createProcessDefinitionQuery().orderByDeploymentId().desc();
        List<ProcessDefinition> processDefinitionList = processDefinitionQuery.listPage(pageParams[0], pageParams[1]);
        for (ProcessDefinition processDefinition : processDefinitionList) {
            String deploymentId = processDefinition.getDeploymentId();
            //Deployment deployment = repositoryService.createDeploymentQuery().deploymentId(deploymentId).singleResult();
            //objects.add(new Object[]{processDefinition, deployment});
            for(Deployment dep : repositoryService.createDeploymentQuery().deploymentId(deploymentId).list()){
            	objects.add(new Object[]{processDefinition, dep});
            	System.out.println(deploymentId);
            }
        }
```
## 删除

 
```java
repositoryService.deleteDeployment(deploymentId, true);
```
第二个参数为true时表示级联删除流程实例、历史记录和任务。也可以通过deleteDeploymentCascade（deploymentId）来级联删除。

 

 

## 启动流程

可以使用RuntimeService来启动一个流程实例，启动流程实例的方式有很多种，主要涉及到

processDefinitionId：流行定义id

processDefinitionKey：流程定义关键字

businessKey：业务关键字

variables(Map)：流程变量

tenantId：

message：

你可以根据不同的情况尝试组合使用上面的信息来启动一个流程实例。
```java

    /**
     * 启动流程
     * @param entity
     */
    public ProcessInstance startWorkflow(Leave entity, Map<String, Object> variables) {
        leaveManager.saveLeave(entity);
        String businessKey = entity.getId().toString();
        ProcessInstance processInstance = null;
        try {
            identityService.setAuthenticatedUserId(entity.getUserId());
            processInstance = runtimeService.startProcessInstanceByKey("leave", businessKey, variables);
            String processInstanceId = processInstance.getId();
            entity.setProcessInstanceId(processInstanceId);
        } finally {
            identityService.setAuthenticatedUserId(null);
        }
        return processInstance;
    }
```
事实上，一个流程启动之后，流程当中并不关心实际的业务内容，而是人为地将这些业务变量放到了流程实例的”存储空间“当中，比如这里的businessKey和variables。真正当流程实例跑起来的时候，它也不关心具体的业务内容，这种关心被挪到了流程定义当中，而流程定义是用户来做的，从而将流程的运行和具体业务解耦出来。

## 待我处理

TaskService提供了这样的查询API来查询与我相关的任务。TaskQuery中有多种方式来确定任务的所有者：

和其他丰富的限制条件，这些任务查询出来之后，还需要与具体的业务对象关联，这样，在更上一层的处理上面对的只是业务对象，而不是工作流中的Task。
```java
    public List<Leave> findTodoTasks(String userId, Page<Leave> page, int[] pageParams) {
        List<Leave> results = new ArrayList<Leave>();
        List<Task> tasks = new ArrayList<Task>();

        // 根据当前人的ID查询
        TaskQuery todoQuery = taskService.createTaskQuery().processDefinitionKey("leave").taskAssignee(userId).active().orderByTaskId().desc()
                .orderByTaskCreateTime().desc();
        List<Task> todoList = todoQuery.listPage(pageParams[0], pageParams[1]);

        // 根据当前人未签收的任务
        TaskQuery claimQuery = taskService.createTaskQuery().processDefinitionKey("leave").taskCandidateUser(userId).active().orderByTaskId().desc()
                .orderByTaskCreateTime().desc();
        List<Task> unsignedTasks = claimQuery.listPage(pageParams[0], pageParams[1]);

        // 合并
        tasks.addAll(todoList);
        tasks.addAll(unsignedTasks);

        // 根据流程的业务ID查询实体并关联
        for (Task task : tasks) {
            String processInstanceId = task.getProcessInstanceId();
            ProcessInstance processInstance = runtimeService.createProcessInstanceQuery().processInstanceId(processInstanceId).active().singleResult();
            String businessKey = processInstance.getBusinessKey();
            if (businessKey == null) {
                continue;
            }
            Leave leave = leaveManager.getLeave(new Long(businessKey));
            leave.setTask(task);
            leave.setProcessInstance(processInstance);
            leave.setProcessDefinition(getProcessDefinition(processInstance.getProcessDefinitionId()));
            results.add(leave);
        }
        page.setTotalCount(todoQuery.count() + claimQuery.count());
        page.setResult(results);
        return results;
    }

```

## 任务签收

 
```java 
  void claim(String taskId, String userId);
```

## 任务处理
```java
taskService.complete(taskId, variables);
```

##　动态表单

在之前的请假流程当中，可能因为我们对请假有比较特殊的，所以请假的核心对象Leave是由我们自己管理的，我们自己搞了个库来存放请假过程中的相关表单数据。但是更为通用和灵活的工作流，表单的内容应该是动态定义出来的，而不是事先由程序写好的。只有这样才能体现工作流引擎的最终目的。有了动态表单，每个节点上，有哪些字段、字段类型是什么、默认值、是否可见、是否可修改、是否必填等信息就可以动态定义了。

## 流程定义XML：
```xml
<process id="leave-dynamic-from" name="请假流程-动态表单">
<documentation>请假流程演示-动态表单</documentation>
<startEvent id="startevent1" name="Start" activiti:initiator="applyUserId">
<extensionElements>
<activiti:formProperty id="startDate" name="请假开始日期" type="date" datePattern="yyyy-MM-dd" required="true" readable="true" writable="true"/>
<activiti:formProperty id="endDate" name="请假结束日期" type="date" datePattern="yyyy-MM-dd" required="true" readable="true" writable="true"/>
<activiti:formProperty id="reason" name="请假原因" type="string" required="true" readable="true" writable="true"/>
</extensionElements>
</startEvent>
<userTask id="deptLeaderAudit" name="部门领导审批" activiti:candidateGroups="deptLeader">
<extensionElements>
<activiti:formProperty id="startDate" name="请假开始日期" type="date" value="${startDate}" datePattern="yyyy-MM-dd" readable="true" writable="false"/>
<activiti:formProperty id="endDate" name="请假结束日期" type="date" value="${endDAte}" datePattern="yyyy-MM-dd" readable="true" writable="false"/>
<activiti:formProperty id="reason" name="请假原因" type="string" value="${reason}" readable="true" writable="false"/>
<activiti:formProperty id="deptLeaderPass" name="审批意见" type="enum" required="true" writable="true">
<activiti:value id="true" name="同意"/>
<activiti:value id="false" name="不同意"/>
</activiti:formProperty>
</extensionElements>
</userTask>
[......]
```
> **疑问**：这些动态表单字段的定义信息保存在哪儿的？

 

## 表单渲染
FormService中提供了按照processDefinitionId或者taskId来获取对应表单数据的方法：

获取到表单数据之后，还能对其中每一项，根据其不同的数据类型进行处理，比如下面的代码中，对枚举类型的数据进行了整理，统一存放到了一个Map中，以供页面使用：

 
```java
        Map<String, Object> result = new HashMap<String, Object>();
        StartFormDataImpl startFormData = (StartFormDataImpl) formService.getStartFormData(processDefinitionId);
        startFormData.setProcessDefinition(null);
        List<FormProperty> formProperties = startFormData.getFormProperties();
        for (FormProperty formProperty : formProperties) {
            Map<String, String> values = (Map<String, String>) formProperty.getType().getInformation("values");
            if (values != null) {
                result.put("enum_" + formProperty.getId(), values);
            }
        }
        result.put("form", startFormData);
```

前台页面对于这样的数据及其类型（单行文本、多行文本域、下拉选择、日期、数字等等）动态生成一个表单。

## 表单数据搜集与存储

这个也没没有社么特别的了，根据上一节中渲染出来的表单，填写数据，然后提交数据到后台即可了。关键是表单数据如何与流程引擎关联起来？很简单，因为我们之间渲染动态表单的时候其中所有字段的name都是按照表单定义来的，所以只要通过request.getParameterMap()，将这个参数键值对送给FormService就可以了，根本不用做其他多余的处理，代码如下：
```java
        Map<String, String> formProperties = new HashMap<String, String>();
        Map<String, String[]> parameterMap = request.getParameterMap();
        Set<Entry<String, String[]>> entrySet = parameterMap.entrySet();
        for (Entry<String, String[]> entry : entrySet) {
            String key = entry.getKey();
            if (StringUtils.defaultString(key).startsWith("fp_")) {
                formProperties.put(key.split("_")[1], entry.getValue()[0]);
            }
        }
        User user = UserUtil.getUserFromSession(request.getSession());
        ProcessInstance processInstance = null;
        try {
            identityService.setAuthenticatedUserId(user.getId());
            processInstance = formService.submitStartFormData(processDefinitionId, formProperties);
        } finally {
            identityService.setAuthenticatedUserId(null);
        }
```
上面的代码对于parameterMap还是稍微做了一点处理，这是因为前台页面中对于动态表单字段的的name都统一增加了fp_，这里要过滤只带有fp_的name值。从而起到一个过滤杂质，避免将其他没用的数据传递给FormService。

这些数据最终被存放到哪里了呢？答案是数据库表ACT_RU_VARIABLE，没错，就是流程标量表中，和相对于动态表单的静态表单而言，动态表单唯一不同的是，动态表单把业务相关的数据也当成流程变量存放起来了，而静态表单是使用自己的表来存放业务数据。