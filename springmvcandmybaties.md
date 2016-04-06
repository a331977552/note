 #1配置
##web.xml	
	<!-- 加载spring容器, 就是加载beans-*.xml的所有文件 -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/classes/spring/beans-*.xml</param-value>
	</context-param>
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>

	  <servlet>
	  	<servlet-name>dispatcherServlet</servlet-name>
	  	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	 	<init-param>
	 		<param-name>contextConfigLocation</param-name>
	 		<param-value>classpath:springmvc.xml</param-value>
	 	</init-param>
	  </servlet>
	  <servlet-mapping>
	  <servlet-name>dispatcherServlet</servlet-name>
		<!--
			*.action 解析以.action 结尾的地址
				
			/ 所有地址都解析 而静态文件的解析需要配置不让dispathcherServlet进行解析.  使用这种方式可以实现RESTful风格的url
			/*   这种配置不正确,使用这种配置 最终要转发到一个js页面
				仍然会有dispatcherServelet解析jsp地址,不能根据jsp页面找到handler会报错
		-->
	  <url-pattern>/</url-pattern>
	  </servlet-mapping>
	<!--编码处理过滤器-->
	<filter>
		<filter-name>CharacterEncodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>utf-8</param-value>
		</init-param>
	</filter>
	

##Beans-dao.xml
	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://www.springframework.org/schema/beans" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop" xmlns:mvc="http://www.springframework.org/schema/mvc"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
     http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
	http://www.springframework.org/schema/mvc
	http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd
	http://www.springframework.org/schema/aop 
	http://www.springframework.org/schema/aop/spring-aop-4.2.xsd
	http://www.springframework.org/schema/context 
	http://www.springframework.org/schema/context/spring-context-4.2.xsd
	http://www.springframework.org/schema/tx
	http://www.springframework.org/schema/tx/spring-tx-4.2.xsd
	">
	<context:property-placeholder location="classpath:db.properties" />
	

	<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource">
		<property name="driverClassName" value="${jdbc.driver}"></property>
		<property name="url" value="${jdbc.url}"></property>
		<property name="username" value="${jdbc.username}"></property>
		<property name="password" value="${jdbc.password}"></property>
		<property name="maxTotal" value="10"></property>
		<property name="maxIdle" value="5"></property>
	</bean>

	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="configLocation" value="classpath:mybatis/SqlmappingConfig.xml"></property>
		<property name="dataSource" ref="dataSource"></property>
	</bean>

	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<property name="basePackage" value="com.go.test.dao"></property>
		<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>
	</bean>
	</beans>

##beans-transaction.xml
	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xmlns="http://www.springframework.org/schema/beans" xmlns:context="http://www.springframework.org/schema/context"
		xmlns:aop="http://www.springframework.org/schema/aop" xmlns:mvc="http://www.springframework.org/schema/mvc"
		xmlns:tx="http://www.springframework.org/schema/tx"
		xsi:schemaLocation="http://www.springframework.org/schema/beans
	     http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
	http://www.springframework.org/schema/mvc
	http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd
	http://www.springframework.org/schema/aop 
	http://www.springframework.org/schema/aop/spring-aop-4.2.xsd
	http://www.springframework.org/schema/context 
	http://www.springframework.org/schema/context/spring-context-4.2.xsd
	http://www.springframework.org/schema/tx
	http://www.springframework.org/schema/tx/spring-tx-4.2.xsd
	">
		<!--事物管理器-->
		<bean  id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
			<property name="dataSource" ref="dataSource"></property>
		</bean>
	
		<!--哪些方法需要执行事物-->
		<tx:advice transaction-manager="transactionManager" id="txAdvice">
			<tx:attributes>
				<tx:method name="save*" propagation="REQUIRED"/>
				<tx:method name="update*" propagation="REQUIRED"/>
				<tx:method name="insert*" propagation="REQUIRED"/>
				<tx:method name="delete*" propagation="REQUIRED"/>
											<!--不是必须的.只读-->
				<tx:method name="find*" propagation="SUPPORTS" read-only="true"/>
				<tx:method name="get*" propagation="SUPPORTS" read-only="true"/>
				<tx:method name="select*" propagation="SUPPORTS" read-only="true"/>
			</tx:attributes>
		</tx:advice>

			<!--对 com.go.test.service.imp包下的所有类,所有方法实施事物控制-->
		<aop:config >
			<aop:advisor advice-ref="txAdvice"
				pointcut="execution (* com.go.test.service.imp.*.*(..))"
			/> 
		</aop:config>
	
	</beans>


##springmvc.xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://www.springframework.org/schema/beans" xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop" xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
    http://www.springframework.org/schema/mvc
    http://www.springframework.org/schema/mvc/spring-mvc-4.2.xsd
    http://www.springframework.org/schema/aop 
    http://www.springframework.org/schema/aop/spring-aop-4.2.xsd
    http://www.springframework.org/schema/context 
    http://www.springframework.org/schema/context/spring-context-4.2.xsd
    http://www.springframework.org/schema/tx
    http://www.springframework.org/schema/tx/spring-tx-4.2.xsd
    ">
    
    <mvc:annotation-driven conversion-service="conversionService"></mvc:annotation-driven>
    
    <context:component-scan base-package="com.go.test.controll"></context:component-scan>
    <!-- 视图解析器, 默认会加载 解析 jstl view 以下配置是加载 前缀为 /WEB-INF/page/ 和后缀为.jsp -->
    <bean
    class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/page/" />
    <property name="suffix" value=".jsp" />
    </bean>
    <bean id="conversionService"  class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
    <property name="converters">
    <list>
    <!--自定义转换器-->
    <bean  class="com.go.test.converter.DateConverter"></bean>
    </list>
    </property>
    </bean>
    </beans>
---------
###参数绑定
1. pojo绑定:   		  item.name  形参是 Item item 就可以绑定 

2. 数组绑定:   		  直接 绑定. id=1,id=2,  Integer[] id;

3. list<pojo>绑定:      则在Vo中声明List<item>  itemList  形参填写 ItemVo itemVo  页面定义:  itemList[${state.index}].id=1

4. Map<pojo>绑定:  		则在Vo中声明Map<String,item>  itemMap  形参填写 ItemVo itemVo  页面定义:  itemList['price'].id=1

#利用好第一步加mybatis.xml 就整合完成了
--------------------------



#2.非注解的映射器和适配器
##      非注解的映射器	
	<!---->
	<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"></bean>
	<!---->
	<bean id="myHander"  name="/myfirst.action" class="com.springmvc.test.bean.handler.myHander"></bean>		
	<bean   class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
	<property name="mappings">
		<props >
			<prop key="/myfirst1.action">myHander</prop>
			<prop key="/myfirst2.action">myHander</prop>
		</props>
	</property>
	</bean>

##      非注解的适配器
	<!--要求编写的handler 实现 HttpRequestHandler-->
	<bean  class="org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter"></bean>
	<!--要实现controller 接口-->
	<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"></bean>

##    注解的映射器
		<bean  class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"></bean>
##    注解的适配器
		<bean  class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"></bean>
		<!--或者这个-->	
		<mvc:annotation-driven></mvc:annotation-driven>

#视图解析器 
	<!-- 视图解析器, 默认会加载 解析 jstl view    以下配置是加载  前缀为 /WEB-INF/page/ 和后缀为.jsp -->
	<bean  class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/WEB-INF/page/"/>
		<property name="suffix" value=".jsp"/>
	</bean>

##自动注解开发(直接用这个,前面都是过程,当然要加上视图解析器)

	<!-- 扫描. 这样就不用再配controller了 -->
	<context:component-scan base-package="com.springmvc.test.bean.handler"/>
		<!-- 加上这句话 就不用再配置 注解适配器和映射器了 而且他还会加载一堆有用的组件 -->
	<mvc:annotation-driven></mvc:annotation-driven>

----------------------
###controll开发
	//加Controller
	@Controller
	public class AnnotationController1 {

		//加这个
	@RequestMapping("/queryItems.action")
	public ModelAndView queryItems(){
		
	List<OrderDetails> details=new ArrayList<>();
		for(int i =0;i<10;i++){
			OrderDetails details2=new OrderDetails();
			details2.goods_count="5";
			details2.goods_price="5000";
			details2.items_id="1"+i;
			details2.orderId="10";
			
			details.add(details2);			
		}
		ModelAndView view=new ModelAndView();
		view.addObject("list",details);
		view.setViewName("/WEB-INF/page/details.jsp");
			return view;
		}

--------
	//声明了mapper以后,加载自动扫描的组件,就可以自动装载了.不用再getBean
	@Autowired
	ItemsMapperCustom itemsMapperCustom;

------
	//或者
	@Controller
	//对url进行分类管理
	@RequestMapping("/items")
	public class Myhandler  implements HttpRequestHandler{
	//实际请求的路径是 /items/dppppp
	 //可以用method字段控制请求方法
	//	@RequestMapping(value="/queryItem",method={RequestMethod.POST,RequestMethod.GET})

	@RequestMapping("dpppppp")
	@Override
	public void handleRequest(HttpServletRequest arg0, HttpServletResponse arg1) throws ServletException, IOException {
		System.out.println("aaaaaaaaa");
		arg0.getRequestDispatcher("/WEB-INF/page/index.jsp").forward(arg0, arg1);
	}

----
	//返回字符串类型		
	@RequestMapping("queryItemsById")        //自动参数注入,根据映射注入
	public String queryItemsById(Model model,@RequestParam(value="id",defaultValue="1",required=true)String item_id){

		model.addAttribute("items", items);
		//返回到 前缀+success+后缀页面
		return "success";
		//重定向到 当前的controll @RequestMapping value=queryItem 中;
		//return "redirect:queryItem";
		//页面转发到 当前的controll @RequestMapping value=queryItem 中;
		//return "forward:queryItem";
		
	}




-------
##事物控制
		<bean  id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
			<property name="dataSource" ref="dataSource"></property>
		</bean>
		<tx:advice transaction-manager="transactionManager" id="txAdvice">
			<tx:attributes>
				<tx:method name="save*" propagation="REQUIRED"/>
				<tx:method name="update*" propagation="REQUIRED"/>
				<tx:method name="insert*" propagation="REQUIRED"/>
				<tx:method name="delete*" propagation="REQUIRED"/>
				<tx:method name="find*" propagation="SUPPORTS" read-only="true"/>
				<tx:method name="get*" propagation="SUPPORTS" read-only="true"/>
				<tx:method name="select*" propagation="SUPPORTS" read-only="true"/>
			</tx:attributes>
		</tx:advice>
		
		<aop:config >
			<aop:advisor advice-ref="transactionManager" 
				pointcut="execution (* com.go.test.service.imp.*.(..))"
			/>
		</aop:config>


####pojo参数的绑定
	在input域中的参数名称,要跟poji的属性名称一致,就可以映射成功



##写接口
	
	response.setCharacterEncoding("UTF8");
		response.setContentType("application/json;charset=utf-8");
		response.getWriter().write("啊啊啊啊啊啊啊啊啊啊啊啊啊啊"
				+ "");



--------------------------------------------







# mybatis
	
##1 log4j.properties
	###############################log4j.properties###############################
	##### Global Log Level(OFF,FATAL,ERROR,WARN,INFO,DEBUG,ALL) #############
	log4j.rootLogger=DEBUG,STDOUT
	###### STDOUT Logger ###############
	log4j.appender.STDOUT=org.apache.log4j.ConsoleAppender
	#输出目的Appender的日志级别，Appender的级别设置要优先于logger的
	#级别设置，即先使用Appender的，而不管logger的日志级别是怎样设置的
	log4j.appender.STDOUT.layout=org.apache.log4j.PatternLayout
	log4j.appender.STDOUT.layout.ConversionPattern=[%p] [%l] %10.10c - %m%n



##2 SqlMappingConfig.xml
		<?xml version="1.0" encoding="UTF-8" ?>  
	<!DOCTYPE configuration  
	  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"  
	  "http://mybatis.org/dtd/mybatis-3-config.dtd">  
	<configuration>  
	    <!-- 对事务的管理和连接池的配置 -->  
	    <environments default="development">  
	        <environment id="development">  
	            <transactionManager type="JDBC" />  
	            <dataSource type="POOLED">  
	                <property name="driver" value="com.mysql.jdbc.Driver" />  
	                <property name="url" value="jdbc:mysql://localhost:3306/shop" />  
	                <property name="username" value="root" />  
	                <property name="password" value="123456" />  
	            </dataSource>  
	        </environment>  
	    </environments>  
	      
	    <!-- mapping 文件路径配置 -->  
	    <mappers>  
	        <mapper resource="com/yu/res/UserMapper.xml" />  
	    </mappers>  
	</configuration>

##3 mapping.xml
	<?xml version="1.0" encoding="UTF-8" ?>  
	<!DOCTYPE mapper
	    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
	    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
	<mapper namespace="test">
	    <select id="findUserByid" parameterType="int"  resultType="com.mybaties.test.po.User" >
	    select * from user where  id=#{id}
	    </select>
	</mapper>

###获取session

		InputStream resourceAsStream = Resources.getResourceAsStream("SqlmappingConfig.xml");
				SqlSessionFactory build = new SqlSessionFactoryBuilder().build(resourceAsStream);
				SqlSession openSession = build.openSession();
###获取自增主键   keyProperty="id" id是bean的 字段会自动填充
	<insert id="insertUser" parameterType="com.mybaties.test.po.User">
		<selectKey order="AFTER" resultType="int" keyProperty="id">
			select last_insert_id()
		</selectKey>
		insert into
		user(username,birthday,sex,address)values(#{username},#{birthday},#{sex},#{address})
	</insert>
###获取非自增主键id  order 是在insert 语句之前执行,填充到userbean中,然后再设置insert语句中
	<insert id="insertUser" parameterType="com.mybaties.test.po.User">
		<selectKey order="BEFORE" resultType="int" keyProperty="id">
			select UUID()
		</selectKey>
		insert into
		user(id,username,birthday,sex,address)values(#(id),#{username},#{birthday},#{sex},#{address})
	</insert>


###利用mapper 对象代理自定生成dao 
	namespace="com.mybaties.test.mapper.UserDao" //namespace 就是dao的全县定名
	里面的各种类型类型和xml的类型是对应的. id="finduserById"则是 方法名字. parameterType="sss" sss就是 方法参数.
	resultType 就是方法的返回值.
	调用的时候 是 用 openSession.getMapper(UserDao.class); 获取dao层实例. 然后调用相应方法就可以实现功能.
	
###sql动态语句:
	![](http://i.imgur.com/M4dp7EW.png)

###懒加载
			 
			<settings>
			<!--开启加载-->
			<setting name="lazyLoadingEnabled" value="true"/>
			<!--按需加载-->
			<setting name="aggressiveLazyLoading" value="false"/>
			</settings>
			<!--开启缓存-->
###一二级缓存,
		1级缓存 是在同一个sqlsession中 执行查询时会缓存,如果执行的 update insert delete 并且 commit以后则会清空一级缓存
		2级缓存 是在同一个mapper.xml中 执行查询时会缓存,如果执行的 update insert delete 并且 commit以后则会清空二级缓存
		sqlmapconfig中开启   (默认开启)	<setting name="cacheEnabled" value="true"/>
		还需要单独在mapper.xml文件中 单独开启,语法: <cache />
		二级缓存以mapper的namespace做区别,需要让支持二级缓存的poji实现序列化接口.
		因为缓存的介质不同,可以是内存或者硬盘,或者远程服务器
		缓存的数据结构是 hashMap.
		select 的xml字段中可以用 userCache字段 设置为false 来禁用二级缓存. 默认为true;
		<select id="findUserByid" parameterType="int"   resultType="user" usecache="true" >
		update insert delete 的xml字段中,flushCache="true" 来刷新缓存,默认为true, 避免数据库脏读
		<delete id="deleteUserById"  parameterType="int" flushCache="true">		
###整合ehcache
	<cache type="ehcache实现mybatis cache接口的类全限定名"/>
	还要加入ehcache的xml配置文件


###mybatis-spring整合
#####SqlmappingConfig.xml中 基本只写别名,
#####在写原始dao时	SqlmappingConfig配置mapper
	<mappers>
		<mapper resource="UserMapping.xml"/>
	</mappers>

在写mapperDao时 SqlmappingConfig不用配置mapper,也不用配置扫描包,全在spring 配置完成
	
####beans.xml
#####配置数据源(!)  该应用配置的是dhcp数据源  
	<!--加载数据库配置项-->
	<context:property-placeholder location="classpath:db.properties"/>
	<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource">
		<property name="driverClassName" value="${jdbc.driver}"></property>
		<property name="url" value="${jdbc.url}"></property>
		<property name="username" value="${jdbc.username}"></property>
		<property name="password" value="${jdbc.password}"></property>
		<property name="maxTotal" value="10"></property>
		<property name="maxIdle" value="5"></property>
	</bean>
####配置 mybatis的 sqlSessionfactory(!)
	<!--配置 mybatis的 sqlSessionfactory,注入配置文件.和数据源-->
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="configLocation" value="mybatis/SqlmappingConfig.xml"></property>
		<property name="dataSource" ref="dataSource"></property>
	</bean>

###开发原始dao
	<!--开发原始dao,注入sessionfactory, 让userDao  extends  SqlSessionDaoSupport 就可以使用 getSqlSession(); -->
	<bean class="com.mybatis.test.dao.UserDaoImp" id="userdao">
		<property name="sqlSessionFactory" ref="sqlSessionFactory"></property>
	</bean>
	

##开发mapper dao(!)
		把UserMapper.java和UserMapper.xml 放在一个包下. 同理多个类 放置多个xml文件 xml namespace必须为对应的全类名
		SqlmappingConfig 不用设置任何东西
			<!--导入扫描mapperdap的 类.  告诉 包地址,自动扫描,  并且告诉sqlsessionfactory 如果有多个包, 就用 (半月),隔开 -->
			<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
			<property name="basePackage" value="com.mybatis.test.dao"></property>
			<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>



#逆向工程
#### 根据表自动创建mapper,mapper.xml,pojo ####
	//需要运行的代码
			List<String> warnings = new ArrayList<String>();
		   boolean overwrite = true;
		   File configFile = new File("generatorConfig.xml");
		   ConfigurationParser cp = new ConfigurationParser(warnings);
		   Configuration config = null;
		
			try {
			config = cp.parseConfiguration(configFile);
			   DefaultShellCallback callback = new DefaultShellCallback(overwrite);
			   MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
			   myBatisGenerator.generate(null);
		
			} catch (Exception e){
				e.printStackTrace();
			}

----------------
#### xml文件. ####

		<?xml version="1.0" encoding="UTF-8"?>
		<!DOCTYPE generatorConfiguration
		  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
		  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
		
		<generatorConfiguration>
		
		  <context id="DB2Tables" targetRuntime="MyBatis3">
		  <commentGenerator>
				<!--不生成注释-->
				<property name="suppressAllComments" value="true"/>
			</commentGenerator>
		    <jdbcConnection driverClass="com.mysql.jdbc.Driver"
		        connectionURL="jdbc:mysql://localhost:3306/shop"
		        userId="root"
		        password="123456">
		    </jdbcConnection>
			<!--false代表int类型自动圣城integer类型,true代表生成bigdecimal-->
		    <javaTypeResolver >
		      <property name="forceBigDecimals" value="false" />
		    </javaTypeResolver>
			<!--生成的po路径-->
		    <javaModelGenerator targetPackage="com.go.test.po" targetProject=".\src">
		      <property name="enableSubPackages" value="true" />
		      <property name="trimStrings" value="true" />
		    </javaModelGenerator>
				<!--生成的的mapper.java路径-->
		    <sqlMapGenerator targetPackage="com.go.test.dao"  targetProject=".\src">
		      <property name="enableSubPackages" value="true" />
		    </sqlMapGenerator>
			<!--生成的的mapper.xml路径-->
		    <javaClientGenerator type="XMLMAPPER" targetPackage="com.go.test.dao"  targetProject=".\src">
		      <property name="enableSubPackages" value="true" />
		    </javaClientGenerator>
			<!--哪些表-->
		    <table  tableName="items" >
		   
		    </table>
		    <table  tableName="order_" >
		   
		    </table>
		    <table  tableName="user" >
		   		
		    </table>
		   
		
		  </context>
		</generatorConfiguration>
//逆向工程需要的jar
mybatis-generator-core-1.3.2.jar

//jdbc连接的jar						
mysql-connector-java-5.1.38-bin.jar

//生成的文件防止报错所需要的jar(可以不用)					
mybatis-3.2.8.jar