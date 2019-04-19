1. SpringMVC核心处理流程：
- 1、DispatcherServlet前端控制器接收发过来的请求，交给HandlerMapping处理器映射器
- 2、HandlerMapping处理器映射器，根据请求路径找到相应的HandlerAdapter处理器适配器（处理器适配器就是那些拦截器或Controller）
- 3、HandlerAdapter处理器适配器，处理一些功能请求，返回一个ModelAndView对象（包括模型数据、逻辑视图名）
- 4、ViewResolver视图解析器，先根据ModelAndView中设置的View解析具体视图
- 5、然后再将Model模型中的数据渲染到View上

2. springMVC支持多种方式将客户端的数据传输到处理器方法中：
- a.查询参数(Query Paramter) -> @RequestParam(value="...",defualtValue="")
- b.表单参数(Form Paramter) -> @RequestMapping(value="...") 返回一个表单页面供客户端填写
- c.路径变量(Path Variable) -> @PathVariable(value="...")

3. Java在提供了多种校验注解来校验客户端提交表单内容是否合法，使逻辑处理器更专注于核心业务处理 通过@Valid注解开启对象校验
- @NotBlank    检验字符串参数不能为空
- @NotNull    校验参数不能为null
- @Null    校验参数为null
- @NotEmpty    字符串不能为空，集合不能为空
- @Size(min = 1,max = 20)    检验集合元素的个数是否满足要求
- @Email    检验参数是否是邮箱格式
- @Pattern(regexp = “a{0,1}”)    使用正则表达式校验字符串
- @CreditCardNumber()    是否是美国的信用卡号
- @Length(min = 1,max = 100)    校验字符串的长度是否满足要求
- @Range(min = 1,max = 2)    校验数字的值
- @SafeHtml    校验字符串是否是安全的html
- @URL    校验url是否是合法的url
- @AssertFalse    校验值是否是false
- @AssertTrue    校验值是否是true
- @DecimalMax(value = “1.00”,inclusive = true)    校验数字或者是字符串是否小于等于某个值，inclusive为false的时候为小于
- @DecimalMin(value = “2.00”,inclusive = false)    校验数字或者是字符串是否大于等于某个值，inclusive为false的时候为大于
- @Digits(integer = 1,fraction = 2)    校验数字的格式　integer指定整数部分的长度 fraction指定小数部分的长度
- @Past    日期必须是过去的日期
- @Future    日期必须是未来的日期
- @Max(value = 1)    小于等于，不能注解在字符串上
- @Min(2)    大于等于，不能注解在字符串上
- @JsonFormat、@DateTimeForma  时间格式校验

4. spring提供了多种方式将异常转换为响应：
- a.特定的Spring异常将会自动映射为指定的HTTP状态码
- b.异常上可以添加@ResponseStatus注解，从而将其映射为某一个HTTP状态码
- c.在方法上可以添加@ExceptionHandler注解，使其用来处理异常

5. 在默认情况下，Spring会将自身的一些异常自动转换为合适的状态码。
- BindException 400 - Bad Request
- ConversionNotSupportedException 500 - Internal Server Error
- HttpMediaTypeNotAcceptableException 406 - Not Acceptable
- HttpMediaTypeNotSupportedException 415 - Unsupported Media Type
- HttpMessageNotReadableException 400 - Bad Request
- MissingServletRequestParameterException 400 - Bad Request
- MissingServletRequestPartException 400 - Bad Request
- NoSuchRequestHandlingMethodException 404 - Not Found
- TypeMismatchException 400 - Bad Request
- HttpMessageNotWritableException 500 - Internal Server Error
- HttpRequestMethodNotSupportedException 405 - Method Not Allowed

6. 当控制器的结果是重定向的话，原始的请求就结束了，并且会发起一个新的GET请求。原始请求中所带有的模型数据也就随着请求一起消亡了。在新的请求属性中，没有任何的模型数据,这个请求必须要自己计算数据。一些其他方案，spring能够从发起重定向的方法传递数据给处理重定向方法中：
- 使用URL模板以路径变量和/或查询参数的形式传递数据
- 通过flash属性发送数据

7. 通过路径变量和查询参数的形式跨重定向传递数据是很简单直接的方式，但它也有一定的限制。它只能用来发送简单的值，如String和数字的值。在URL中，并没有办法发送更为复杂的值，但这正是flash属性能够提供帮助的领域。实际上，Spring也认为将跨重定向存活的数据放到**会话**中是一个很不错的方式。但是，Spring认为我们并不需要管理这些数据，相反，Spring提供了将数据发送为flash属性（flashattribute）的功能。按照定义，flash属性会一直携带这些数据直到下一次请求，然后才会消失。

***

8. ** 流程执行器（flow executor ）**就是用来驱动流程的执行。当用户进入到一个流程时，流程执行器会为该用户创建并启动一个流程执行器的实例。当流程暂停时（例如为用户展示视图时），流程执行器会在用户执行操作后恢复流程。在Spring中，``` <flow:flow-executor> ```元素可以创建一个流程执行器：```<flow:flow-executor id="flowExecutor" />```尽管流程执行器负责创建和执行流程，但它并不负责加载流程定义。这个要由流程注册表（flow registry）负责，下面会创建它。


9. 流程注册表的工作就是加载流程定义，并让流程执行器可以使用它们。可以在Spring中使用```<flow:flow-registry>```进行配置：
```
<flow:flow-registry id="flowRegistry" base-path="/WEB-INF/flows">
    <flow:flow-location-pattern value="/**/*-flow.xml" />
</flow:flow-registry>
```
10. 在Spring Web Flow中，流程是由3个主要元素组成的：**状态（state）**、**转移（transition）**和**流程数据（flow data）**。

11. 状态类型对应的状态作用
- 1、行为（Action） 	-->流程逻辑发生的地方
- 2、决策（Decision） 	-->决策状态将流程分为两个方向，它会基于流程数据的评估结果确定流程方向
- 3、结束（End） 	    -->结束状态是流程的最后一站，进入End状态，流程就会终止
- 4、子流程（Subflow） 	-->子流程状态会在当前正在运行的流程上下文中启动一个新的流程
- 5、视图（View） 	    -->视图状态会暂停流程并邀请用户参与流程

12. 转移：转移连接了流程中的状态。流程中除结束状态外的每个状态，至少需要一个转移，这样就知道在状态完成时的走向。一个状态也许有多个转移，分别表示当前状态结束时可以执行的不同路径。
- 转移是通过```<transition>```元素来定义的，作为其他状态元素```（<action-state>、<view-state>和<subflow-state>）```的子元素。最简单的形式就是```<transition>```元素在流程中指定下一个状态：``` <transition to="customerReady" />  ```,在任意事件中，你可以使用on属性来指定触发转移的事件：
```
 <transition on="phoneEntered" to="lookupCustomer"/> 
```
- 全局转移:与其在多个流程状态中重复通用的转移，不如将其作为```<globaltransitions>```的子元素，从而作为全局转移:
```
<global-transitions>
    <transition on="cancel" to="endState" />
</global-transitions>
```

13. 流程数据:当流程从一个状态到达另一个状态时，它会带走一些数据。有时这些数据很快就会被使用，比如直接展示给用户，有时这些数据需要在整个流程中传递并在流程结束时使用。流程数据是保存在变量中的，而变量可以在流程的任意位置进行引用，并且可以以多种方式进行创建。其中最简单的方式就是使用```<var>```元素：
```
使用<var>元素来创建变量
<var name="customer" class="com.springinaction.pizza.domain.Customer"/>

使用<evaluate>元素来创建变量
<evaluate result="viewScope.toppingsList"
    expression="T(com.springinaction.pizza.domain.Topping).asList()" />

使用<set>元素来创建变量
<set name="flowScope.pizza"
    value="new com.springinaction.pizza.domain.Pizza()" />
```

14. 流程数据的作用域
- 1、Conversation 	-->最高层级的流程开始时创建，在最高层级的流程结束时销毁。由最高层级的流程和其所有的子流程所共享
- 2、Flow 	        -->当流程开始时创建，在流程结束时销毁。只在创建它的流程中是可见的
- 3、Request 	    -->当一个请求进入流程时创建，流程返回时销毁
- 4、Flash      	-->流程开始时创建，流程结束时销毁。在视图状态解析后，才会被清除
- 5、View 	        -->进入视图状态时创建，退出这个状态时销毁，只在视图状态内可见

***

15. Spring Security是为基于Spring的应用程序提供声明式安全保护的安全性框架。Spring Security提供了完整的安全性解决方案，它能够在Web请求级别和方法调用级别处理身份认证和授权。因为基于Spring框架，所以Spring Security充分利用了依赖注入和面向切面的技术。Spring Security从两个角度来解决安全性问题。它使用Servlet规范中的Filter保护Web请求并限制URL级别的访问。Spring Security还能够使用Spring AOP保护方法调用—借助于对象代理和使用通知，能够确保只有具备适当权限的用户才能访问安全保护的方法。

16. 为了让Spring Security满足我们应用的需要，还需要再添加一点配置。具体来讲，我们需要：
- 配置用户储存；
- 指定哪些请求需要认证，哪些请求不需要认证，以及所需要的权限；
- 提供一个自定义的登录页面，替代原来简单的默认登录页

17. 我们可以通过重载WebSecurityConfigurerAdapter的三个configure()方法来配置Web安全性，这个过程中会使用传递进来的参数设置行为。下表描述了这三个方法。
|方法|描述|
|---|---|
|configure(WebSecurity)|通过重载，配置Spring Security的Filter链|
|configure(HttpSecurity)|通过重载，配置如何通过拦截器保护请求|
|configure(AuthenticationManagerBuilder)|通过重载，配置user-detail服务|

18. 使用基于内存的用户存储
```
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()  // 启动内存用户存储
                .withUser("user").password("password").roles("USER").and()
                .withUser("admin").password("password").roles("USER","ADMIN");
    }
}

方法
描述

accountExpired(boolean)
定义账号是否已经过期

accountLocked(boolean)
定义账号是否已经锁定

and()
用来连接配置

authorities(GrantedAuthority...)
授予某个用户一项或多项权限

authorities(List<? extends GrantedAuthority>)
授予某个用户一项或多项权限

authorities(String...)
授予某个用户一项或多项权限

credentialsExpired(boolean)
定义凭证是否已经过期

disabled(boolean)
定义账号是否已经被禁用

password(String)
定义用户的密码

roles(String...)
授予某个用户一项或多项角色

```

19. 基于数据库表进行认证.用户数据通常会存储在关系型数据库中，并通过JDBC进行访问。为了配置Spring Security使用以JDBC为支撑的用户存储，我们可以使用jdbcAuthentication()方法，所需的最少配置如下所示：
```
@Autowired
private DataSource dataSource;

/**
    * 配置Spring Security的Filter链
    * @param auth
    * @throws Exception
    */
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.jdbcAuthentication()
            .dataSource(dataSource);
}
```

20. 这里唯一的问题在于如果密码明文存储的话，会很容易收到黑客的窃取。但是，如果数据库中的密码进行了转码的话，那么认证就会失败，因为它与用户提交的明文密码并不匹配。
```
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.jdbcAuthentication()
            .dataSource(dataSource)
            .usersByUsernameQuery(
                    "select username, password,  true " +
                    "from Spitter where username=?")
            .authoritiesByUsernameQuery(
                    "select username, 'ROLE_USER' from Spitter where username=?")
            .passwordEncoder(new StandardPasswordEncoder("53cr3t"));
}


//不管你使用哪一个密码转码器，都需要理解的一点是，数据库中的密码是永远不会解码的。所采取的策略与之相反，用户在登录时输入的密码会按照相同的算法进行转码，然后再在数据库中已经转码过的密码进行对比。这个对比是在PasswordEncoder的matches()方法中进行的。
```

21. 配置自定义的用户服务。假设我们需要认证的用户存储在非关系型数据库中，如Mongo或Neo4j，在这种情况下，我们需要提供一个自定义的UserDetailsService接口实现。UserDetailsService接口非常简单：
```
public interface UserDetailsService {
UserDetails loadUserByUsername(String username) 
                                    throws UsernameNotFoundException;
}
```

22. 在任何应用中，并不是所有的请求都需要同等程度地保护。有些请求需要认证，而另一些可能并不需要。有些请求可能只有具备特定权限的用户才能访问，没有这些权限的用户会无法访问。对每个请求进行细粒度安全性控制的关键在于重载configure(HttpSecurity)方法。如下的代码片段展现了重载的configure(HttpSecurity)方法，它为不同的URL路径有选择地应用安全性：
```
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
        .authorizeRequests()
            .antMatchers("/spitter/me").authenticated()
            .antMatchers(HttpMethod.POST, "/spittles").authenticated()
            .anyRequest().permitAll();
}
```

23. 除了authenticated()和permitAll()方法以外，还有其他的一些方法能够用来定义该如何保护请求。下表描述了所有可用的方案:
|方法|能够做什么|
|---|---|
|access(String)|如果给定的SpEL表达式计算结果为true，就允许访问|
|anonymous()|允许匿名用户访问|
|authenticated()|允许认证过的用户访问|
|denyAll()|无条件拒绝所有访问|
|fullyAuthenticated()|如果用户是完整认证的话（不是通过Remember-me功能认证的），就允许访问|
|hasAnyAuthority(String...)|如果用户具备给定权限中的某一个的话，就允许访问|
|hasAnyRole(String...)|如果用户具备给定角色中的某一个的话，就允许访问|
|hasAuthority(String)|如果用户具备给定权限的话，就允许访问|
|hasIpAddress(String)|如果请求来自给定IP地址的话，就允许访问|
|hasRole(String)|如果用户具备给定角色的话，就允许访问|
|not()|对其他访问方法的结果求反|
|permitAll()|无条件允许访问|
|rememberMe()|如果用户是通过Remember-me功能认证的，就允许访问|

24. 使用Spring表达式进行安全保护,借助access()方法，我们也可以将SpEL作为声明访问限制的一种方式。例如，如下就是使用SpEL表达式来声明具有“ROLE_SPITTER”角色才能访问“/spitter/me”URL:
```
.antMatchers("/spitter/me").access("hasRole('ROLE_SPITTER')")

安全表达式
计算结果

authentication
用户的认证对象

denyAll
结果始终为false

hasAnyRole(list of roles)
如果用户被授予了列表中任意的指定角色，结果为true

hasRole(role)
如果用户被授予了指定的角色，结果为true

hasIpAddress(IPAddress)
如果请求来自指定的IP的话，结果为true

isAnonymous()
如果当前用户为匿名用户，结果为true

isAuthenticated()
如果当前用户进行了认证的话，结果为true

isFullAuthenticated()
如果当前用户进行了完整认证的话（不是用过Remember-me功能进行的认证），结果为true

isRememberMe()
如果当前用户是通过Remember-me自动认证的话，结果为true

permitAll
结果始终为true

principal
用户的principal对象
```

25. 强制通道的安全性,使用HTTP提交数据是一件具有风险的事情。通过HTTP发送的数据没有经过加密，黑客就有机会拦截请求并且能够看到他们想看的数据。这就是为什么敏感信息要通过HTTPS来加密发送的原因。为了保证注册表单的数据通过HTTPS传送，我们可以在配置中添加requiresChannel() 方法，如下所示：
```
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            .antMatchers("/spitter/me").hasRole("SPITTER")
            .antMatchers(HttpMethod.POST, "/spittles").hasRole("SPITTER")
            .anyRequest().permitAll()
            .and() 
            .requiresChannel()
            .antMatchers("/spitter/form").requiresSecure();   // 需要 HTTPS
    }
```
26. 防止跨站请求伪造,当一个POST请求提交到“/spittles”上，SpittleController将会为用户创建一个新的Spittle对象。但是如果这个POST请求来源于其他站的话，这就是跨站请求伪造（cross-site request forgery，CSRF）的一个简单样例。简单来讲，如果一个站点欺骗用户提交请求到其他服务器的话，就会发生CSRF攻击，这可能会带来消极的后果。

27. 启用HTTP Basic认证
```
 @Override
protected void configure(HttpSecurity http) throws Exception {
    http.formLogin()
            .loginPage("/login")// 如果不设置，则默认为/login
            .and()
            .httpBasic()
                .realmName("Spittr")
            .and()
    ...
}
//注意，和前面一样，在configure()方法中，通过调用and()方法来将不同的配置指令连接在一起。
```

28. 启用Remember-me功能
```
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.formLogin()
            .loginPage("/login")// 如果不设置，则默认为/login
            .and()
            .rememberMe()
                .tokenValiditySeconds(2419200)
                .key("spittrKey"); // cookie中的私钥
}
//
```

29. 退出
```
<a href="<c:url value="/logout" />">Logout</a>

//当用户点击这个链接的时候，会发起对“/logout”的请求，这个请求会被Spring Security的LogoutFilter所处理。用户会退出应用，所有Remember-me token都会被清除掉。在退出完成后，用户浏览器将会重定向“/login?logout”，从而允许用户进行再次登录。
```