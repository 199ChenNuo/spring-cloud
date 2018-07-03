# spring cloud

## 1. 小组成员  

| 学号 | 姓名 | github | 邮箱 |  
|:-:|:-:|:-:|:-:| 
| 516030910257 | 胡雨奇 | ReimuYK |  yuki.h@sjtu.edu.cn |  
| 516030910199 | 陈诺 | 199ChenNuo | cn.222@sjtu.edu.cn |
| 515015910005 | 丁丁 | derFischer | dingd2015@sjtu.edu.cn |
| 516030910101 | 罗雨辰 | 592McAcoy | 592mcavoy@sjtu.edu.cn |  

----
## 2. 搭建  

### 1. 注册与发现  

#### 1.1 工程组成  
- server:  *eureka-server*
- client:  *service-hi*    
#### 1.2 创建过程  
### 1.2.1 server   

在主工程下新建Maven module  
本小组采用的是Intellij IDEA，在创建过程中选择spring initializer项目，在dependencies中勾选cloud discovery下的eureka server   
创建完成后的项目的pom.xml中除了Maven项目常见依赖外，还应包含以下依赖：
  
    <dependency>  
        <groupId>org.springframework.cloud</groupId>  
        <artifactId>spring-cloud-starter-eureka-server</artifactId>  
    </dependency>
  
   
为了使得本module成为eureka server，在主类前要加上注解@EnableEurekaServer  
   
在配置文件中指定本module的运行端口并表明自己是eureka server（见本项目中的    erueka-server.src.main.ersources.application.yml)  

----

### 1.2.2 client  

在主工程中新建Maven module  
创建过程同server  
  
在配置文件中标明自己是client：  
在主类前加上标注@EnableEurekaClient（代替前文中的@EnableEurekaServer）  
在配置文件中注明运行端口以及自己要注册服务的地址（见本项目service-hi.src.main.resources.bootstrap.yml)  

#### 1.3 访问：
- server：  
http://localhost:8761    
- 服务：   
http://localhost:8762/hi?name=bob  

----
----

### 2. 负载均衡  
#### 2.1 工程组成  
- ribbon: *service-ribbon*  
- Feign: *service-feign*  
#### 2.2 创建过程  
### 2.2.1 ribbon
在工程下新建一个spring boot的module，在pom.xml除了spring boot的依赖，前文提到过的spring-cloud-starter-eureka以外，还需要加入ribbon的依赖spring-cloud-starter-ribbon  

    <dependency>  
        <groupId>org.springframework.cloud</groupId>  
        <artifactId>spring-cloud-starter-ribbon</artifactId>  
    </dependency>
  
  
同时在配置文件（见service-ribbon.src.main.resources.application.yml)中指明module运行端口（8764）以及服务注册地址（http://localhost:8761/eureka)  
  
在主类前加上注解@EnableDiscoveryClient，注入一个bean：restTemplate（加上注解@bean、@LoadBalanced）其中@LoadBalanced表示restTemplate开启附在均衡功能  

---
### 2.2.2 Feign
同上，在工程下新建一个spring bootmodule，与ribbon的pom.xml不同之处在于把spring-cloud-starter-ribbon替换为spring-cloud-starter-feign  
<code>
\<dependency>  
    \<groupId>org.springframework.cloud\</groupId>  
    \<artifactId>spring-cloud-starter-feign\</artifactId>  
\</dependency>  
</code>
  
在配置文件（service-feign.src.main.resources.application.yml）中指定module运行端口与注册服务地址  
  
在主类前加上@EnableFeignClient注解
#### 2.3 访问  
- server：  
http://localhost:8761    
- ribbon  
http://localhost:8764/hi?name=bob  
- Feign  
http://localhost:8765/hi?name=bob  
（开启多个hi服务后，若多次访问能够发现此时已满足负载均衡）  

---
---
  
### 3. 路由网关  
#### 3.1 工程组成  
- zuul: *service-zuul*
#### 3.2 构建过程
在工程中新建一个spring boot的module，在pom.xml中注明的依赖除了eureka以及spring boot项目的依赖外，还加入了spring-cloud-starter-zuul  
    
    <dependency>  
        <groupId>org.springframework.cloud</groupId>  
        <artifactId>spring-cloud-starter-zuul</artifactId>  
    </dependency>  
    
   
在主类前要额外加上注解：  
- @EnableEurekaClient   
- @EnableZuulProxy  
       
在配置文件中指定服务注册中心的地址，注明本模块运行的端口与服务名称，配置文件中表明以/api-a/开头的请求都转发给ribbon,/api-b/的请求都转发给frign服务  
  
主类中的run()方法起到了过滤请求的作用，所有没有带有token参数的请求都会被过滤掉  
#### 3.3 访问  
- server：  
http://localhost:8761    
- 成功请求  
http://localhost:8769/api-a/hi?name=bob&token=anyToken   
http://localhost:8769/api-b/hi?name=bob&token=anyToken  
- 失败请求  
http://localhost:8769/api-a/hi?name=bob   
http://localhost:8769/api-b/hi?name=bob    

----  
---- 

### 4. 服务链追踪  
#### 4.1 工程组成  
- zuul: *server-zipkin*
- hi: *zipkinService-hi*
- miya: *zipkinService-miya*
#### 4.2 创建过程  
在项目中新建spring boot的module，在pom.xml除了spring boot项目的依赖外，标注引用zipkin-server, zipkin-autoconfigure-ui  

        <dependency>
            <groupId>io.zipkin.java</groupId>
            <artifactId>zipkin-server</artifactId>
        </dependency>

        <dependency>
            <groupId>io.zipkin.java</groupId>
            <artifactId>zipkin-autoconfigure-ui</artifactId>
        </dependency>
  
在主类前加上注解@EnableZipkinServer  

在配置文件中指定服务运行的端口  

----

zipkinService-hi & zipkinService-miya  

新建module，在pom.xml引入依赖spring-cloud-starter-zipkin  
  
在配置文件中指定zipkin的base-url，本身的运行端口和本身的服务名称  

各自有调用对方的方法（才有职责链让zipkin去追踪）
#### 4.3 访问  
- server  
http://localhost:9411  
-service
http://localhost:8988/hi  
http://localhost:8989/miya  

---
---

### 5. 其他  
启动顺序：   
eureka-server  
-> service-hi( should run mutiple service)   
-> service-ribbon / service-feign   
-> service-zuul   

server-zipkin   
-> zipkinService-hi / zipkinService-miya  

---
---

### 6. 参考  
史上最简单的spring cloud教程 | 终篇（https://blog.csdn.net/forezp/article/details/70148833 ）  
