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

8. **流程执行器（flow executor ）**就是用来驱动流程的执行。当用户进入到一个流程时，流程执行器会为该用户创建并启动一个流程执行器的实例。当流程暂停时（例如为用户展示视图时），流程执行器会在用户执行操作后恢复流程。在Spring中，``` <flow:flow-executor> ```元素可以创建一个流程执行器：```<flow:flow-executor id="flowExecutor" />```尽管流程执行器负责创建和执行流程，但它并不负责加载流程定义。这个要由流程注册表（flow registry）负责，下面会创建它。


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

15. 
