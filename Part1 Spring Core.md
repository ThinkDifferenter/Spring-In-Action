# Chapter 1 Spring Travels
1. Spring最根本的使命是简化Java的开发。

2. 为了降低Java开发的复杂性，spring采取以下4种关键策略：
- a.基于POJO的轻量级和最小入侵性编程；
- b.通过依赖注入和面向切面实现松耦合；
- c.基于切面和惯例进行声明式编程；
- d.通过切面和模板减少样板代码；
几乎spring所做的任何事都可以追溯到上述的一条或多条策略。

3. 耦合具有两面性，一方面，紧耦合的代码难以测试，难以复用，难以理解，并且典型的表现出“打地鼠”式的bug特性（修复一个bug，将会出现一个或者多个新的bug）。另一方面，一定程度的耦合是必须的-完全没有耦合的代码什么也做不了。总而言之，耦合是必须的，但应当谨慎管理。

4. 如果一个对象只通过接口（而不是具体实现或者初始化过程）来表明关系，那么这种依赖就能够在对象本身毫不知情的情况下，用不同的具体实现进行替换。

5. DI能够让祥和与协作的软件组件保持松散耦合，而面向切面编程允许你把遍布应用各处的功能分离出来形成可重用的组件。

6. 诸如日志、事务管理和安全这样的系统服务经常融入到自身具有核心业务逻辑的组件中去，这样的系统服务通常被称为**横切关注点**，应为它们会跨域系统的多个组件。

7. 如果将这些关注点分散到多个组件中去，你的代码将会带来双重的复杂性：
- a.实现系统关注点功能的代码将会重复出现在多个组件中。这意味着如果你要改动这些关注点的逻辑，必须修改各个模块中的相关实现。即使你把这样的关注点抽象为一个独立的模块，其他模块只是调用它的方法，但方法的调用还是会重复出现在各个模块中；
- b.组件会因为那些与自身核心业务无关的代码而变得混乱。一个向地址簿增加地址条目的方法应该只关注如何添加地址，而不是应该关注它是不是安全的或者是否需要支持事物。

8. bean的生命周期：Start->实例化-> 填充属性 ->调用BeanNameAware的setName()方法 ->调用BeanNameAware的setFacto()方法 ->调用ApplicationContextAware的setApplicationContext()方法 ->调用BeanPostProcessor的预初始化方法 ->调用InitializingBean的afterPropertiesSet()方法 ->调用自定义的初始化方法 ->调用BeanPostProcessor的初始化后方法 ->bean可以使用了 ->容器关闭 ->调用DisposiableBean的destroy()方法 ->调用自定义销毁方法 ->End

9. spring boot大量依赖于自动配置技术，很大程度上减少了spring配置。

10. 在spring中，对象无需自己查找或创建与其所关联的其他对象，相反，容器负责吧需要协作的对象引用赋予各个对象。

11. 创建应用对象之间的协作关系的行为通常称为**装配**，这也是**依赖注入**的本质。

12. spring具有非常大的灵活性，它提供了三种主要的装配机制：
- a.在XML中进行显示的装配；
- b.在Java中进行显示的装配；
- c.隐式的bean发现机制和自动装配；
我们建议尽可能的使用自动装配机制，显示的装配越少使用越好。

***

13. spring从两个方面来实现自动装配：
- a.**组件扫描（Component Scanning）**：spring会自动发现应用上下文中所创建的的bean；
- b.**自动装配（autowiring）**：spring自动满足bean之间的依赖；
组件扫描和自动装配组合在一起能将显示配置降低到最小。

14. spring支持将@Named作为@Component注解的替代方式，在大多数场景中两者完全等效，不过@Named没有像@Component那样清楚的说明它是做什么的。

15. 组件扫描和自动装配逻辑过程：声明接口compactDisc-> 创建接口接口compactDisc的实现类SgtPeppers并打上@Component注解-> 创建CDPlayerConfig配置类并打上@Configuration和@ComponentScan注解->创建单元测试类CDPlayerTest并在声明接口成员变量cd上打上@Autowired注解。

16. 带有@Bean注解的方法可以采用任何必要的Java功能来产生bean实例，构造器和setter方法（构造注入和设值注入）只是@Bean方法的两个简单实例。

17. 没有指定profile的bean始终都会创建，与激活哪个profile没有关系。

18. 可以在根`<beans>`元素中嵌套定义`<beans>`元素，而不是为每个环境都创建一个profile XML文件，这能够将所有的profile bean定义到一个XML文件中。

19. XML配置文件中嵌套定义`<beans>`实现所有的profile bean在一个文件中：
```
<beans>
    <beans profile="dev">
        ...
    </beans>    

    <beans profile="prod">
        ...
    </beans>
    ...
</beans>
```

***

20. spring在确定哪个profile处于激活状态时，需要依赖两个独立的属性：spring.profiles.active 和 spring.profiles.defualt。如果两者都没有设置的话，那么就没有profile被激活，因此只会创建那些没有定义在profile中的bean。注意到两个属性的profiles均为复数形式，这意味着你可以同时激活多个不相关的profile，中间用逗号隔开即可。

21. 如果你希望一个或者多个bean只有在应用的类路径下包含特定的库时才创建，或者希望某个bean只有当另外某个特定的bean声明时才创建，或者要求某个特定的环境变量设置之后，才创建某个bean。spring4引入@Conditional注解，它可以用到带#Bean注解的方法上，如果计算的结果为true才创建该bean。

22. 从spring4开始，对@Profile注解进行了重构，使其基于@Conditional和Condition实现，@Profile注解具体实现如下：
```
...(原注解)
@Conditional(ProfileCondition.class)
public @interface Profile(){
    String[] value();
}
```

23. 对于仅有一个确定的bean需要创建时，自动装配才是有效的。如果存在歧义，会阻碍spring自动装配属性、构造器参数或者方法参数，并抛出NoUniqueBeanDefinitionException。（如@Component注解打在接口上，但是该接口有多个实现类，在自动装配时存在歧义）

24. 在声明bean的时候，可通过@Primary注解设置首选装配的bean，形式如下：
```
@Component
@Primary
public class IceCream implement Dessert{...} 

or

@Bean
@Primary
public Dessert Icecream(){
    return new Icecream();
}
```
不管你用什么方式来标识首选bean，但是对于多个首选bean的标识就会出现问题了。

25. 利用@Autowired和@Qualifier注解设置bean装配限定 
```
@Autowired
@Qualifier("iceCream")
public void setDessert(Dessert dessert){
    this.dessert = dessert;
}

// 组件扫描+自动装配时bean的ID为首字母小写的类名，不过这里的问题在于限定符与注入对象bean名称是耦合的，这意味着修改类的名称会使限定符失效。
```
26. 当使用自定义的@Qualifier注解是，最佳的实践是bean选择特征为描述性的术语，而不是随意的命名。

27. java不会允许在同一个条目上重复出现相同类型的多个注解。（Java 8允许出现重读的注解，只要这一个注解本身在定义的时候带有@Repeatable注解就可以了，spring的@Qualifier注解并没有带上这个注解）

***

28. 在默认的情况下，spring所创建的bean都是单例的，即不管给定的bean被注入到其他bean多少次，都是同一个实例。

29. 大多数情况下，单例的bean都能满足需求，不过当你所使用的类是易变的，因为对象会被污染，稍后重用的时候会出现意想不到的问题。

30. spring定义了多种作用域，可以基于这些作用域创建bean：
- a.单例（Singleton）:在整个应用中，只创建bean的一个实例；
- b.原型（Prototype）：每次注入或者通过spring应用上下文获取的时候，都会创建新的bean实例；
- c.会话（Session）：在web应用中，为每个会话创建一个bean的实例；
- d.请求（Request）：在web应用中，为每个请求创建一个bean的实例；

31. 通过@Scope与@Bean和@Component实现bean作用域的设置：
```
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class NotePad(){
    ....
}

or 

@Bean
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public NotePad notepad(){
    return new NotePad();
}
```

32. 使用会话和请求作用域
```
@Component
@Scope(ConfigurableBeanFactory.SCOPE_SESSION,
    proxyMode=ScopeProxyMode.INTERFACES)
public ShoppingCart cart(){...}

proxyMode=ScopeProxyMode.INTERFACES 这个属性解决了将会话或请求作用域的bean注入到单例bean中所遇到的问题。
```

33. 运行时注入bean的属性或者构造器参数：为了减少硬编码，spring提供两种运行时求值得方法
- a.属性占位符
- b.spring 表达式语言

34. 属性占位符
```
@Configuration
@PropertySource("classpath:...配置文件路径")
public class ExpressiveConfig{
    @AutoWired
    Environment env;

    public BlankDisc disc(){
        return new BlankDisc(
            env.getProperty("disc.title");
            env.getProperty("disc.artist");   
        )
    }
}

```

35. 属性占位符
```
public BlankDisc(@Value(${disc.title}) String title,@Value(${disc.artist}) String artist){
    this.artist=disc.artist;
    this.title=disc.title;
}

// 解析外部属性能够将值得处理推迟到运行时，但是它的关注点在于根据名称解析来自于spring Environment和属性源的值
```

36. SpEL(Spring Expression Language)是一种强大的和简洁的方式将值装配到bean属性和构造器中，在这个过程中所使用的表达式会在运行时计算得到值，SpEL特性包括以下几点：
- a.使用bean的ID来引用bean；
- b.调用方法和访问对象的属性；
- c.对值进行计算、关系和逻辑运算；
- d.正则表达式匹配；
- e.集合操作

37. SpEL使用样例：
- a.表示字面值：
```
#{3.14159} ,#{6.18E4}, #{'hello worlds'}, #{false}   
```
- b.引用bean、属性和方法
```
#{sgtPeppers}, #{sgtPeppers.title}, #{artistSelector.selectArtist()} 
```
- c.在表达式中使用类型
```
//如果在SpEL中访问类作用域的方法和常亮的话，要依赖T()这个关键的运算符
T(java.lang.Math)

//T()运算符真正的意义在于它能够方位目标类型的静态方法和常亮。
T(java.lang.Math).PI
T(java.lang.Math).random()
```
- d.SpEL运算符
```
//计算圆的周长
#{2*sT(java.lang.Math).PI*cicrcle.radius}

//数值型的判断
#{counter.total == 100}

or

//文本型的判断
#{counter.total eq 100}
```
- e.计算正则表达式
```
//在文本处理时检查某种模式的存在非常有用
#{adim.email matches '[...正则表达式]'}
```
- f.计算集合
```
//定位集合中的摸个元素(下标从零开始)
#{jukebox.song[4].title}
#{'hello world'[3]}

#{jukebox.song[T(java.lang.Math).random()*jokebox.song.size()].title}
```

***

