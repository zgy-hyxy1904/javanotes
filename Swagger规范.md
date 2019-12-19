# Swagger规范

1. 所有关于参数描述的文字必须以变量的形式来引用，统一到一个类中。

2. Bean对象上添加注解

   （1）Class上添加注解 @ApiModel

   （2）u 属性上添加注解 @ApiModelProperty(value = "测试姓名",example = "测试张三") 属性如果属于对象关联属性则不需要添加example。

示例：

~~~Java
@ApiModel

public class TestVo {
    @ApiModelProperty(value = "id",example = "1")
    private String id;
    @ApiModelProperty(value = "测试姓名",example = "测试张三")
    private String name;
    @ApiModelProperty(value = "测试密码",example = "password")
    private String passWord;
    @ApiModelProperty(value = "嵌套")
    private TestVo2 testVo2;
}
~~~

3. Controller上添加注解

   （1）Class上添加注解 @Api(description = "swagger test") description 描述这个controller是用来做什么的 @ApiIgnore：在class上是过滤掉这个constroller不让这个类下面的接口在前端显示，在方法上不让这个方法在前端显示

4. 方法上添加注解

   （1） @ApiOperation(value = "接口名",notes = "test",produces = "application/json")；说明：value：方法名；notes：接口详细描述；producces：响应格式

   （2）@ApiParam(value = "订单编号",required = true)；说明：value：参数描述；required ：是否必填

示例：

~~~java
@RequestMapping
@RestController
@Api(description = "swagger test")
//@ApiIgnore
public class TestSwaggerController {
    @ApiOperation(value = "获取新的订单信息",notes = "test",produces = "application/json")
//    @RequestMapping(value = "/getOrder",method = RequestMethod.GET)
    @GetMapping(value = "test/swagger/getOrder")
    public TestVo getOrder(@ApiParam(value = "订单编号",required = true) @RequestParam(value = "orderNo", required=false) String orderNo,
                           @ApiParam(value = "当前页") @RequestParam(value = "pageNum",required = false) Integer pageNum,
                           @ApiParam(value = "每页显示数量") @RequestParam(value = "pageSize",required = false) Integer pageSize){
        return new TestVo();
    }

~~~

# swagger环境配置

(1) Pom集成

~~~java
<!-- swagger-ui -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>${swagger2.version}</version>
</dependency>
<!-- swagger2 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>${swagger2.version}</version>
</dependency>
<!-- 解决FluentIterable.class找不到问题 -->
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>26.0-jre</version>
</dependency>

~~~

(2) 配置文件

**swagger:**  **enable:** true 是否启用ui true启用 false不启用

(3) 代码集成

 ~~~java
@Configuration
@EnableSwagger2
@ConditionalOnExpression("${swagger.enable}")
public class SwaggerConf extends WebMvcConfigurationSupport {
//根据url配置@Bean
public Docket alarm_api_bm() {
    return new Docket(DocumentationType.SWAGGER_2)
            .apiInfo(apiInfo("alarm", "alarm", "1.0"))
            .select()
            .apis(RequestHandlerSelectors.any())
            .paths(PathSelectors.ant("/v2.0/alertHistory/**"))
            .build()
            .groupName("alarm-接口文档V1.0")
            .pathMapping("/");
}
//按照包名来配置
    @Bean
    public Docket createAccepterRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
            .groupName("业务数据模块API")//分组名,不指定默认为default
            .select()
            .apis(RequestHandlerSelectors.withClassAnnotation(Api.class))
            .apis(RequestHandlerSelectors.basePackage("com.paas.bocloud.registry"))
            // 扫描的包路径
            .paths(PathSelectors.any())// 定义要生成文档的Api的url路径规则
            .build()
            .apiInfo(apiInfo())// 设置swagger-ui.html页面上的一些元素信息
            .enable(true);
/*** ** api信息 * * @param name        标题 * @param** description 描述 * @param** version     版本 * @return* **/private ApiInfo apiInfo(String name, String description, String version) {
    return new ApiInfoBuilder().title(name).description(description).version(version).build();
}
    //addResourceHandlers方法添加了两个资源处理程序，
    //这段代码的主要作用是对Swagger UI的支持。(访问接口页面为空白时可加上)
    @Override
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");
        registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/");
    }
 ~~~

(4) 测试

~~~java
	@ApiOperation(value = "获取新的订单信息")
//	@RequestMapping(value = "/getOrder",method = RequestMethod.GET)
  	@GetMapping(value = "test/swagger/getOrder")
    public String getOrder(@ApiParam(value = "订单编号",required = true) @RequestParam(value = "orderNo", required=false) String orderNo,
                           @ApiParam(value = "当前页") @RequestParam(value = "pageNum",required = false) Integer pageNum,
                           @ApiParam(value = "每页显示数量") @RequestParam(value = "pageSize",required = false) Integer pageSize){
        return "请求测试成功";
    }
~~~

结果

![img](file:///C:\Users\大仲马\AppData\Local\Temp\ksohtml6792\wps1.jpg) 

![1576736820202](C:\Users\大仲马\AppData\Local\Temp\1576736820202.png)