#1配置
##Beans.xml
	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xmlns="http://www.springframework.org/schema/beans" 
		xmlns:context="http://www.springframework.org/schema/context"
		xmlns:aop="http://www.springframework.org/schema/aop"
		 xmlns:mvc="http://www.springframework.org/schema/mvc"
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
	<bean  name="/myfirst.action" class="com.springmvc.test.bean.handler.myHander"></bean>
	<!-- 适配器 -->
	<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"></bean>
<!-- 映射器 -->	
	<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"></bean>
</beans>

##web.xml
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
	  <url-pattern>*.action</url-pattern>
	  </servlet-mapping>






#2.非注解的映射器和适配器
##      非注解的映射器
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

##自动注解开发

	<!-- 扫描. 这样就不用再配controller了 -->
	<context:component-scan base-package="com.springmvc.test.bean.handler"/>
		<!-- 加上这句话 就不用再配置 注解适配器和映射器了 而且他还会加载一堆有用的组件 -->
	<mvc:annotation-driven></mvc:annotation-driven>
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
	