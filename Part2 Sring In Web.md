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

4. 



