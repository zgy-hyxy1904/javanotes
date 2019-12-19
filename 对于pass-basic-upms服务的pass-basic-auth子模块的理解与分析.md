# 对于pass-basic-upms服务的pass-basic-auth子模块的理解与分析(大写为类，小写为包)

## 一. pass-basic-auth-api（提供api方法的模块）

暂时没有完全理解这个子模块的作用，这里面只有一个paas-basic-auth-api.iml文件，里面全都是 maven对应的lib包，我猜测可能是将maven库中的各种jar包，变成项目中的lib包，为本服务中的其他模块提供api方法。

## 二.pass-basic-auth-provider（提供auth主要的接口）

这个模块是auth的核心所在，可以说是一个非常基础的模块，对外定义了封装好的返回值类型和一些常量，生成token的接口，还有一些基础的controller层的一些基础接口，主要是为了其他服务提供支持。

### (一)common(基础的工具类)

#### 1.response

##### (1)BaseResponse

返回的基础类，其中有两个属性，一个是web返回的网页代码，默认是200，一个是传递消息的一个属性，其他为get，set方法和构造器。

##### (2)ObjectRespponse

对象返回类，两个属性分别代表成功与否和返回的数据，有六种构造器，方便给对象传值，然后是get，set方法，还有一个isSuccess是否成功的方法。

#### 2.AuthConstants

定义了许多授权的常量，结合英文拼写看个大概意思，并不是很准确。

#### 3.BaseContextHandle

基础的上下文信息类，首先定义常量ThreadLocal对象，保证除了互斥锁之外的线程方法，规避多线程访问出现线程不安全 。下面都是保证多个线程修改同一个属性时候，保证数据的一致性，安全性的API操作方法。例如，getUserId，getUserName，setToken，getToken，setIsRefreshToken，isRefreshToken。

### (二)config（一些接口的配置）

#### 1.KeyConfiguration

公私钥存储对象类，里面有公钥和私钥两个属性，以及一些get私钥/公钥或者set公钥/私钥方法,公钥私钥应该和token有关。

~~~java
@Value("${jwt.rsa-secret}")
private String rsaSecret;//猜测通过注解的方式将值注入到rsaSecret变量中。
public String getRsaSecret() {
		return rsaSecret;
}
	/**
	 * @param rsaSecret the rsaSecret to set
	 */
public void setRsaSecret(String rsaSecret) {
    this.rsaSecret = rsaSecret;
}
	/**
	 * @return the userPubKey
	 */
~~~

#### 2.LoginProperties

登录的配置类

~~~java
@Data//为了这个类提供读写属性
@Configuration//表明他是有一个配置类
@ConfigurationProperties(prefix = "login")//获取配置文件的属性值
public class LoginProperties {
	private Boolean sso;	
	private int expire;
}
~~~

#### 3.UserAuthConfig

用户授权配置类，@Configuration注解，表示这是一个配置类，只有一个headerToken属性，使用value注解将配置文件中的值，注入到属性中，然后是get和set方法。

#### 4.WebConfig

web网络配置类，@Configuration注解，表示这是一个配置类，他是采用配置类的方式代替了传统的XML的配置方式，实现了WebMvcConfigurer接口，重写了addInterceptors方法，这个方法是配置了一个拦截器，拦截器的作用是在静态资源访问后台的代码时，起到一个过滤的作用，是一种切面的编程思想。然后用@Bean注解自定义配置了一个用户授权的拦截器。后面在interceptors包中进行了实现。

### (三)controller（客户端和JWT的控制器）

#### 1.ClientController

~~~java
@Slf4j //这是一个日志输出到控制台的注解，方便程序员开发
@RestController //这是将ClientController类对象注入到Spring容器中
@RequestMapping("client") //这是一个URL定位注解，如果http://localhost:8080/umps/bocloud/client就可以访问到
public class ClientController {
    private final KeyConfiguration keyConfiguration;//定义公私钥存储对象为常量。

    @Autowired//这个注解能够将keyConfiguration的属性注入进来，而且避免了空指针异常。
    public ClientController(KeyConfiguration keyConfiguration) {
        this.keyConfiguration = keyConfiguration;
    }
    //这是一个构造器方便ClientController传递keyConfiguration对象进行初始化赋值。

    @PostMapping("/userPubKey")
    //这是一个处理请求地址映射的注解，相当于@RestController（method：post）
    public ObjectResponse<byte[]> getUserPublicKey(@RequestParam("clientId") String clientId, @RequestParam("secret") String secret){
        return new ObjectResponse<byte[]>().data(keyConfiguration.getUserPubKey());
    }
}
//@RequestParam将报文体中的clientId和secret属性的值，注入到参数列表中。
//该方法是一个获取用户公钥的方法
~~~



#### 2.JwtController

这是一个JWT的控制器，这个类能生成token，刷新token，注销token，校验token，里面的注解，和上一个控制器的注解一样，就不多解释了，业务逻辑不太复杂，无非是正确了怎么样，错误了怎么样。

### (四)exception（基础的异常处理）

#### 1.base

##### (1)BaseException

异常信息基础类，继承了运行时异常类，实现了BaseExceptionCode接口，重写实现了获取code和message方法，这是一个自定义的基础异常，用来处理比较常见的异常信息。

##### (2)BaseExceptionCode

.这是一个异常接口，在里面定义了两种异常，一种是获取code出现的异常，一种是获取message出现的异常。

#### 2.AuthAssert

授权维护类，继承BizAssert（暂时没有理解这单词），定义了两个一样的方法fail，但是有两种不同的参数列表，

参数列表是int code, String message和AuthExceptionCode exceptionCode。

方法的作用是，如果失败则抛出AuthException异常。

#### 3.AuthException

这是一个授权异常类，继承了BaseException基础异常类，这里面定义了serialVersionUID常量为1L,还有多种参数不同的构造器。方便对AuthException进行初始化赋值。

#### 4.AuthExceptionCode

这是一个枚举类，在这里面定义了多种Code值，例如

~~~java
TOKEN_NOT_NULL(1020005, "TOKEN不能为空"),
TOKEN_VALIDATE_FAILED(1020005, "TOKEN校验失败！"),
TOKEN_INVALID_FAILED(1020004, "TOKEN注销失败！"),
TOKEN_REFRESH_FAILED(1020003, "TOKEN刷新失败！"),
TOKEN_CREATE_FAILED(1020002, "TOKEN生成失败！"),
TOKEN_IS_DISABLED(1020001, "TOKEN失效");
~~~

然后定义了code和message属性，以及获取获取code和message的get，set方法，以及构造器。重写了基础类中的getMsg方法。

### (五)interceptor（拦截器）

#### 1.UserAuthInterceptor

这里对应WebConfig类中的拦截器，这里将自定义的拦截器进行实现。

此类继承了HandlerInterceptorAdapter（这是SpringMVC中的重要组件），重写了preHandle和afterCompletion方法，加入了我们对于获取token的拦截，（具体的细节没太看懂，涉及到部分源代码）

### (六)runner（初始化执行）

#### 1.AuthServerRunner

初始化执行类，实现了CommandLineRunner接口，重写了run方法，这样做的目的是想要在项目启动后执行一些功能，将功能的代码放在了从写的run方法中。

这里将公钥和私钥的信息通过RedisTemplate属性，将值通过set的方式储存到redis数据库中，

这里将公钥和私钥的信息通过KeyConfiguration属性，将值通过set的方式储存到对象中，

以上都是在想在项目启动后进行的初始化操作。

### (七)service（授权的业务接口与实现）

#### 1.impl

##### (1)AuthServiceImpl

这是服务实现的接口实现类，分别实现了获取token，刷新token，注销token，校验token方法，有比较复杂了业务逻辑，但是通过注释还算看得明白。

StrUtil：这是一个字符串过滤类

validate：判断token是否为空	

有待细看看

#### 2.AuthService

这是一个服务实现的接口，里面定义了获取token，刷新token，注销token，校验token的抽象方法。是为了AuthServiceImpl做的设计模式。

### (八)util（对JWT获取token的接口的实现）

#### 1.jwt

##### (1)IJWTInfo（接口）

定义好的接口，里面有获取用户Id，获取用户账号唯一，获取用户名，获取过期时间的抽象方法

##### (2)JwtAuthenticationRequest

用户登录信息类，定义了serialVersionUID常量，以及userID，username，token属性，无参的构造方法，有参数的构造方法。

##### (3)JWTHelper

是使用JWT的帮助文档，有的方法，而且方法都都两种，参数列表的不同来区分

1. 密钥加密token
2. 公钥加密token
3. 获取token中的用户信息

##### (4)IJWTInfo（实现类）

JWT信息类，实现了Serializable, IJWTInfo接口

Serializable：实现这个接口，表明这个类需要实现序列化。

IJWTInfo：重写了IJWTInfo接口，返回要求的userId，username，name，expiration属性

##### (5)RsaKeyHelper

公钥私钥算法帮助文档，这里面有获取公钥，获取密钥，获取公钥，获取密钥，生存rsa公钥和密钥，生存rsa公钥，生存rsa私钥的相关方法，为其他的类做服务，如果需要，就调用对应的方法。

#### 2.BizAseert

此类比较诡异，代码不是很复杂，大多是一样的代码，注释也比较详细，知道各个属性的意义，但是还是看不太明白。

#### 3.Common

定义了所有我们项目需要的常量信息，方便进行修改和调用保证了一致性。

#### 4.DateSeralizer

数据序列化，实现了JsonSerializer<Date>接口，重写serialize方法，总体的来说就是对时间的格式进行了重写，采用"yyyy-MM-dd HH:mm:ss"在种时间格式。

#### 5.JwtTokenUtil

此类是JWT的token工具类，自动注入LoginProperties，KeyConfiguration两个接口

LoginProperties：登录配置接口

KeyConfiguration：秘钥配置接口

generateToken（）：生成Token的方法，调用JWTHelper.generateToken,参数为上面两个接口的属性值

getInfoFromToken（）：从token获取详细信息

