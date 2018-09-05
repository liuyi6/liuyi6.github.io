---
title: mybatis逆向工程与分页在springboot中的应用
date: 2018-09-06 23:26:19
tags: [springboot,mybatis,分页,逆向工程]
categories: SpringBoot
---
最近在项目中应用到springboot与mybatis，在进行整合过程中遇到一些坑，在此将其整理出来，便于以后查阅与复习。
项目运行环境为：eclispe+jdk1.8+maven
搭建Spring Boot环境
===
首先建立maven project,在生成的pom文件中加入依赖，代码如下：

	<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.0</version>
        </dependency>
        
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <!--分页插件-->
		<dependency>
		      <groupId>com.github.pagehelper</groupId>
		      <artifactId>pagehelper-spring-boot-starter</artifactId>
		      <version>1.2.1</version>
		</dependency>
        
        <!-- alibaba的druid数据库连接池 -->
        <dependency>
        	<groupId>com.alibaba</groupId>
        	<artifactId>druid</artifactId>
        	<version>1.0.29</version>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <!-- mybatis generator 自动生成代码插件 -->
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.2</version>
                <configuration>
                    <configurationFile>${basedir}/src/main/resources/generator/generatorConfig.xml</configurationFile>
                    <overwrite>true</overwrite>
                    <verbose>true</verbose>
                </configuration>
            </plugin>
        </plugins>
    </build>
	
配置好依赖后进行maven install,此时注意的坑：
坑一：启动maven install报错：

	[ERROR] Failed to execute goal org.springframework.boot:spring-boot-maven-plugin:1.5.2.RELEASE:repackage (default) on project springboot-mybatis: Execution default of goal org.springframework.boot:spring-boot-maven-plugin:1.5.2.RELEASE:repackage failed: Unable to find main class -> [Help 1]
	[ERROR] 
	[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
	[ERROR] Re-run Maven using the -X switch to enable full debug logging.
	[ERROR] 
	[ERROR] For more information about the errors and possible solutions, please read the following articles:
	[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/PluginExecutionException
报错原因是没有启动类
解决方法:编写启动类Main.java正常运行！

	@SpringBootApplication
	public class Main {
		public static void main(String[] args) {
			SpringApplication.run(Main.class, args);
		}
	}
坑二：启动maven install报错：

	[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.1:compile (default-compile) on project springboot-mybatis: Compilation failure
	[ERROR] No compiler is provided in this environment. Perhaps you are running on a JRE rather than a JDK?
	[ERROR] -> [Help 1]
	[ERROR] 
	[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
	[ERROR] Re-run Maven using the -X switch to enable full debug logging.
	[ERROR] 
	[ERROR] For more information about the errors and possible solutions, please read the following articles:
	[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoFailureException
报错原因：项目的java环境与电脑环境不符合，例如我的新建项目运行环境为J2SE-1.5,报错是在我将其改为jre1.8之后
解决方案：
右键项目——build path——Configure build path——Libraries——双击Jre System Libraries如下图所示：
![](/images/2018-09/eror_erivorment_project_jdk.png)
选择Alternate JRE 如2处的下拉框只有jre，点击3处的Install JREs，依次经过add——Standard VM——next——Directory,选择本机的jdk位置点击finish
在install JREs位置将默认勾选更改jdk,如下图所示，并保存。
![](/images/2018-09/install_jres_default_jdk.png)
再次maven install项目正常。如下所示：

	[INFO] BUILD SUCCESS
	[INFO] ------------------------------------------------------------------------
	[INFO] Total time: 3.351 s
	[INFO] Finished at: 2018-09-05T21:20:48+08:00
	[INFO] Final Memory: 23M/181M
	[INFO] ------------------------------------------------------------------------
在src/main/resources新建文件：application.yml，内容如下所示：

	server:
	  port: 8080

	spring:
		datasource:
			name: test
			url: jdbc:mysql://localhost:3306/test
			username: root
			password: 123456
			driver-class-name: com.mysql.jdbc.Driver

	## 该配置节点为独立的节点
	mybatis:
	  mapper-locations: classpath:mapper/*.xml
	  
	#pagehelper分页插件
	pagehelper:
		helperDialect: mysql
		reasonable: true
		supportMethodsArguments: true
		params: count=countSql
至此springboot环境搭建完毕！
逆向工程应用
===
首先应该注意到在pom文件中有配置逆向工程xml文件的位置：src/main/resources/generator/generatorConfig.xml
因而在src/main/resources下新建generator文件夹，并建立generatorConfig.xml文件，代码如下：

	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE generatorConfiguration PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN" "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
	<generatorConfiguration>
		<!-- 数据库驱动:选择你的本地硬盘上面的数据库驱动包-->
		<classPathEntry  location="E:\plugins\maven\repo\mysql\mysql-connector-java\5.1.41\mysql-connector-java-5.1.41.jar"/>
		<context id="DB2Tables"  targetRuntime="MyBatis3">
			<commentGenerator>
				<property name="suppressDate" value="true"/>
				<!-- 是否去除自动生成的注释 true：是 ： false:否 -->
				<property name="suppressAllComments" value="true"/>
			</commentGenerator>
			<!--数据库链接URL，用户名、密码 -->
			<jdbcConnection driverClass="com.mysql.jdbc.Driver" connectionURL="jdbc:mysql://localhost/test" userId="root" password="123456">
			</jdbcConnection>
			<javaTypeResolver>
				<property name="forceBigDecimals" value="false"/>
			</javaTypeResolver>
			<!-- 生成pojo类的位置-->
			<javaModelGenerator targetPackage="com.luis.entity" targetProject="src/main/java">
				<property name="enableSubPackages" value="true"/>
				<property name="trimStrings" value="true"/>
			</javaModelGenerator>
			<!-- 生成映射文件的包名和位置-->
			<sqlMapGenerator targetPackage="mapper" targetProject="src/main/resources">
				<!-- enableSubPackages 是否让schema作为包的后缀-->
				<property name="enableSubPackages" value="false"/>
			</sqlMapGenerator>
			<!-- 生成mapper接口的位置-->
			<javaClientGenerator type="XMLMAPPER" targetPackage="com.luis.mapper" targetProject="src/main/java">
				<!-- enableSubPackages 是否让schema作为包的后缀-->
				<property name="enableSubPackages" value="false"/>
			</javaClientGenerator>
			<!-- 指定数据库表 -->
			<table schema="" tableName="user"></table>
		</context>
	</generatorConfiguration>
根据个人环境将配置文件中的配置进行更改，如数据库密码，包名，对应数据库表
所用的数据库表如下：

	DROP TABLE IF EXISTS `user`;
	CREATE TABLE `user` (
	  `id` bigint(20) NOT NULL,
	  `name` varchar(255) NOT NULL,
	  `age` int(4) NOT NULL,
	  PRIMARY KEY (`id`)
	) ENGINE=InnoDB DEFAULT CHARSET=utf8;
	
	INSERT INTO `user` VALUES ('1', 'wanger', '22');
	INSERT INTO `user` VALUES ('2', 'zhangsan', '18');
	INSERT INTO `user` VALUES ('3', 'lisi', '23');
	INSERT INTO `user` VALUES ('4', 'wangwu', '21');
配置完成后，右键项目，选择run as——Maven build——在下面两处分别填入：
Goals: 		mybatis-generator:generate -e
Profiles:	generatorConfig.xml
如下图所示：
![](/images/2018-09/mybatis_springboot_product_code.png)
出现如下所示，代码生成成功，刷新项目即可。

	[INFO] Generating Example class for table user
	[INFO] Generating Record class for table user
	[INFO] Generating Mapper Interface for table user
	[INFO] Generating SQL Map for table user
	[INFO] Saving file UserMapper.xml
	[INFO] Saving file UserExample.java
	[INFO] Saving file User.java
	[INFO] Saving file UserMapper.java
	[INFO] ------------------------------------------------------------------------
	[INFO] BUILD SUCCESS
	[INFO] ------------------------------------------------------------------------
需要注意的是：逆向工程生成的代码不会覆盖，因而不能重复多次生成。
此处也有个小坑，逆向工程的代码生成后，启动项目会报如下错误：

	***************************
	APPLICATION FAILED TO START
	***************************

	Description:

	Field userMapper in com.luis.service.impl.UserServiceImpl required a bean of type 'com.luis.mapper.UserMapper' that could not be found.


	Action:

	Consider defining a bean of type 'com.luis.mapper.UserMapper' in your configuration.
解决方案：
1、给生成的mapper接口文件前加注解：@Mapper 即可解决。但需要给每一个mapper文件前加，繁琐，因而有第二种类解决办法
2、在启动类前加@MapperScan({"com.luis.mapper"})，其中com.luis.mapper为mapper文件的所在位置。
参考自：http://412887952-qq-com.iteye.com/blog/2392672

分页应用
===
逆向工程已经生成了entity类，及dao层的mapper接口与*mapper.xml文件，因而/只用编写service层与web层。
首先在UserService中编写接口，代码如下：

	public interface UserService {
		User selectByName(String name);
		List<User> findAllUser(int pageNum, int pageSize);
	}
在UserServiceImpl文件进行实现，代码如下所示：

	@Service
	public class UserServiceImpl implements UserService {

		@Autowired
		private UserMapper userMapper;
		
		@Override
		public User selectByName(String name) {
			UserExample example = new UserExample();
			Criteria criteria = example.createCriteria();
			criteria.andNameEqualTo(name);
			List<User> users = userMapper.selectByExample(example);
			if (users != null && users.size() > 0) {
				return users.get(0);
			}
			return null;
		}
		
		/**
		 * pageNum 开始页数
		 * pageSize 每页显示的数据条数
		 */
		@Override
		public List<User> findAllUser(int pageNum, int pageSize) {
			//将参数传给方法实现分页
			PageHelper.startPage(pageNum, pageSize);
			UserExample example = new UserExample();
			List<User> list = userMapper.selectByExample(example);
			return list;
		}
	}
最后在controller层对查询结果进行接收，UserController代码如下：

	@Controller
	@RestController
	public class UserController {
		
		@Autowired
		private UserService userService;
		
		@RequestMapping("/test")
		public User querUserByName() {
			User user = userService.selectByName("luis");
			System.out.println(user.toString());
			return user;
		}
		
		@RequestMapping("/list")
		public List<User> querUser() {
			List<User> list = userService.findAllUser(1, 2);
			//获取分页信息
			PageInfo<User> pageInfo = new PageInfo<>(list);
			System.out.println("total:" + pageInfo.getTotal());
			System.out.println("pages:" + pageInfo.getPages());
			System.out.println("pageSize:" + pageInfo.getPageSize());
			return list;
		}
	}
测试
===
此前在项目编写过程中已经对可能出现的错误进行了总结，最后，对项目的功能进行测试，通过加@RestController注解将数据传输到浏览器中。
测试mybatis与springboot，浏览器输入http://localhost:8080/test，浏览器输出：

	{"id":1,"name":"wanger","age":22}
分页测试，浏览器输入http://localhost:8080/list，浏览器输出：

	[{"id":1,"name":"wanger","age":22},{"id":2,"name":"zhangsan","age":18}]
eclipse输出：

	total:4
	pages:2
	pageSize:2
项目搭建完毕，具体代码参见[github](https://github.com/liuyi6/JavaWeb/tree/master/springboot-mybatis)

本文参考了：
http://412887952-qq-com.iteye.com/blog/2392672
https://blog.csdn.net/rico_rico/article/details/79408474
https://blog.csdn.net/winter_chen001/article/details/77249029