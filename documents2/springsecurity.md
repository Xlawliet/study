# springsecurity

重量级

结合spring

本质上：是一个过滤器链



FilterSecurityInterceptor:是一个方法级过滤器，基本位于过滤链的最底部

ExceptionTranslationFilter:是个异常过滤器，用来处理在认证授权过程中抛出的异常

UsernamePasswordAuthenticationFilter:对/login的POST请求做拦截，校验表单中用户名，密码 （默认中使用框架中默认的user和密码，实际中对数据库中的数据做校验）

在Springboot中，Springboot对于SpringSecurity提供了自动化配置方案，可以使用更少的配置来使用SpringSecurity



### 过滤器如何进行加载？

使用SpringSecurity配置过滤器（DelegatingFileterProxy）

DelegatingFileterProxy中有一个doFilter方法

其中有一个初始化过滤器的操作

delegateToUse  = this.initDelegate(wca)

进入该初始化方法中可以看到

Filter delegate = (Filter)wca.getBean(targerBeanName......)

再次进入后可以看到

List<Filter> filter = this.getFilters....

这是在获取项目中的过滤器集合

并将其逐一加载到过滤链汇总





使用SpringSecurity时，有两个重要的接口

### userDetailsService

当什么也没有配置的时候，账号和密码都是由SpringSecurity定义生成的。而在实际开发中账号和密码都是从数据库中查询得到的。所以我们要通过自定义逻辑控制认证逻辑。

需要自定义逻辑，只需要实现UserDetailsService接口即可。

- 创建类UsernamePasswordAuthenticationFilter，并重写三个方法（attemptAuthentication，sucess方法和unsucess方法）
- 创建类实现UserDetailService，编写查询数据过程，返回User对象，这个User对象是我们安全框架中的框架，按照对应的格式返回即可。



### PasswordEncoder

在实际操作中，密码不会明文存储，所以这个接口就是对密码加密的接口。一般用于User对象中密码的加密。



## 认证

通过用户名和密码登陆的过程

方法一：

通过配置文件

```xml
spring:
  security:
    user:
      name: xzy123
      password: 123456
```

方法二：

通过配置类

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception{
        //用于对密码进行加密的对象
        BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
        String password = passwordEncoder.encode("123");
        auth.inMemoryAuthentication().withUser("xzy").password(password).roles("admin");
    }

    @Bean
    PasswordEncoder password(){
        return new BCryptPasswordEncoder();
    }
}
```



**方法三：**

自定义实现类设置：

第一步 创建配置类，设置使用哪个userDetailsService实现类

```java
@Configuration
public class SecurityConfig1 extends WebSecurityConfigurerAdapter {
    @Autowired
    private UserDetailsService userDetailsService;
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception{
        //用于对密码进行加密的对象
        auth.userDetailsService(userDetailsService).passwordEncoder(password());
    }
    @Bean
    PasswordEncoder password(){
        return new BCryptPasswordEncoder();
    }
}
```

第二步 编写实现类，返回User对象，User对象有用户名密码和操作权限

```java
@Service("userDetailsService")
public class MyUserDetailsService implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        //查询到数据库账号密码和权限集合
        List<GrantedAuthority> auths =
                AuthorityUtils.commaSeparatedStringToAuthorityList("role");
        return new User("mary",new BCryptPasswordEncoder().encode("123"),auths);
    }
}
```



可以使用前两种方法来设置超级管理员简化操作



实例：

第一 引入相关依赖 以及配置数据库信息

第二 创建数据库和表

第三 创建user表对应的实体类entity

第四 创建接口

第五 编写dao以及mapper

第六 给启动类添加注解



## 授权