# Spring注解

# 注解

LDAP：轻量级目录访问协议



### 常见注解：

```java
@Configuration
定义配置类,将其注解的类作为bean注入容器
相当于在XML中配置beans，相当于Ioc容器，用该注解的类必须使用<context:component-scanbase-package=”XXX”/> 扫描。
  
@Bean
 	告诉Spring该方法将会返回一个对象，会交给Ioc来负责对象的创建。
  
@Component
  注解表明一个类会作为组件类，并告知Spring要为这个类chu a

@Value{}
赋值，相当于在properties中的value。
  
@Controller, @Service, @Repository,@Component
四种注解作用相同，都是控制器用于处理某个请求。
  
@RestController
  相当于@Controller和@ResponseBody的结合，返回json数据不需要在方法前加@ResponseBody注解了，但是不能返回jsp.html页面，视图解析器无法解析jsp页面。
  
@Autowired
用于自动装配，通过byType的形式进行注入。
  
@Qualifier
  
  
@Resource
  
@RequestBody
  主要是用于接收前端传递给后端的数据的。使用该注解接受数据时，前端不能使用GET方式提交数据，而是用POST方法。@RequestBody只能有一个，而@RequestParam可以有多个。
  
@RequestParam(value="参数名",required="true/false",defaultValue="")
  将请求参数绑定到指定的控制器的方法参数上
  获取到url中对应参数名的值绑定到所给参数上。
  value：参数名。
  required：是否必须包含该参数，默认为true，表示该请求路径中必须包含该参数，如果不包含就报错。
  defaultValue：默认参数值，如果设置了该值，required=true将失效，自动为false，如果没有传该参数，就使用默认值。
  
@Data
  等价于Setter、Getter、ToString、RequiredArgsConstructor...
  
@GetMapping(path)
  已经默认封装了@RequestMapping(method=RequestMethod.GET)
  
@PostMapping(path)
  和Get相似
  
 @ApiParam
  为Rest接口参数添加其他院数据
  
@ApiModelProperty(value="",description="")
  表示对model属性对说明或者数据操作更改
  value：属性简要说明
  name：重写属性名称
  dataType：参数的数据类型，可以是原始数据类型，此值将覆盖从类属性读取的数据类型
  required：是否为必传参数
  example：属性的示例值
  hidden：隐藏模型属性 true隐藏 fakse不
  
@PathVariable（"URL中{}里的参数名"）
  可以将URL中占位符参数绑定到控制器处理方法的入参中
  主要就是获取url{}中的参数的值
  https://blog.csdn.net/weixin_45393094/article/details/108814901?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param
  
@Transactional
	是声明式事务管理编程中使用的注解。
  添加位置：在接口实现类或接口实现方法上，而不是在接口类中。
  访问权限：public的方法才起作用。
  value：非必选，指定使用的事务管理器。
  propagation：可选，事务传播行为设置。
  rollbackFor：导致事务回滚的异常类数组。
  isolation：可选的事务隔离级别设置。
  timeout：事务超时时间设置。  
  

@LoginRequired
  配置拦截的路由，即判断要访问的方法是否需要登陆，如果不满足则会被拦截无法访问该方法/类。
  
@Contract(threading = ThreadingBehavior.SAFE)
  表明线程是安全的，多线程时也可以放心使用。
 
@Slf4j
  简化代码，省略：
    private finnal Logger logger = LoggerFactory.getLogger
  简化为可以直接使用log.info()

@ApiOperation(value"接口说明",httpMethod="接口请求方式",response="接口返回参数类型",notes="接口发布说明"...)
  不是Spring自带的，是swagger里用来构建Api文档的。
  
@Aspect
  把当前类表示为一个切面供容器读取
    
@JsonFormat
    注解的作用是格式化时间类型数据传输时的格式，以自己想要的格式来展示日期，同时也设置时区，避免时间展示与想要的结果产生误差。
@DateTimeFormat
    注解作用则是将前端传来的字符串类型的日期转为后台需要的时间类型结果，不加此注解，请求会报错400，请求参数错误，对于此类错误要注意int类型数据传输也是一样。
    
    
```



### lombok注解：

```java

@Getter/Setter
  自动生成get/set方法，默认为public，可手动设置为public、protected、private、package；
  使用属性AccessLevel.（NONE、PUBLIC...）可以覆盖类上的@Getter、@Setter和@Data的访问级别
  
@EqualsAndHashCode
  生成equals和hashcode方法
 
@Builder
  将生成类的一个当前流程的一种链式构造工厂，可以配合@Singular使用，如果未使用@Singular，那么在一般的setter时，会直接覆盖原来的引用，标注以后，集合属性支持添加操作，会在属性原来的基础上增加。
```

