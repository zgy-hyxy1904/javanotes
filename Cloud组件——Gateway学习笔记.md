# Cloud组件——Gateway学习笔记

## 1.什么是网关？

一句话，客户端访问后台的统一入口

![img](https://img-blog.csdnimg.cn/2019082411015627.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hhbnRhbmdkdWhleQ==,size_16,color_FFFFFF,t_70) 

## 2.为什么我们需要网关？

**如果没有网关**，客户端与每个微服务直接通讯，会造成以下的不便

- 登录认证不便

如果微服务很多，每个微服务里都要有登录认证这一套逻辑，这就很烦了；如果有网关，把登录认证放到网关里实现，只需要做一次，很方便

- 客户端迭代不便

如果微服务的地址，域名改变了，客户端要做大量的改动；
用网关就不一样了，客户端只需要知道网关的地址就行了，请求都是经过网关转发，客户端不用关心微服务地址的改变

- 无法访问特殊协议的微服务

如果有些微服务使用了客户端不友好的协议，没有网关去转换协议，就无法访问这些微服务了

## 3.Spring Cloud Gateway

- 是spring cloud的第二代网关，未来会取代第一代网关zuul
- 基于Netty、Reactor以及WebFlux构建
- 性能强劲，是第一代网关zuul的1.6倍
- 功能强大，内置很多实用功能，例如转发，监控，限流
- 不支持springboot 1.x
- 还在改进当中，下面写的这版本没错，下版本就不一定对了

## 4.搭建Gateway网关

实现访问localhost：8080路由到consul服务或者百度浏览器上。

1.从Spring官网下载源码到本地

2.切换到阿里云的maven镜像仓库

3.下载依赖包（可能会下载失败，但是网络的问题，多下载几遍就好了）

4.找到GatewaySampleApplication类

![1577950017857](C:\Users\大仲马\AppData\Local\Temp\1577950017857.png)

5.改写代码

~~~Java
//				.route(r -> r.path("/**").uri("https://www.baidu.com"))
				.route(r -> r.path("/**").uri("http://192.168.182.128:8500"))
~~~

6./和*的作用

如果是单独的/，只是访问到当前的服务，

如果是单独的/**，可以访问到当前的服务后面的服务

## 5.spring cloud gateway的两大核心

### 5.1 gateway的架构图

由图可见，网关的两大核心，1.**转发请求**(路由，routes)，2.**过滤器**

![img](https://img-blog.csdnimg.cn/2019082416400457.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hhbnRhbmdkdWhleQ==,size_16,color_FFFFFF,t_70) 

### 5.2 核心之一 routes 路由

#### 5.2.1 路由nacos其它微服务

- **开启 gateway 发现服务注册组件上其它微服务的自动路由**
  **效果**：访问 ${gateway_url}/{微服务X}/** -------> 微服务X的/** 路径

~~~yaml
spring:
	cloud:
		gateway:
	      discovery:
	        locator:
	          # 开启 gateway 发现服务注册组件上其它微服务的自动路由
	          enabled: true
~~~

#### 5.2.2 自定义路由

##### 5.2.2.1 示例 (可以配置多个)：

如果有多个路由，请求会被第一个符合条件的路由转发走
将 http://localhost:8040/baidu/search/error.html?chid=wx 路由转发到 https://www.baidu.com/search/error.html，不带chid=wx参数的不转发

~~~yaml
spring:
	cloud:
		gateway:
	      routes:
	        - id: test_baidu_route
	          uri: https://www.baidu.com
	          predicates:
	            - Query=chid,wx
	            - Path=/baidu/**
	          filters:
	            - name: RewritePath
	              args:
	                regexp: /baidu/(?<remaining>.*)
	                replacement: /${remaining}
~~~

- id

这个路由的唯一标识，随便定义

- uri


转发的去处，就是要转发到哪里去

- predicates，


这是断言，也叫谓词工厂，这里使用了两个谓词，被转发的url必须满足这两个条件
第一个是Query，Query=chid,wx表示，请求gateway的链接必须要有参数 chid=wx才会被转发
第一个是Path，Path=/baidu/**表示，请求gateway的链接必须是 http://localhost:8040/baidu/****************，gateway_url后面必须是/baidu/的才会被转发

内置的谓词工厂有11个，起到不同的作用，如果这11个内置的谓词不能满足你的需求，你也可以写一个谓词工厂。这里是官方文档：https://cloud.spring.io/spring-cloud-gateway/reference/html/#gateway-request-predicates-factories

- 
  filters


过滤器，这里只配置了一个过滤器RewritePath，如果不配置这个路径替换过滤器
http://localhost:8040/baidu/search/error.html?chid=wx被转发到 https://www.baidu.com/baidu/search/error.html，你会发现中间多了一个 /baidu/，这就不对了，所以这个过滤器就是把 /baidu/替换掉的逻辑，使最终的url是 https://www.baidu.com/search/error.html，一个可以正常访问的url

内置的过滤器工厂有二三十个，各种功能都有，同样你也可以自定义符合自己需求的过滤器。这里是官方文档：https://cloud.spring.io/spring-cloud-gateway/reference/html/#_rewritepath_gatewayfilter_factory