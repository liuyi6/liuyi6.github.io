---
title: springboot整合JPA,Swagger
date: 2018-08-02 21:13:19
tags: [SpringBoot]
categories: SpringBoot
---
项目目标
---
前面我们讲了简单的springboot项目的搭建，接下来，我们就将给springboot项目整合组件JPA和Swagger2。
JPA是数据持久化的集合，即数据库操作。JPA的目标是制定多数据实现的API，目前java中提到的JPA一般是由hibernate实现的。
Swagger是一个功能强大的在线API文档框架，目前版本为2.x,故称为swagger2，它提供了在线文档的查阅测试功能，能够方便的构建RESTful风格的API。
项目实现
---
项目环境：eclipse、maven、jdk1.8、mysql数据库
项目结构如下图所示：
![](/images/2018-7-31/springboot_swagger_jpa_project_structure.png)
**项目搭建**
首先，新建maven项目，并在pom文件中引入依赖，springboot的版本，引入springboot项目的起步依赖，再加入JPA依赖与Swagger2依赖。具体代码如下所示：

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    
  	<dependencies>		
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
	        <groupId>org.springframework.boot</groupId>
	        <artifactId>spring-boot-starter-web</artifactId>
	    </dependency>
	    
        <-- jpa依赖 -->
	    <dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>

		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
		</dependency>
		
        <-- swagger2依赖 -->
	    <dependency>
			<groupId>io.springfox</groupId>
			<artifactId>springfox-swagger2</artifactId>
			<version>2.6.1</version>
		</dependency>
		<dependency>
			<groupId>io.springfox</groupId>
			<artifactId>springfox-swagger-ui</artifactId>
			<version>2.6.1</version>
		</dependency>
    </dependencies>
编写启动类Application，代码如下：

    @SpringBootApplication
    public class Application {
        public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
        }
    }
**配置文件**
在resources包中添加文件application.yml,其内容如下：

    spring:
      datasource:
        driver-class-name: com.mysql.jdbc.Driver
        url: jdbc:mysql://localhost:3306/spring-cloud?useUnicode=true&characterEncoding=utf8&characterSetResults=utf8
        username: root
        password: 123456

      jpa:
        hibernate:
          ddl-auto: create  # 第一次建表create  后面用update
        show-sql: true
其中端口名，数据库名，用户名，密码需与本机的数据库名一致。
可新建数据库后将user.sql文件导入。
**配置swagger2**
接下来在com.luis.swagger包中新建类，代码如下：

    @Configuration
    @EnableSwagger2
    public class Swagger2 {
        @Bean
        public Docket createRestApi() {
            return new Docket(DocumentationType.SWAGGER_2)
                    .apiInfo(apiInfo())
                    .select()
                    .apis(RequestHandlerSelectors.basePackage("com.luis.web"))
                    .paths(PathSelectors.any())
                    .build();
        }
        private ApiInfo apiInfo() {
            return new ApiInfoBuilder()
                    .title("springboot利用swagger构建api文档")
                    .description("简单优雅的restfun风格，https://liuyi6.top")
                    .termsOfServiceUrl("https://liuyi6.top")
                    .version("1.0")
                    .build();
        }
    }
**entity层**
    
    @Entity
    public class User {

        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;

        @Column(nullable = false,  unique = true)
        private String username;

        @Column
        private String password;

        public User() {
        }
        public Long getId() {
            return id;
        }
        public void setId(Long id) {
            this.id = id;
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
    }
**dao层**

    public interface UserDao extends JpaRepository<User, Long>{

        User findByUsername(String username);
    }
**service层**
本项目主要构建RESTful风格的API接口，以对User进行操作，Service代码如下：

    @Service
    public class UserService {

        @Autowired
        private UserDao userRepository;

        public User findUserByName(String username){

            return userRepository.findByUsername(username);
        }

        public List<User> findAll(){
           return userRepository.findAll();
        }

        public User saveUser(User user){
          return  userRepository.save(user);
        }

        public User findUserById(long id){
            return userRepository.findOne(id);
        }
        public User updateUser(User user){
           return userRepository.saveAndFlush(user);
        }
        public void deleteUser(long id){
            userRepository.delete(id);
        }
    }
**web层**

    @RequestMapping("/user")
    @RestController
    public class UserController {

        @Autowired
        UserService userService;

        @ApiOperation(value="用户列表", notes="用户列表")
        @RequestMapping(value={""}, method= RequestMethod.GET)
        public List<User> getUsers() {
            List<User> users = userService.findAll();
            return users;
        }

        @ApiOperation(value="创建用户", notes="创建用户")

        @RequestMapping(value="", method=RequestMethod.POST)
        public User postUser(@RequestBody User user) {
          return   userService.saveUser(user);

        }
        @ApiOperation(value="获用户细信息", notes="根据url的id来获取详细信息")

        @RequestMapping(value="/{id}", method=RequestMethod.GET)
        public User getUser(@PathVariable Long id) {
            return userService.findUserById(id);
        }

        @ApiOperation(value="更新信息", notes="根据url的id来指定更新用户信息")
        @RequestMapping(value="/{id}", method= RequestMethod.PUT)
        public User putUser(@PathVariable Long id, @RequestBody User user) {
            User user1 = new User();
            user1.setUsername(user.getUsername());
            user1.setPassword(user.getPassword());
            user1.setId(id);
           return userService.updateUser(user1);

        }
        @ApiOperation(value="删除用户", notes="根据url的id来指定删除用户")
        @RequestMapping(value="/{id}", method=RequestMethod.DELETE)
        public String deleteUser(@PathVariable Long id) {
            userService.deleteUser(id);
            return "success";
        }
    }
项目测试
---
启动工程，在浏览器上访问localhost:8080/swagger-yi.html,浏览器显示Swagger在线文档界面，如下图所示：
![](/images/2018-7-31/springboot_swagger_jpa_show.png)
开发者可通过此界面便捷的查阅，测试项目功能。

附：
swagger常用注解及其说明
使用的版本：springfox-swagger2（2.4）springfox-swagger-ui （2.4）

 注解 | 应用对象 |注解作用
--------|--------|--------
@Api()|用于类|标识类是swagger的资源
@ApiOperation()|用于方法|表示一个http请求操作
@ApiParam()|用于方法，参数，字段说明|表示对参数的添加元数据（说明或是否必填等）
@ApiModel()|用于类|表示对类进行说明，用于参数用实体类接收
@ApiModelProperty()|用于方法，字段|表示对model属性的说明或者数据操作更改
@ApiIgnore()|用于类，方法，方法参数|表示这个方法或者类被忽略
@ApiImplicitParam() |用于方法|表示单独的请求参数
@ApiImplicitParams()| 用于方法|包含多个 @ApiImplicitParam


本文主要参考自《深入理解Spring+Cloud与微服务构建》一书
另参考博客：
https://blog.csdn.net/u014231523/article/details/76522486