# SpringCloud 入门案例



## 模块概览：

<a href="https://sm.ms/image/8TfiulosH5RaSpD" target="_blank"><img src="https://s2.loli.net/2022/04/25/8TfiulosH5RaSpD.png" style="zoom:50%;"  ></a>



## 一、服务提供者和消费者

### 介绍

我们会使用一个Dept部门模块作为Consumer消费者(Client)，通过REST调用Provider提供者(Server)提供的服务。

父工程(Project)下初次带着3个子模块(Module)

- springcloud-api 【entity/接口/公共配置等】
- springcloud-provider-dept-8081 【服务提供者】
- springcloud-consumer-dept-80 【服务消费者】

### 创建工程

#### 创建父工程

新建父工程项目springcloud，定义POM文件，将后续各个子模块公用的jar包等统一提取出来，类似一个抽象父类。

**父工程pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>nuc.ss</groupId>
    <artifactId>springcloud</artifactId>
    <version>1.0-SNAPSHOT</version>
    <modules>

    </modules>

    <!--打包方式 pom-->
    <packaging>pom</packaging>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <junit.version>4.12</junit.version>
        <lombok.version>1.18.12</lombok.version>
        <log4j.version>1.2.17</log4j.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <!--springcloud的依赖-->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Hoxton.SR8</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--springboot的依赖-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.1.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--数据库-->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>8.0.20</version>
            </dependency>
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>1.0.9</version>
            </dependency>
            <!--springboot启动器-->
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>2.1.3</version>
            </dependency>

            <!--日志和测试-->
            <dependency>
                <groupId>ch.qos.logback</groupId>
                <artifactId>logback-core</artifactId>
                <version>1.2.3</version>
            </dependency>
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>${junit.version}</version>
            </dependency>
            <dependency>
                <groupId>log4j</groupId>
                <artifactId>log4j</artifactId>
                <version>${log4j.version}</version>
            </dependency>
            <!--lombok-->
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>${lombok.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
    
    <build>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.yml</include>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.yml</include>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
        </resources>
    </build>
</project>
```



#### 子模块的创建流程

1. 导入依赖
2. 编写配置文件yaml
3. 开启这个功能 @EnableXXXX
4. 配置类



#### 1、创建子模块springcloud-api

**pom配置**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springcloud</artifactId>
        <groupId>nuc.ss</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>sprintcloud-api</artifactId>

    <!--当前的module自己需要的依赖，如果父依赖中已经配置了版本，这里就不用写了-->
    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>
</project>
```

**数据库的创建**

```mysql
CREATE DATABASE `db01`
USE `db01`;
DROP TABLE IF EXISTS `dept`;

CREATE TABLE `dept` (
  `deptno` BIGINT(20) NOT NULL AUTO_INCREMENT,
  `dname` VARCHAR(60) DEFAULT NULL,
  `db_source` VARCHAR(60) DEFAULT NULL,
  PRIMARY KEY (`deptno`)
) ENGINE=INNODB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COMMENT='部门表';


INSERT  INTO `dept`(`dname`,`db_source`) 
VALUES ('开发部',DATABASE()),('人事部',DATABASE()),('财务部',DATABASE()),('市场部',DATABASE()),('运维部',DATABASE());
```

**实体类的编写**

```java
package nuc.ss.springcloud.pojo;

import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.experimental.Accessors;
import java.io.Serializable;

@Data
@NoArgsConstructor
@Accessors(chain = true)   // 支持链式编程
public class Dept implements Serializable {

    private long deptno;//主键
    private String dname;

    //这个数据保存在哪个数据库
    private String db_source;

    public Dept(String dname) {
        this.dname = dname;
    }

    /*
    * 链式写法：
    * Dept dept = new Dept();
    *
    * dept.setDeptNo(11).setDname('ssss').setDb_source('db01')
    * */
}
```

#### 2、子模块springcloud-provider-dept-8081服务的提供者的编写

​	<img src="https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20200924195248477.png" alt="image-20200924195248477" style="zoom:50%;" />

**pom配置：**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springcloud</artifactId>
        <groupId>nuc.ss</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>springcloud-provider-dept-8081</artifactId>

    <dependencies>
        <!--我们需要拿到实体类，所以要配置api module-->
        <dependency>
            <groupId>nuc.ss</groupId>
            <artifactId>springcloud-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!--junit-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>

        <!--test-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-test</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!--jetty-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>

        <!--热部署工具-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
    </dependencies>
</project>
```

**application.yml的配置**

```yaml
server:
  port: 8081

# mybatis的配置
mybatis:
  type-aliases-package: nuc.ss.springcloud.pojo
  config-location: classpath:mybatis/mybatis-config.xml
  mapper-locations: classpath:mybatis/mapper/*.xml

# spring的配置
spring:
  application:
    name: springcloud-provider-dept
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource #数据库
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql//localhost:3306/db01?useUnicode=true&characterEncoding=utf-8
    username: root
    password: admin
```

**mybatis-config.xml的配置**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <settings>
        <setting name="cacheEnabled" value="true"/>
    </settings>
</configuration>
```

**DeptMapper.xml的编写**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="nuc.ss.springcloud.dao.DeptDao">

    <insert id="addDept" parameterType="Dept">
        insert into dept (dname,db_source)
        values (#{dname},DATABASE())
    </insert>

    <select id="queryById" resultType="Dept" parameterType="Long">
        select * from dept where deptno = #{id}
    </select>
    
    <select id="queryAll" resultType="Dept">
        select * from dept
    </select>

</mapper>
```

**接口DeptController的编写**

```java
@RestController
public class DeptController {
    @Autowired
    DeptService deptService;

    @PostMapping("/dept/add")
    public boolean addDept(Dept dept) {
        return deptService.addDept(dept);
    }

    @GetMapping("/dept/get/{id}")
    public Dept get(@PathVariable("id") Long id) {
        return deptService.queryById(id);
    }

    @GetMapping("/dept/list")
    public List<Dept> queryAll() {
        return deptService.queryAll();
    }
}
```

**整体目录结构**

<img src="https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20200925201520700.png" alt="image-20200925201520700" style="zoom:50%;" />

### 3、子模块springcloud-consumer-dept-80的编写

pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springcloud</artifactId>
        <groupId>nuc.ss</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>springcloud-consumer-dept-80</artifactId>

    <dependencies>
        <dependency>
            <groupId>nuc.ss</groupId>
            <artifactId>springcloud-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
</project>
```

application.yml

```yaml
server:
  port: 80
```

**将RestTemplate注册到spring中：**ConfigBean.java

```java
@Configuration
public class ConfigBean {

    @Bean
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

**DeptConsumerController.java**

```java
@RestController
public class DeptConsumerController {
    
    // 消费者不应该有service层
    // RestTemplate供我们直接调用就可以了，将它注册到spring中
    @Autowired
    private RestTemplate restTemplate;//提供多种访问Http的方法

    private static final String REST_URL_PREFIX = "http://localhost:8081";

    @RequestMapping("/consumer/dept/add")
    public boolean add(Dept dept) {
        // (url,实体:Map, Class<T> responseType) 
        return 
           restTemplate.postForObject(REST_URL_PREFIX+"/dept/add",dept,Boolean.class);
    }

    @RequestMapping("/consumer/dept/get/{id}")
    public Dept get(@PathVariable("id") Long id) {
        return restTemplate.getForObject(REST_URL_PREFIX+"/dept/get/"+id,Dept.class);
    }

    @RequestMapping("/consumer/dept/list")
    public List<Dept> list() {
        return restTemplate.getForObject(REST_URL_PREFIX+"/dept/list",List.class);
    }
}
```

**启动服务: **

```java
@SpringBootApplication
public class DeptConsumer_80 {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer_80.class,args);
    }
}
```



## 二、Eureka

### 1.服务注册

#### Eureka-server

1. 创建springcloud-eureka-7001 模块

2. pom.xml

   ```xml
   <dependencies>
       <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-eureka-server -->
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-eureka-server</artifactId>
           <version>1.4.6.RELEASE</version>
       </dependency>
   
       <!--热部署-->
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-devtools</artifactId>
       </dependency>
   </dependencies>
   ```

3. application.yml

   ```yaml
   server:
     port: 7001
   
   # Eureka配置
   eureka:
     instance:
       hostname: localhost # Eureka Service的名字
     client:
       register-with-eureka: false # 是否向Eureka中心注册自己
       fetch-registry: false # 如果为false，则表示自己为注册中心
       # Eureka Service的访问路径
       service-url:
         defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
   ```

4. 主启动类EurekaServer_7001.java

   ```java
   @SpringBootApplication
   @EnableEurekaServer
   public class EurekaServer_7001 {
       public static void main(String[] args) {
           SpringApplication.run(EurekaServer_7001.class,args);
       }
   }
   ```

5. 启动成功后访问 http://localhost:7001/ 得到以下页面，现在还没有注册任何服务

   ![image-20200926110158171](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20200926110158171.png)

#### Eureka-client：将服务注册到Eureka-server

**调整之前创建的springcloud-provider-dept-8081**

**导入Eureka依赖**

```xml
<!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-eureka -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>
```

application.yml中新增Eureka配置

```yaml
# Eureka的配置
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka/
```

为主启动类添加@EnableEurekaClient注解

```java
//启动类
@SpringBootApplication
@EnableEurekaClient //在服务启动后自动注册到Eureka中
public class DeptProvider_8081 {
    public static void main(String[] args) {
        SpringApplication.run(DeptProvider_8081.class,args);
    }
}
```

先启动7001服务端后启动8001客户端进行测试，然后访问监控页http://localhost:7001/ 

![image-20200926120411242](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20200926120411242.png)

**修改Eureka Client上的默认描述信息**

```yaml
# Eureka的配置
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka/
  instance:
    instance-id: springcloud-provider-dept-8081 #修改Eureka上的默认描述信息
```

**结果如图：**

![image-20200926121313373](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20200926121313373.png)



#### 获取注册进来的微服务的信息（团队开发会用到）

DiscoveryClient的作用：可以从注册中心中根据服务别名获取注册的服务器信息。

启动类添加注解`@EnableDiscoveryClient`（服务发现）

```java
//启动类
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
public class DeptProvider_8081 {
    public static void main(String[] args) {
        SpringApplication.run(DeptProvider_8081.class,args);
    }
}
```

服务提供者中的DeptController.java新增方法

```java
//获取一些配置的信息，得到具体的微服务
@Autowired
private DiscoveryClient client;

 //获取一些注册进来的微服务的消息
 @GetMapping("/dept/discovery")
 public Object discovery() {
     //获取微服务列表的清单
     List<String> services = client.getServices();
     System.out.println("discovery=>services:" + services);
     //得到一个具体的微服务信息
     List<ServiceInstance> instances = client.getInstances("SPRINGCLOUD-PROVIDER-DEPT");
     // 遍历输出每个微服务信息
     for (ServiceInstance instance : instances) {
         System.out.println(
                 instance.getHost() + "\t" + // 主机名称
                         instance.getPort() + "\t" + // 端口号
                         instance.getUri() + "\t" + // uri
                         instance.getServiceId() // 服务id
         );
     }
     return this.client;
 }

```

结果

<img src="https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20200927155502206.png" alt="image-20200927155502206" style="zoom:50%;" />



### 2.服务发现

**修改springcloud-consumer-dept-80**

向pom.xml中添加Eureka依赖

```xml
<!--eureka-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>
```

在application.yml文件中配置Eureka

```yaml
# Eureka配置
eureka:
  client:
    register-with-eureka: false
    service-url: # 从三个注册中心中随机取一个去访问
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
```

主启动类加上@EnableEurekaClient注解，开启Eureka（服务的提供方和消费方都需要加@EnableEurekaClient注解）

```java
@SpringBootApplication
@EnableEurekaClient //开启Eureka客户端
public class DeptConsumer_80 {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer_80.class,args);
    }
}
```



### 3.Eureka集群环境配置

#### 1.初始化

新建springcloud-eureka-7002、springcloud-eureka-7003 模块

为pom.xml添加依赖 (与springcloud-eureka-7001相同)

application.yml配置(与springcloud-eureka-7001相同)，端口号分别换成7002和7003

主启动类(与springcloud-eureka-7001相同)

#### 2.集群成员相互关联

配置一些自定义本机名字，找到本机hosts文件并打开，在hosts文件最后加上，要访问的本机名称，默认是localhost

<img src="C:\Users\黄睿楠\AppData\Roaming\Typora\typora-user-images\image-20220417170025257.png" alt="image-20220417170025257" style="zoom:50%;" />

修改Eureka Service的名字

<img src="https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20200927164209557.png" alt="image-20200927164209557" style="zoom:50%;" />

使springcloud-eureka-7001、springcloud-eureka-7002、springcloud-eureka-7003互相关联

以7001为例：

```yaml
server:
  port: 7001

# Eureka配置
eureka:
  instance:
    hostname: eureka7001.com # Eureka服务端的名字
  client:
    register-with-eureka: false
    fetch-registry: false 
    service-url:  #监控页面
      # 集群（关联）：7001关联7002、7003
      defaultZone: http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/

```

springcloud-provider-dept-8081下的yml配置文件：配置服务注册中心地址

```yaml
# Eureka的配置
eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
  instance:
    instance-id: springcloud-provider-dept-8081 #修改Eureka上的默认描述信息
```

这样模拟集群就搭建好了，就可以把一个项目挂载到三个服务器上了

![image-20200927170039729](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20200927170039729.png)



## 三、Ribbon

**springcloud-consumer-dept-80**向pom.xml中添加Ribbon依赖

```xml
<!-- Ribbon -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-ribbon</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>
```

### 1.实现负载均衡

**服务消费者：**

自定义Spring配置类：ConfigBean.java 配置负载均衡实现RestTemplate

```java
@Configuration
public class ConfigBean {   //Cofiguration -- spring applicationContext.xml

    @LoadBalanced //配置负载均衡实现RestTemplate
    @Bean
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }   
}
```

**服务提供者：**

新建两个服务提供者Moudle：springcloud-provider-dept-8082、springcloud-provider-dept-8083

创建db02和db03数据库(与db01一样)

参照springcloud-provider-dept-8081 依次为另外两个Moudle添加pom.xml依赖 、resourece下的mybatis和application.yml配置，Java代码

<font color=red>三个服务（spring.application.name）的名称必须一致</font>

流程图：

![image-20200927193721521](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20200927193721521.png)

启动所有服务测试，访问http://localhost/consumer/dept/list 这时候默认轮询访问的是服务提供者8081

<img src="https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20200927193308957.png" alt="image-20200927193308957" style="zoom:50%;" />

- 再次访问http://localhost/consumer/dept/list这时候默认轮询的是服务提供者8083

  <img src="https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20200927193329021.png" alt="image-20200927193329021" style="zoom:50%;" />

- 再次访问http://localhost/consumer/dept/list这时候默认轮询的是服务提供者8082

  <img src="https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20200927193346522.png" alt="image-20200927193346522" style="zoom:50%;" />



### 2.切换规则

在springcloud-consumer-dept-80模块下的ConfigBean中进行配置，切换使用不同的规则

```java
@Configuration
public class ConfigBean {

    /**
     * IRule:
     * RoundRobinRule 轮询
     * RandomRule 随机
     * AvailabilityFilteringRule ： 会先过滤掉跳闸的服务，对剩下的进行轮询
     * RetryRule ： 会先按照轮询获取服务，如果服务获取失败，则会在指定的时间内进行重试
     */
    @LoadBalanced
    @Bean
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }

    @Bean
    public IRule myRule(){
        return new RandomRule();//使用随机规则
    }
}

```

### 3.自定义规则

新建myRule包，在myRule包下自定义一个配置类MyRule.java，注意：**<font color=red>该包要跟启动类所在的包同级</font>**：

<a href="https://sm.ms/image/RcONQ8Lw7lgaobj" target="_blank"><img src="https://s2.loli.net/2022/04/25/RcONQ8Lw7lgaobj.png" style="zoom:50%;"  ></a>

MyRule.java

```java
@Configuration
public class MyRule {

    @Bean
    // 方法名任意
    public IRule hrnMyRule() {
        return new MyRandomRule();//默认是轮循，现在我们自定义
    }
}
```

主启动类开启负载均衡并指定自定义的MyRule配置类

```java
@SpringBootApplication
@EnableEurekaClient
//name表示消费者想要获取的服务
//在微服务启动的时候就能加载自定义的Ribbon类(自定义的规则会覆盖原有默认的规则)
//开启负载均衡,并指定自定义的规则
@RibbonClient(name = "SPRINGCLOUD-PROVIDER-DEPT",configuration = MyRule.class)
public class DeptConsumer_80 {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer_80.class,args);
    }
}
```

自定义的规则(这里我们参考Ribbon中默认的规则代码自己稍微改动)：MyRandomRule.java

```java
package nuc.ss.myrule;

import com.netflix.client.config.IClientConfig;
import com.netflix.loadbalancer.AbstractLoadBalancerRule;
import com.netflix.loadbalancer.ILoadBalancer;
import com.netflix.loadbalancer.Server;

import java.util.List;
import java.util.concurrent.ThreadLocalRandom;

public class MyRandomRule extends AbstractLoadBalancerRule {

    /**
     * 每个服务访问5次，则换下一个服务(总共3个服务)
     * total=0,默认=0,如果=5,指向下一个服务节点
     * index=0,默认=0,如果total=5,index+1
     */

    private int total = 0;          //被调用的次数
    private int currentIndex = 0;   //当前是谁在提供服务

    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            return null;
        }
        Server server = null;

        while (server == null) {
            if (Thread.interrupted()) {
                return null;
            }
            List<Server> upList = lb.getReachableServers(); //获得当前活着的服务
            List<Server> allList = lb.getAllServers(); //获取所有的服务

            int serverCount = allList.size();
            if (serverCount == 0) {
                return null;
            }

            //int index = chooseRandomInt(serverCount);//生成区间随机数
            //server = upList.get(index);//从或活着的服务中,随机获取一个

            //=====================自定义代码=========================

            if (total < 5) {
                server = upList.get(currentIndex);
                total++;
            } else {
                total = 0;
                currentIndex++;
                if (currentIndex >= upList.size()) {
                    currentIndex = 0;
                }
                //server = upList.get(currentIndex);//从活着的服务中,获取指定的服务来进行操作
            }

            //======================================================

            if (server == null) {
                Thread.yield();
                continue;
            }

            if (server.isAlive()) {
                return (server);
            }
            server = null;
            Thread.yield();
        }

        return server;

    }

    protected int chooseRandomInt(int serverCount) {
        return ThreadLocalRandom.current().nextInt(serverCount);
    }

	@Override
	public Server choose(Object key) {
		return choose(getLoadBalancer(), key);
	}

	@Override
	public void initWithNiwsConfig(IClientConfig clientConfig) {
		// TODO Auto-generated method stub
		
	}
}
```



## 四、Feign

**改造springcloud-api模块**

1.pom.xml添加feign依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>
```

2.新建service层，并新建DeptClientService.java接口

```java
@Service
//@FeignClient:微服务客户端注解,value:指定微服务的名字,这样就可以使Feign客户端直接找到对应的微服务
@FeignClient(value = "SPRINGCLOUD-PROVIDER-DEPT")
public interface DeptClientService {

    @GetMapping("/dept/get/{id}")
    Dept queryById(@PathVariable("id") Long id);

    @GetMapping("/dept/list")
    List<Dept> queryAll();

    @PostMapping("/dept/add")
    boolean addDept(Dept dept);
}
```

**创建springcloud-consumer-dept-feign模块**

拷贝springcloud-consumer-dept-80模块下的pom.xml，resource，以及java代码到springcloud-consumer-feign模块，并添加feign依赖。

```xml
<!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-feign -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>
```

需要使用到springcloud-api中的DeptClientService.java接口，引入它的依赖：

```xml
        <dependency>
            <groupId>nuc.ss</groupId>
            <artifactId>springcloud-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
```

通过Feign实现DeptConsumerController.java

```java
@RestController
public class DeptConsumerController {

    @Autowired
    private DeptClientService deptClientService = null;

    @RequestMapping("/consumer/dept/add")
    public boolean add(Dept dept) {
        return deptClientService.addDept(dept);
    }

    @RequestMapping("/consumer/dept/get/{id}")
    public Dept get(@PathVariable("id") Long id) {
       return deptClientService.queryById(id);
    }

    @RequestMapping("/consumer/dept/list")
    public List<Dept> list() {
        return deptClientService.queryAll();
    }
}

```

主配置类

```java
@SpringBootApplication
@EnableEurekaClient //开启Eureka 客户端
@EnableFeignClients(basePackages = {"nuc.ss.springcloud"})
public class FeignDeptConsumer_80 {
    public static void main(String[] args) {
        SpringApplication.run(FeignDeptConsumer_80.class,args);
    }
}
```

结果

![image-20201002135742053](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002135742053.png)



## 五、Hystrix

### 1.服务熔断

**新建springcloud-provider-dept-hystrix-8081模块**

拷贝springcloud-provider-dept–8081内的pom.xml、resource和Java代码进行初始化并调整。

导入hystrix依赖

```xml
<!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-hystrix -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>
```

调整yml配置文件

```yaml
# Eureka的配置
eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
  instance:
    instance-id: springcloud-provider-dept-hystrix-8081 #修改Eureka上的默认描述信息
    prefer-ip-address: true #改为true后默认显示的是ip地址而不再是localhost
```

prefer-ip-address: true：

![image-20201002180111151](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002180111151.png)

**修改controller**

```java
@RestController
public class DeptController {
    @Autowired
    private DeptService deptService;

    @Autowired
    private DiscoveryClient client;

    @GetMapping("/dept/get/{id}")
    @HystrixCommand(fallbackMethod = "hystrixGet")//如果出现异常,走后面的hystrixGet代码
    public Dept get(@PathVariable("id") Long id) {
        Dept dept = deptService.queryById(id);
        if (dept==null){
            throw new RuntimeException("这个id=>"+id+",不存在该用户，或信息无法找到");
        }
        return dept;
    }

    //根据id查询备选方案(熔断)
    public Dept hystrixGet(@PathVariable("id") Long id){

        return new Dept().setDeptno(id)
                .setDname("这个id=>"+id+",没有对应的信息,null---@Hystrix")
                .setDb_source("在MySQL中没有这个数据库");
    }
}
```

**为主启动类添加对熔断的支持注解@EnableCircuitBreaker**

```java
@SpringBootApplication
@EnableEurekaClient //在服务启动后自动注册到Eureka中
@EnableDiscoveryClient //服务发现
@EnableCircuitBreaker//添加对熔断的支持
public class DeptProviderHystrix_8081 {
    public static void main(String[] args) {
        SpringApplication.run(DeptProviderHystrix_8081.class,args);
    }
}
```

**测试**：

使用熔断后，当访问一个存在的id时，前台页展示数据如下

![image-20201002175040487](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002175040487.png)

使用熔断后，当访问一个不存在的id时，前台页展示数据如下

![image-20201002175118082](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002175118082.png)

不用熔断的springcloud-provider-dept–8081模块访问相同地址会出现下面状况

<img src="https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002175735407.png" alt="image-20201002175735407" style="zoom:50%;" />



### 2.服务降级

在springcloud-api模块下的service包中，新建降级配置类DeptClientServiceFallBackFactory.java

```java
@Component
public class DeptClientServiceFallBackFactory implements FallbackFactory {

    @Override
    public Object create(Throwable throwable) {
        return new DeptClientService() {
            @Override
            public Dept queryById(Long id) {
                return new Dept()
                        .setDeptno(id)
                        .setDname("id=>" + id + "没有对应的信息，客户端提供了降级的信息，这个服务现在已经被关闭")
                        .setDb_source("没有数据");
            }

            @Override
            public List<Dept> queryAll() {
                return null;
            }

            @Override
            public boolean addDept(Dept dept) {
                return false;
            }
        };
    }
}
```

在DeptClientService中指定刚才写好的降级配置类DeptClientServiceFallBackFactory

```java
@Service
//@FeignClient:微服务客户端注解,value:指定微服务的名字,这样就可以使Feign客户端直接找到对应的微服务
@FeignClient(value = "SPRINGCLOUD-PROVIDER-DEPT",fallbackFactory = DeptClientServiceFallBackFactory.class)   //
public interface DeptClientService {

    @GetMapping("/dept/get/{id}")
    Dept queryById(@PathVariable("id") Long id);

    @GetMapping("/dept/list")
    List<Dept> queryAll();

    @PostMapping("/dept/add")
    boolean addDept(Dept dept);
}
```

在springcloud-consumer-dept-feign模块中开启降级

```yaml
# 开启降级feign.hystrix
feign:
  hystrix:
    enabled: true
```

**测试**

正常访问

![image-20201002184343632](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002184343632.png)

关掉服务DeptProvider_8081继续访问

![image-20201002183509891](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002183509891.png)



### 3.Dashboard 流监控

**新建springcloud-consumer-hystrix-dashboard模块**

添加依赖

```xml
<dependencies>

    <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-hystrix -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-hystrix</artifactId>
        <version>1.4.7.RELEASE</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
        <version>1.4.7.RELEASE</version>
    </dependency>

    <!-- Ribbon -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-ribbon</artifactId>
        <version>1.4.7.RELEASE</version>
    </dependency>

    <!--eureka-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
        <version>1.4.7.RELEASE</version>
    </dependency>

    <dependency>
        <groupId>nuc.ss</groupId>
        <artifactId>springcloud-api</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

主启动类

```java
@SpringBootApplication
@EnableHystrixDashboard //开启
public class DeptConsumerDashboard_9001 {
    public static void main(String[] args) {
        SpringApplication.run(DeptConsumerDashboard_9001.class,args);
    }
}

```

启动应用程序，访问：localhost:9001/hystrix

<img src="https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002191522305.png" alt="image-20201002191522305" style="zoom:50%;" />

**服务端8081添加监控应用程序依赖**

```xml
<!--actuator完善监控信息-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

给springcloud-provider-dept-hystrix-8081模块下的主启动类添加如下代码,添加监控

```java
@SpringBootApplication
@EnableEurekaClient //在服务启动后自动注册到Eureka中
@EnableDiscoveryClient //服务发现
@EnableCircuitBreaker//添加对熔断的支持
public class DeptProviderHystrix_8081 {
    public static void main(String[] args) {
        SpringApplication.run(DeptProviderHystrix_8081.class,args);
    }

    //增加一个 Servlet
    @Bean
    public ServletRegistrationBean hystrixMetricsStreamServlet(){
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(new HystrixMetricsStreamServlet());
        //访问该页面就是监控页面
        registrationBean.addUrlMappings("/actuator/hystrix.stream");
        return registrationBean;
    }
}
```

**<font color=red>注意：先访问localhost:8081/dept/get/1，再访问localhost:8081/actuator/hystrix.stream，不然也会报错</font>**

在springcloud-consumer-hystrix-dashboard中的yml中添加配置（<font color=red>刚开始没加，一直报这个错: Unable to connect to Command Metric Stream</font>）

```yml
hystrix:
  dashboard:
    proxy-stream-allow-list: "*"
```

![image-20201002213325619](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002213325619.png)

运行结果：（注意心跳和圆的大小变化）

![image-20201002205116058](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002205116058.png)



## 六、Zuul

**新建springcloud-zuul模块**

导入依赖

```xml
<dependencies>
        <!--zuul-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zuul</artifactId>
            <version>1.4.7.RELEASE</version>
        </dependency>
        <!-- Hystrix依赖 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix</artifactId>
            <version>1.4.7.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
            <version>1.4.7.RELEASE</version>
        </dependency>

        <!-- Ribbon -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-ribbon</artifactId>
            <version>1.4.7.RELEASE</version>
        </dependency>

        <!--eureka-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
            <version>1.4.7.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>nuc.ss</groupId>
            <artifactId>springcloud-api</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
```

application.yml

```yaml
server:
  port: 9527
spring:
  application:
    name: springcloud-zuul
eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
  instance:
    instance-id: zuul9527.com
    prefer-ip-address: true

info:
  app.name: springcloud
  company.name: blog.kuangstudy.com
```

启动如下图三个服务（先去host文件里面添加www.kuangstudy.com的服务）

![image-20201002215932203](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002215932203.png)

访问http://localhost:8081/dept/get/1和http://www.kuangstudy.com:9527/springcloud-provider-dept/dept/get/1都可以获得数据

![image-20201002220043786](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002220043786.png)

![image-20201002220035277](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201002220035277.png)



在地址栏隐藏微服务springcloud-provider-dept的名称，application.yml中添加配置

```yaml
zuul:
  routes:
    mydept.serviceId: springcloud-provider-dept
    mydept.path: /mydept/**
```

访问这个地址即可http://www.kuangstudy.com:9527/mydept/dept/get/1

但是原路径http://www.kuangstudy.com:9527/springcloud-provider-dept/dept/get/1也能访问

![image-20201003104704709](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201003104704709.png)

继续配置application.yml之后,原来的http://www.kuangstudy.com:9527/springcloud-provider-dept/dept/get/1不能访问了

```yaml
zuul:
  routes:
    mydept.serviceId: springcloud-provider-dept
    mydept.path: /mydept/**
  ignored-services: "*"  # 不能再使用某个(*：全部)路径访问了，ignored ： 忽略,隐藏全部的
```

![image-20201003105303720](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201003105303720.png)

继续向application添加公共的访问前缀,访问路径变为http://www.kuangstudy.com:9527/kuang/mydept/dept/get/1

```yaml
zuul:
  routes:
    mydept.serviceId: springcloud-provider-dept
    mydept.path: /mydept/**
  ignored-services: "*"  # 不能再使用某个(*：全部)路径访问了，ignored ： 忽略,隐藏全部的~
  prefix: /kuang # 设置公共的前缀,实现隐藏原有路由
```

![image-20201003105852674](https://gitee.com/lzh_gitee/springboot_image/raw/master/img/image-20201003105852674.png)
