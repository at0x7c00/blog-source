---
title: 基于Hibernate实现多租户（Multi-Tendency）功能
date: 2016-06-21 23:50
categories: 
- Java
tags: 
- SAAS
- 多租户
- Hibernate
---

这几天的研究，四处搜寻资料，基本理清实现多租户的一个思路。至于多租户是什么可参考《[浅析多租户在 Java 平台和某些 PaaS 上的实现](http://www.ibm.com/developerworks/cn/java/j-lo-mutiltenancy/index.html?utm_source=tuicool&utm_medium=referral)》。里面提到了很多激动人心的是JavaEE8会加入对多租户的支持。但是这个真的不知道要等到什么时候了。

----

我所维护的一个系统是基于Hibernate的，现在准备修改架构，希望能够提供Saas服务。这样一个升级，首先想到的是，数据库的数据会急剧增加，面对大数据的时候，首先的选择估计会扔掉Hibernate，使用纯的JDBC或者用MyBatis。

如果使用JDBC那就完全自由了，但自由确实自由了，现有的整个系统是以Hibernate为基础的，扔掉Hibernate的前提是你要自己来实现至少用起来还算凑合的一个DAO。我没有找到市面上有这么一种JDBC框架，方便使用者进行分表、分库等操作。

另外我还看到EclipseLink也可以实现多租户功能，但据说他的社区比Hibernate差多了，也只能作罢。

----
所以，还是回到了这篇文章上来了：《[数据层的多租户浅谈](http://www.ibm.com/developerworks/cn/java/j-lo-dataMultitenant/index.html)》（看完了记得回来，我这里有补充）。

这里讲到了Hibernate在4.0的时候就已经支持多租户了，实现起来分几种方式
1.一个租户一个单独数据库（DATABASE）-**注意，是物理意义上的独立，可以理解为不同的数据源**；
2.多个租户公用一个数据源，但每个租户有不同的数据库（SCHEMA）；
3.多个租户都在一个数据库里，数据也完全在一起，通过一个字段（列）来区分（DISCRIMINATOR）；

我在看上面那篇引文的时候对这三种的描述上是有**误解**的，我以为：
DATABASE：不同数据库，也许是同一个数据源上的
SCHEMA：对多个租户进行分表（但从头到尾，没有看到任何地方用分表的方式实现过多租户）

说说我对这三种情况如何选择的想法。
首先如果你的数据量不大那你完全可以使用DISCRIMINATOR方式，也就是把所有数据都放到一个表里面。这样做的前提是，你是在从无到有新建一个项目。否则，将不支持多租户的系统改造成多租户时不建议这么做。因为这有可能会影响到你程序的方方面面，你要改的代码也许不计其数。
第二种，分数据库（SCHEMA）。反正我是选择的这种方式，因为多租户首先考虑的是有可能数据量会很大，相对于DISCRIMINATOR方式而言，这种分数据库其实就是分表了。另外，对于数据的备份、迁移都是非常方便的。
最后一种，分数据源（DATABASE）。我觉得这完全可以和SCHEMA方式配合起来用，作为它的一种补充，什么时候需要补充？当一个数据源上的数据库多到严重影响性能的时候，就可以考虑分多个数据源了。

----
好了说了这么多，直接上代码吧，首先，Hibernate给了我们一个确定tendantId的接口：


```
import org.hibernate.context.spi.CurrentTenantIdentifierResolver;

import com.xyz.util.threadlocal.ThreadLocalUtil;

public class TenantIdResolver implements CurrentTenantIdentifierResolver {

	@Override
	public String resolveCurrentTenantIdentifier() {
		String tendantId = ThreadLocalUtil.tendantId.get();
		tendantId  = "dataSource1:db1";//TODO 删除TEST
		return tendantId;
	}

	@Override
	public boolean validateExistingCurrentSessions() {
		return true;
	}

}
```
这里也前面引文里提到的不太一样的是，我把tendantId的信息量加大了，不光能表示使用哪个数据库，还能同时表示使用哪个数据源。（前面提到的，实现SCHEMA时还实现DATABASE）
这里使用了一个ThreadLocal来存储tendantId，具体的设置点在哪儿，你可以根据你的应用场景来考虑。我现在还没考虑到这里，等考虑好了再来补充。

```
import java.sql.Connection;
import java.sql.SQLException;
import java.util.HashMap;
import java.util.Map;

import org.hibernate.HibernateException;
import org.hibernate.service.jdbc.connections.internal.C3P0ConnectionProvider;
import org.hibernate.service.jdbc.connections.spi.MultiTenantConnectionProvider;
import org.hibernate.service.spi.Configurable;
import org.hibernate.service.spi.ServiceRegistryAwareService;
import org.hibernate.service.spi.ServiceRegistryImplementor;
import org.hibernate.service.spi.Stoppable;

import com.xyz.util.threadlocal.ThreadLocalUtil;

/**
 * 分数据库多租户
 * @author huqiao(412853638@qq.com)
 */
public class SchemaBasedMultiTenantConnectionProvider implements MultiTenantConnectionProvider, Stoppable,
Configurable, ServiceRegistryAwareService   {
	
	private static final long serialVersionUID = 1L;
	private final C3P0ConnectionProvider connectionProvider = new C3P0ConnectionProvider();
	private final C3P0ConnectionProvider connectionProvider2 = new C3P0ConnectionProvider();
	
	private final Map<String,C3P0ConnectionProvider> tenantIdConnMap = new HashMap<String,C3P0ConnectionProvider>();

	private C3P0ConnectionProvider getProvider(){
		String tenantIdentifier = ThreadLocalUtil.tendantId.get();
		tenantIdentifier = tenantIdentifier.split(":")[0];
		return tenantIdConnMap.get(tenantIdentifier);
	}
	
	@Override
	public Connection getAnyConnection() throws SQLException {
		return getProvider().getConnection();
	}

	@Override
	public void releaseAnyConnection(Connection connection) throws SQLException {
		getProvider().closeConnection(connection);
	}

	@Override
	public Connection getConnection(String tenantIdentifier) throws SQLException {
		//ThreadLocalUtil.tendantId.set(tenantIdentifier);
		tenantIdentifier = tenantIdentifier.split(":")[1];
		final Connection connection = getAnyConnection();
		try {
			connection.createStatement().execute("USE " + tenantIdentifier);
		} catch (SQLException e) {
			throw new HibernateException("Could not alter JDBC connection to specified schema [" + tenantIdentifier
					+ "]", e);
		}
		return connection;
	}

	@Override
	public void releaseConnection(String tenantIdentifier, Connection connection) throws SQLException {
		try {
			connection.createStatement().execute("USE test");
		} catch (SQLException e) {
			throw new HibernateException("Could not alter JDBC connection to specified schema [" + tenantIdentifier
					+ "]", e);
		}
		getProvider().closeConnection(connection);
	}

	@Override
	public boolean isUnwrappableAs(Class unwrapType) {
		return this.getProvider().isUnwrappableAs(unwrapType);
	}

	@Override
	public <T> T unwrap(Class<T> unwrapType) {
		return this.getProvider().unwrap(unwrapType);
	}

	@Override
	public void stop() {
		this.getProvider().stop();
	}

	@Override
	public boolean supportsAggressiveRelease() {
		return this.getProvider().supportsAggressiveRelease();
	}

	@SuppressWarnings({ "unchecked", "rawtypes" })
	@Override
	public void configure(Map configurationValues) {
		
		//connectorProvider初始化
		this.connectionProvider.configure(configurationValues);
		
		configurationValues.put("hibernate.connection.url", "jdbc:mysql://{db-server-url}:3306/dbname?useUnicode=true&amp;characterEncoding=utf8");
		configurationValues.put("hibernate.connection.password", "password");
		this.connectionProvider2.configure(configurationValues);
		
		//connectorProvider与tenantId的关系映射
		tenantIdConnMap.put("dataSource1", connectionProvider);
		tenantIdConnMap.put("dataSource2", connectionProvider2);
		

	}

	@Override
	public void injectServices(ServiceRegistryImplementor serviceRegistry) {
		connectionProvider.injectServices(serviceRegistry);
		connectionProvider2.injectServices(serviceRegistry);
	}

}

```
和引文不同的是，这里直接使用的是C3P0ConnectionProvider,而不是DriverManagerConnectionProviderImpl，DriverManagerConnectionProviderImpl自己实现了一个连接池，但我觉得使用C3P0应该才是正道。

引文中只创建了一个connectionProvider，我在这里创建了两个，其实可以是N个。然后再在getProvider()方法中来确定使用哪个connectionProvider。OK这其实就是多个数据源的实现。

多个数据库的实现在方法getConnection()里，可以看到，在切换数据库的时候就是简单地使用了一个USE而已。

这个代码现在存在的问题在configure(）方法里。我现在还没考虑好如何根据业务需求来创建N个connectionProvier并且做好映射，这里仅仅是为了试验，仓促地构建了另外一个connectionProvider，它和配置的connectionProvider的区别仅仅是url和password不一样。

最后，injectServices方法里，一定要为每一个connectionProvider都injectServices一下。

接下来看看Hibernate配置：

```
<bean id="sessionFactory"
		class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
		<!-- 
		<property name="dataSource" ref="dataSource"/>
		-->
		<property name="hibernateProperties">
			<props>
				<!-- C3P0的配置 -->
				<prop key="c3p0.jdbcUrl" >${jdbc.url}</prop>
				<prop key="c3p0.user" >${jdbc.username}</prop>
				<prop key="c3p0.password"  >${jdbc.password}</prop>
				<prop key="c3p0.preferredTestQuery" >${preferredTestQuery}</prop>
				<prop key="c3p0.idleConnectionTestPeriod" >${idleConnectionTestPeriod}</prop>
				<prop key="c3p0.testConnectionOnCheckout" >${testConnectionOnCheckout}</prop>
				<!-- 数据源配置 -->
				<prop key="hibernate.connection.url">jdbc:mysql://localhost:3306/dbname?useUnicode=true&amp;characterEncoding=utf8</prop>
				<prop key="hibernate.connection.username">root</prop>
				<prop key="hibernate.connection.password">root</prop>
				<prop key="hibernate.connection.driver_class">com.mysql.jdbc.Driver</prop>
				<prop key="hibernate.connection.autocommit">false</prop>
				<!-- Hibernate配置 -->
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
				<!-- 多租户配置 	-->
				<prop key="hibernate.multiTenancy">SCHEMA</prop>
				<prop key="hibernate.tenant_identifier_resolver">com.xyz.TenantIdResolver</prop>
				<prop key="hibernate.multi_tenant_connection_provider">com.xyz.SchemaBasedMultiTenantConnectionProvider</prop>
			</props>
		</property>
		<property name="packagesToScan" value="com.xyz.*.entity"/>
	</bean>
```
因为使用了C3P0，这里增加了相应的配置。另外注意一下，我这里原先配置的dataSource被废掉了，应为是我们自己创建C3P0的ConnectionProvider。如果看过引文了，那么下面的多租户配置就不多说了吧。

***后话***
其中还有很多不够完善的地方，等想好了再修改补充吧。
接下来应该考虑如何按照业务需要来初始化不同的ConnectionProvider了。如果这一切都做好了，还应该考虑考虑多租户环境下，如何实现用户的登录逻辑。
