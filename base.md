# 基础组件

## 分享

- 首先要感谢 [Pivotal](https://pivotal.io/) 这家公司，他们的产品确实让互联网变得简单了并且几乎要一统天下了。如果你不知道我在说什么，那么你只需要记住 Spring 就是他们做的就行了。

- [Spring Cloud](https://spring.io/projects/spring-cloud)  为开发者提供了各种分布式系统组件，你可以按照你的需求和兴趣进行选择。主体框架我选择 [Spring Cloud Netflix](https://spring.io/projects/spring-cloud-netflix) ，就是常说的（Eureka , Hystrix , Zuul ...），Eureka设计上遵循**AP**原则，保证了高可用型，优势我就不一一列举了。选轮子是停幸福的，我选择轮子的习惯是：在不是那么清楚的情况下，选择大家都在用的。

  ps : 如果不清楚**AP**原则的话，可以看看阮一峰的《[CAP 定理的含义](http://www.ruanyifeng.com/blog/2018/07/cap.html)》。

- Netflix我就不展开说了，我主要想分享的是平时可以用到的业务组件。为了保证你可以正常使用和测试，[基础包](https://github.com/westwong/springcloud-base)中我只配置 [Eureka-Server](https://github.com/westwong/springcloud-base/tree/master/eureka-server)  和 基于Zuul 的 [Api-gateway](https://github.com/westwong/springcloud-base/tree/master/api-gateway) , 熟悉的人可以直接跳过。 
- [Github](https://github.com/westwong/springcloud-base)

##  Eureka-Server

- Eureka Server 很简单，但是特别重要，因为他是微服务的核心，提供注册和发现能力，就是所谓的 **注册中心** ，每一个微服务都会安装 （引入）一个 Eureka Client 向 Eureka Server 注册自己。当然 Eureka Server 本身也可以相互注册形成**注册中心**集群。

- 其实我觉得没什么好说的，因为真的特别简单，虽然我说了面向的人群是有一点 Spring Cloud 基础的人，但是总会担心万一你正好进来看到了，还是不知道怎么做。所以我还是多说一点吧。如果你有兴趣的话可以 查阅官方文档和相关博客。

  step 1: 下载包

  ```bash
  git clone https://github.com/westwong/springcloud-base.git
  ```

  下载下来之后，如果你想直接使用 

  ```bash
  cd eureka-server  # 到达 eureka server 目录
  mvn clean package -Dmaven.test.skip=true # 直接打成jar包 run 起来
  ```

  当然也可以 step 2: idea导入项目

  然后你会发现我为什么要感谢 [Pivotal](https://pivotal.io/) 代码量少到发指。

  ```java
  @SpringBootApplication
  @EnableEurekaServer // 启动  eureka Server
  public class EurekaServerApplication {
  
      public static void main(String[] args) {
          SpringApplication.run(EurekaServerApplication.class, args);
      }
  
  }
  ```

  对！只有这个一个启动类，没了！！！

  application.yml 文件也特别简单：

  ```yaml
  server:
    port: 12345
  spring:
    application:
      name: eureka-server
  eureka:
    instance:
      hostname: localhost  #服务注册中心实例的主机名
    server:
      enableSelfPreservation: false   #关闭自我保护
      eviction-interval-timer-in-ms: 10000   # 续期时间，即扫描失效服务的间隔时间（缺省为60*1000ms）
    client:
      register-with-eureka: false #是否向服务注册中心注册自己
      fetch-registry: false #是否检索服务
      serviceUrl.defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/ #服务注册中心的配置内容，指定服务注册中心的位置
  
  ```

  pom.xml 依赖也很简单：

  ```xml
  		<dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
          </dependency>
  
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-test</artifactId>
              <scope>test</scope>
          </dependency>
  
          <!--for jdk 11 or later-->
          <!-- https://mvnrepository.com/artifact/javax.xml.bind/jaxb-api -->
          <dependency>
              <groupId>javax.xml.bind</groupId>
              <artifactId>jaxb-api</artifactId>
              <version>2.3.0</version>
          </dependency>
          <dependency>
              <groupId>com.sun.xml.bind</groupId>
              <artifactId>jaxb-impl</artifactId>
              <version>2.3.0</version>
          </dependency>
          <dependency>
              <groupId>com.sun.xml.bind</groupId>
              <artifactId>jaxb-core</artifactId>
              <version>2.3.0</version>
          </dependency>
          <dependency>
              <groupId>javax.activation</groupId>
              <artifactId>activation</artifactId>
              <version>1.1.1</version>
          </dependency>
  
  ```

  下面几个 关于`JAXB-api` 的包是因为：`Java11` 移除了 `JavaEE` 模块,所以很多诸如 javax JAXB 等已经被移除，
  所以需要单独引入。

  plugin部分是主要是为了支持Docker，建议查看相关文档，该插件也支持`dockerfile`。

## Api-gateway

- 基于Zuul的网关服务。网关主要向外暴露聚合 API ，屏蔽内部微服务变动带来的影响，保证服务的稳定性。他的功能只会比你想象的更强大，比如：动态路由、监控、弹性负载和安全功能等。

- 一个简单例子：一个场景需要查询用户信息，查询用户信息的 api 为 “/user/get”。 当有一天你的 “用户信息服务器” 性能不足以支撑这项业务，你需要新增服务器。传统的做法是再添加一个 Nginx 。而现在你只需要多启动一个是网关服务就可以解决这个问题。默认配置下，你只需要访问网关所在服务器的地址： “/`微服务名称`/user/get”。

- 你可以按照 Eureka Server 相关步骤来完成，你同样会发现系统的伟大。

- 当然他还有很多骚操作等着你去get 。提出我的一点遗憾吧，感觉他对 web socket 的支持不太好。

## 尾声

- 我们的基本架子就搭建好了，不过虽然 Spring 这么厉害，他也只是帮助你更快的完成某些通用的需求罢了，真正的应用还需要你的 **想象力** 和 **创造力** 。
- 我想做的就是把我写过一些微服务组件上传到 [githup](https://github.com/westwong/) 并在这里做一个集中说明，希望可以帮到你吧。
- 我们开始吧！

<img src="C:\Users\west\AppData\Roaming\Typora\typora-user-images\image-20200116164211754.png" alt="image-20200116164211754" style="zoom:50%;" />

  

  

  