---
title: Spring动态切换数据库
date: 2014-08-22 17:56
categories: Spring
tags: 
---

首先定义一个AbstractRoutingDataSource，Spring给我们留了这样的接口，让我们方便的定义怎么切换数据源：
```java
public class DynamicDataSource extends AbstractRoutingDataSource {

	Logger logger = Logger.getAnonymousLogger();
	
	@Override
	protected Object determineCurrentLookupKey() {
		String p = "a";
		try{
			HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.currentRequestAttributes()).getRequest();
			p = request.getParameter("dataSource");
		}catch(Exception e){
			e.printStackTrace();
		}
		if(p==null){
			p="a";
		}
		return p;
	}

	@Override
	public Logger getParentLogger() throws SQLFeatureNotSupportedException {
		return logger;
	}
}
```
可以通过参数来确定到底使用哪个数据源，总的来说，你可以在你自己的AbstractRoutingDataSource中获取到HttpServletRequest，剩下的就是你自己发挥了。代码里对异常进行了catch，因为这里可能会抛No thread-bound request found的异常，具体什么原因不确定，但是等系统跑起来之后（也就是真的从web访问的时候）可以正常运行。

下面是Spring对sessionFactory的配置：
```xml
  <bean id="faceDataSource" class="test.MyRoutingDataSource">
		<property name="targetDataSources">
			<map>
				<entry key="a" value-ref="dataSource1" />
				<entry key="b" value-ref="dataSource2" />
			</map>
		</property>
		<property name="defaultTargetDataSource" ref="dataSource1" />
	</bean>
	
	
	<bean id="dataSource1" class="com.mchange.v2.c3p0.ComboPooledDataSource"
		destroy-method="close">
		<property name="driverClass" value="${jdbc.driverClassName1}" />
		<property name="jdbcUrl" value="${jdbc.url1}" />
		<property name="user" value="${jdbc.username1}" />
		<property name="password" value="${jdbc.password1}" />
		<property name="preferredTestQuery" value="${preferredTestQuery}" />
		<property name="idleConnectionTestPeriod" value="${idleConnectionTestPeriod}" />
		<property name="testConnectionOnCheckout" value="${testConnectionOnCheckout}" />
	</bean> 
	
	<bean id="dataSource2" class="com.mchange.v2.c3p0.ComboPooledDataSource"
		destroy-method="close">
		<property name="driverClass" value="${jdbc.driverClassName2}" />
		<property name="jdbcUrl" value="${jdbc.url2}" />
		<property name="user" value="${jdbc.username2}" />
		<property name="password" value="${jdbc.password2}" />
		<property name="preferredTestQuery" value="${preferredTestQuery}" />
		<property name="idleConnectionTestPeriod" value="${idleConnectionTestPeriod}" />
		<property name="testConnectionOnCheckout" value="${testConnectionOnCheckout}" />
	</bean> 

    <bean id="sessionFactory"
		class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
		<property name="dataSource" ref="faceDataSource"/>
		<property name="hibernateProperties">
			<props>
				<prop key="hibernate.dialect">${hibernate.dialect}</prop>
				<prop key="hibernate.hbm2ddl.auto">${hibernate.hbm2ddl.auto}</prop>
				<prop key="hibernate.max_fetch_depth">${hibernate.maxFetchDepth}</prop>
				<prop key="hibernate.show_sql">${hibernate.show_sql}</prop>
				<prop key="hibernate.format_sql">${hibernate.format_sql}</prop>
				<prop key="hibernate.jdbc.batch_size">${hibernate.jdbc.batch_size}</prop>
				<prop key="hibernate.cache.use_query_cache">${cache.use_query_cache}</prop>
				<prop key="hibernate.cache.use_second_level_cache">${cache.use_second_level_cache}</prop>
				<prop key="hibernate.cache.region.factory_class">${hibernate.cache.region.factory_class}</prop>
				<prop key="hibernate.temp.use_jdbc_metadata_defaults">${hibernate.temp.use_jdbc_metadata_defaults}</prop>
			</props>
		</property>
		<property name="packagesToScan" value="com.novots.bros.*.entity"/>
	</bean>
```

到目前为止，在几个静态的数据库之间切换已经没有问题了。
没多久，客户又提出，数据库可能是动态增加进来的。而不是想上面那样固定的几个数据库。
几番查找，找到了一个可以动态增加数据源的东西：
```xml
	<bean id="dynamicBeanReader" class="cn.chinacti.crm.dynamicdatasource.entity.DynamicBeanReaderImpl"
		init-method="init">
	</bean>
```
此一段，将引出很多故事：
```java
public class DynamicBeanReaderImpl implements DynamicBeanReader,ApplicationContextAware{  
    private static final Log logger = LogFactory.getLog(DynamicBeanReaderImpl.class);//日记  
      
    private ConfigurableApplicationContext applicationContext = null;    
      
    private XmlBeanDefinitionReader beanDefinitionReader;  
    /*初始化方法*/  
    public void init(){  
        beanDefinitionReader = new XmlBeanDefinitionReader((BeanDefinitionRegistry)  
                applicationContext.getBeanFactory());    
        beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(applicationContext));    
    }  
      
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {    
        this.applicationContext = (ConfigurableApplicationContext)applicationContext;    
    }  
      
    public void loadBean(DynamicBean dynamicBean){   
        long startTime = System.currentTimeMillis();  
        String beanName = dynamicBean.getBeanName();  
        if(applicationContext.containsBean(beanName)){  
            logger.warn("bean【"+beanName+"】已经加载！");  
            return;  
        }  
        beanDefinitionReader.loadBeanDefinitions(new DynamicResource(dynamicBean));  
        logger.info("初始化bean【"+dynamicBean.getBeanName()+"】耗时"+(System.currentTimeMillis()-startTime)+"毫秒。");  
    }   
}
```
这里利用ApplicationContextAware创建一个beanDefinitionReader，通过 beanDefinitionReader可以动态加载loadbean对象到application context中。beanDefinitionReader需要一个Resource对象： 
```java
public class DynamicResource implements Resource {  
    private DynamicBean dynamicBean;  
      
    public DynamicResource(DynamicBean dynamicBean){  
        this.dynamicBean = dynamicBean;  
    }  
    /* (non-Javadoc) 
     * @see org.springframework.core.io.InputStreamSource#getInputStream() 
     */  
    public InputStream getInputStream() throws IOException {  
        return new ByteArrayInputStream(dynamicBean.getXml().getBytes("UTF-8"));  
    }
```
DynamicBean的功能无非是组装bean的xml定义字符串：
```java
/** 
 * 动态bean描述对象 
 */  
public abstract class DynamicBean {  
    protected String beanName;  
  
    public DynamicBean(String beanName) {  
        this.beanName = beanName;  
    }  
  
    public String getBeanName() {  
        return beanName;  
    }  
  
    public void setBeanName(String beanName) {  
        this.beanName = beanName;  
    }  
      
    /** 
     * 获取bean 的xml描述 
     * @return 
     */  
    protected abstract String getBeanXml();  
      
    /** 
     * 生成完整的xml字符串 
     * @return 
     */  
    public String getXml(){  
        StringBuffer buf = new StringBuffer();  
        buf.append("<?xml version=\"1.0\" encoding=\"UTF-8\"?>")  
            .append("<beans xmlns=\"http://www.springframework.org/schema/beans\" xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"")  
            .append("       xmlns:p=\"http://www.springframework.org/schema/p\" xmlns:aop=\"http://www.springframework.org/schema/aop\"")  
            .append("       xmlns:context=\"http://www.springframework.org/schema/context\" xmlns:jee=\"http://www.springframework.org/schema/jee\"")  
            .append("       xmlns:tx=\"http://www.springframework.org/schema/tx\"")  
            .append("       xsi:schemaLocation=\"")  
            .append("           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd")  
            .append("           http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd")  
            .append("           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd")  
            .append("           http://www.springframework.org/schema/jee http://www.springframework.org/schema/jee/spring-jee-2.5.xsd")  
            .append("           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd\">")  
            .append(getBeanXml())  
            .append("</beans>");  
        System.out.println(getBeanXml());
        return buf.toString();  
    }  
}

public class DataSourceDynamicBean extends DynamicBean {  
    private String driverClassName;  
      
    private String url;  
      
    private String username;  
      
    private String password;  
    
    private String preferredTestQuery;
    private String idleConnectionTestPeriod;
    private String testConnectionOnCheckout;
    private String testConnectionOnCheckin;
      
    public DataSourceDynamicBean(String beanName) {  
        super(beanName);  
    }  
    /* (non-Javadoc) 
     * @see org.youi.common.bean.DynamicBean#getBeanXml() 
     */  
    @Override  
    protected String getBeanXml() {  
        StringBuffer xmlBuf = new StringBuffer();  
        
        xmlBuf.append("<bean id=\"").append(beanName).append("\" class=\"com.mchange.v2.c3p0.ComboPooledDataSource\" destroy-method=\"close\">");
        xmlBuf.append("<property name=\"driverClass\" value=\"").append(driverClassName).append("\" />");
        xmlBuf.append("<property name=\"jdbcUrl\" value=\"").append(url).append("\"/>");
        xmlBuf.append("<property name=\"user\" value=\"").append(username).append("\"/>");
        xmlBuf.append("<property name=\"password\" value=\"").append(password).append("\" />");
        xmlBuf.append("<property name=\"preferredTestQuery\" value=\"").append(preferredTestQuery).append("\"/>");
        xmlBuf.append("<property name=\"idleConnectionTestPeriod\" value=\"").append(idleConnectionTestPeriod).append("\"/>");
        xmlBuf.append("<property name=\"testConnectionOnCheckout\" value=\"").append(testConnectionOnCheckout).append("\"/>");
        xmlBuf.append("<property name=\"testConnectionOnCheckin\" value=\"").append(testConnectionOnCheckin).append("\"/>");
        xmlBuf.append("</bean>");
        
        /*xmlBuf.append("<bean id=\""+beanName+"\" class=\"com.mchange.v2.c3p0.ComboPooledDataSource\" destroy-method=\"close\"")  
            .append(" p:driverClassName=\""+driverClassName+"\" ")  
            .append(" p:url=\""+url+"\"")
            .append(" p:username=\""+username+"\"")  
            .append(" p:password=\""+password+"\"/>");*/  
        return xmlBuf.toString();  
    }
      
    public String getDriverClassName() {  
        return driverClassName;  
    }  
    public void setDriverClassName(String driverClassName) {  
        this.driverClassName = driverClassName;  
    }  
    public String getUrl() {  
        return url;  
    }  
    public void setUrl(String url) {  
        this.url = url;  
    }  
    public String getUsername() {  
        return username;  
    }  
    public void setUsername(String username) {  
        this.username = username;  
    }  
    public String getPassword() {  
        return password;  
    }  
    public void setPassword(String password) {  
        this.password = password;  
    }
	public String getPreferredTestQuery() {
		return preferredTestQuery;
	}
	public void setPreferredTestQuery(String preferredTestQuery) {
		this.preferredTestQuery = preferredTestQuery;
	}
	public String getIdleConnectionTestPeriod() {
		return idleConnectionTestPeriod;
	}
	public void setIdleConnectionTestPeriod(String idleConnectionTestPeriod) {
		this.idleConnectionTestPeriod = idleConnectionTestPeriod;
	}
	public String getTestConnectionOnCheckout() {
		return testConnectionOnCheckout;
	}
	public void setTestConnectionOnCheckout(String testConnectionOnCheckout) {
		this.testConnectionOnCheckout = testConnectionOnCheckout;
	}
	public String getTestConnectionOnCheckin() {
		return testConnectionOnCheckin;
	}
	public void setTestConnectionOnCheckin(String testConnectionOnCheckin) {
		this.testConnectionOnCheckin = testConnectionOnCheckin;
	}  
  
} 
```
这个数据源默认使用的是c3p0，你可以根据自己的喜好修改。
OK，这个故事从这里就到头了。

回到故事的开始，谁来使用这个dynamicBeanReader呢？答案是最开始看到的DynamicDataSource，思路是，我需要用这个reader动态修改这个faceDatasources中的datasource Map，这个Map刚开始可能是空的：
```xml
<bean class="cn.chinacti.crm.dynamicdatasource.DynamicDataSource" id="faceDataSource">  
	    <property name="targetDataSources">
	       <map key-type="java.lang.String">
	            <!-- <entry value-ref="defaultLocalhost" key="crmDataSource"></entry>   -->
	       </map>
	    </property>
	    <property name="dynamicBeanReader" ref="dynamicBeanReader"></property> 
	    <property name="defaultTargetDataSource" ref="crmDataSource"></property>
	    <property name="dbName" value="${jdbc.common.dbName}"/>
	    <property name="port" value="${jdbc.common.port}"/>
	    <property name="pwd" value="${jdbc.common.pwd}"/>
	    <property name="userName" value="${jdbc.common.user}"/>
	    
	    <property name="preferredTestQuery" value="${preferredTestQuery}"/>
	    <property name="idleConnectionTestPeriod" value="${idleConnectionTestPeriod}"/>
	    <property name="testConnectionOnCheckout" value="${testConnectionOnCheckout}"/>
	    <property name="testConnectionOnCheckin" value="${testConnectionOnCheckin}"/>
	</bean>
```
问题是AbstractRoutingDataSource貌似没有提供直接访问内置datasource map的功能，那么只能通过反射来搞了：
```java
public class DynamicDataSource extends AbstractRoutingDataSource {

	Logger logger = Logger.getAnonymousLogger();
	
	private DynamicBeanReader dynamicBeanReader;
	private String dbName;
	private String port;
	private String userName;
	private String pwd;
	
    private String preferredTestQuery;
    private String idleConnectionTestPeriod;
    private String testConnectionOnCheckout;
    private String testConnectionOnCheckin;
	
	@Override
	protected Object determineCurrentLookupKey() {
		String ip = null;
		String dsName = null;
		
		try{
			HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.currentRequestAttributes()).getRequest();
			ip = (String)request.getSession().getAttribute("ip");
			if(ip==null ){
				return null;
			}
			dsName = createDSNameByIp(ip);
			ServletContext sc = request.getSession().getServletContext();
			DataSource ds = getByDSName(dsName,sc);
			logger.info("get datasource by name :"+ds);
			logger.info("ip:"+ip);
			if(ds==null){//还没创建
				ds = createNewDataSource(ip, dbName, port, userName, pwd,sc);
				addToTargetDataSources(ds,dsName);
			}
		}catch(Exception e){
			e.printStackTrace();
			logger.info(e.getMessage());
		}
		return dsName;
	}

	private void addToTargetDataSources(DataSource ds, String dsName) {
		Class clazz = AbstractRoutingDataSource.class;
		Field targetDataSourcesField;
		try {
			targetDataSourcesField = clazz.getDeclaredField("resolvedDataSources");
			targetDataSourcesField.setAccessible(true);
			Map<Object,Object> targetDataSources = (Map<Object, Object>) targetDataSourcesField.get(this);
			targetDataSources.put(dsName, ds);
			targetDataSourcesField.set(this, targetDataSources);
		} catch (Exception e) {
			logger.info("修改resolvedDataSources时报错了");
			e.printStackTrace();
			return;
		}
		logger.info("add datasource to resolvedDataSources success.");
	}

	@Override
	public Logger getParentLogger() throws SQLFeatureNotSupportedException {
		return logger;
	}

	@Override
	public Connection getConnection() throws SQLException {
		Connection conn = super.getConnection();
		return conn;
	}
	
	
	private DataSource createNewDataSource(String ip,String dbName,String port,String user,String pwd,ServletContext sc){
		String beanName = createDSNameByIp(ip);
		DataSourceDynamicBean dataSourceDynamicBean = new DataSourceDynamicBean(beanName);  
        dataSourceDynamicBean.setDriverClassName("com.mysql.jdbc.Driver");  
        dataSourceDynamicBean.setUrl("jdbc:mysql://"+ip+":"+port+"/"+dbName+"?useUnicode=true&characterEncoding=utf-8");  
        dataSourceDynamicBean.setUsername(user);
        dataSourceDynamicBean.setPassword(pwd);
        
        dataSourceDynamicBean.setPreferredTestQuery(this.preferredTestQuery);
        dataSourceDynamicBean.setIdleConnectionTestPeriod(this.idleConnectionTestPeriod);
        dataSourceDynamicBean.setTestConnectionOnCheckin(this.testConnectionOnCheckin);
        dataSourceDynamicBean.setTestConnectionOnCheckout(this.testConnectionOnCheckout);
        
        dynamicBeanReader.loadBean(dataSourceDynamicBean);//动态记载dataSource
        logger.info("create datasource:beanName="+beanName+"ip="+ip+"");
        return (DataSource) getApplicationContext(sc).getBean(beanName);
	}
	
	private String createDSNameByIp(String ip){
		return "ds"+ip.replaceAll("\\.", "") ;
	}
	
	
	private DataSource getByDSName(String dsName,ServletContext sc){
		try{
			return (DataSource) getApplicationContext(sc).getBean(dsName);
		}catch(NoSuchBeanDefinitionException e){
			return null;
		}
	}
	
	private ApplicationContext getApplicationContext(ServletContext sc){
		return WebApplicationContextUtils.getWebApplicationContext(sc);
	}

	public DynamicBeanReader getDynamicBeanReader() {
		return dynamicBeanReader;
	}

	public void setDynamicBeanReader(DynamicBeanReader dynamicBeanReader) {
		this.dynamicBeanReader = dynamicBeanReader;
	}

	public String getDbName() {
		return dbName;
	}

	public void setDbName(String dbName) {
		this.dbName = dbName;
	}

	public String getPort() {
		return port;
	}

	public void setPort(String port) {
		this.port = port;
	}

	public String getUserName() {
		return userName;
	}

	public void setUserName(String userName) {
		this.userName = userName;
	}

	public String getPwd() {
		return pwd;
	}

	public void setPwd(String pwd) {
		this.pwd = pwd;
	}

	public String getPreferredTestQuery() {
		return preferredTestQuery;
	}

	public void setPreferredTestQuery(String preferredTestQuery) {
		this.preferredTestQuery = preferredTestQuery;
	}

	public String getIdleConnectionTestPeriod() {
		return idleConnectionTestPeriod;
	}

	public void setIdleConnectionTestPeriod(String idleConnectionTestPeriod) {
		this.idleConnectionTestPeriod = idleConnectionTestPeriod;
	}

	public String getTestConnectionOnCheckout() {
		return testConnectionOnCheckout;
	}

	public void setTestConnectionOnCheckout(String testConnectionOnCheckout) {
		this.testConnectionOnCheckout = testConnectionOnCheckout;
	}

	public String getTestConnectionOnCheckin() {
		return testConnectionOnCheckin;
	}

	public void setTestConnectionOnCheckin(String testConnectionOnCheckin) {
		this.testConnectionOnCheckin = testConnectionOnCheckin;
	}
	
}
```

**原理很简单，AbstractRoutingDataSource本来只是提供了一个口子，说你来决定到底用哪个Key去查找我的Map中的数据源。结果呢，我在查找这个Key的途中，借机使用dynamicBeanReader新建了一个dataSource并成功放到了AbstractRoutingDataSource的datasouce map中（当然，先查查有没有，有就不用创建了）。之后，在返回给AbstractRoutingDataSource一个Key的同时，我确定一定能拿到我想要的dataSource了。 **


**遗留问题：**

在系统启动的时候，根本没有web环境，所以根据前台参数来决定用哪一个库无从谈起。并且faceDataSource必须有一个默认数据源，也就是在系统启动的时候，系统要求一定能从faceDataSource中拿到一个确切的数据源。否则就会抛出异常，这和本身的业务逻辑是相违背的。
