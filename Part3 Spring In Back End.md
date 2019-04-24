1. 为了避免持久化的逻辑分散到应用的各个组件中，最好将数据访问的功能放到一个或多个专注于此项任务的组件中。这样的组件通常称为数据访问对象（ data access object ， DAO ）或 Repositor。为了避免应用与特定的数据访问策略耦合在一起，编写良好的 Repository 应该以接口的方式暴露功能。spring的目标之一就是允许我们在开发因应用时能够遵循面向对象原则中"面向接口编程"。

2. JDBC对于所有的数据访问问题都会抛出 SQLException ，可能导致抛出 SQLException 的常见问题包括：
- 应用程序无法连接数据库
- 要执行的查询存在语法错误；
- 查询中所使用的表和 / 或列不存在；
- 试图插入或更新的数据违反了数据库约束
可见JDBC对于数据访问异常的处理十分粗糙，他并不会告诉你具体问题出现在哪里，而是讲所有问题都指向SQLException。事实上， 能够触发SQLException的问题通常是不能在catch代码块中解决的。 大多数抛出SQLException的情况表明发生了致命性错误。 如果应用程序不能连接到数据库， 这通常意味着应用不能继续使用了。 类似地， 如果查询时出现了错误， 那在运行时基本上也是无能为力。

3. 模板方法模式（设计模式）：模板方法定义过程的主要框架，模板方法将过程中与特定实现相关的部分委托给接口，而这个接口的不同实现定义了过程中的具体行为。Spring将数据访问分为可变（模板）和不可变（固定）两部分：模板（ template ）和回调（ callback ）。

4. Spring提供了在Spring上下文中配置数据源bean的多种方式，包括：
- 通过JDBC驱动程序定义的数据源；
- 通过JNDI查找的数据源；
- 连接池的数据源。

5. 使用JNDI数据源：这种配置的好处在于数据源完全可以在应用程序之外进行管理，这样应用程序只需在访问数据库的时候查找数据源就可以了。另外，在应用服务器中管理的数据源通常以池的方式组织，从而具备更好的性能，并且还支持系统管理员对其进行热切换。
```java
@Bean
public JndiObjectFactoryBean dataSource(){
    JndiObjectFactoryBean jndiObjectFB = new JndiObjectFactoryBean();
    jndiObjectFB.setJndiName("jdbc/SpitterDS");
    jndiObjectFB.setResourceRef(true);
    jndiObjectFB.setProxyInterface(javax.sql.DataSource.class);
    return jndiObjectFB;
}
```

6. 使用数据源连接池：尽管Spring并没有提供数据源连接池实现，但是我们有多项可用方案，包括如下开源的实现：
- Apache Commons DBCP（http://jakarta.apache.org/commons/dbcp）
- Druid （阿里巴巴的开源项目，https://github.com/alibaba/druid/wiki）
- c3p0 （http://sourceforge.net/projects/c3p0/）
- BoneCP (http://jolbox.com/)
```java
@Bean
public DruidDataSource dataSource(){
    DruidDataSource ds = new DruidDataSource();
    ds.setDriverClassName("com.mysql.cj.jdbc.Driver");
    ds.setUrl("jdbc:mysql://localhost:3306/spitter");
    ds.setUsername("root");
    ds.setPassword("123456");
    ds.setInitialSize(5);
    ds.setMaxActive(10);
    return ds;
}
```

7. 基于JDBC驱动的数据源:Spring提供了三个这样的数据源类（均位于org.springframework.jdbc.datasource包中）供选择：
- DriverManagerDataSource：在每个连接请求时都会返回一个新建的连接。与DBCP的BasicDataSource不同，由DriverManagerDataSource提供的连接并没有进行池化管理。
- SimpleDriverDataSource: 与DriverManagerDataSource的工作方式类似，但是它直接使用JDBC驱动，来解决在特定环境下的类加载问题，这样的环境包括OSGi容器；
- SingleConnectionDataSource：在每个连接请求时都会返回同一个的连接。尽管SingleConnectionDataSource不是严格意义上的连接池数据源，但是可以将其视为只有一个连接的池。
```java
@Bean
public DataSource dataSource(){
    DriverManagerDataSource ds = new DriverManagerDataSource();
    ds.setDriverClassName("com.mysql.cj.jdbc.Driver");
    ds.setUrl("jdbc:mysql://localhost:3306/spitter");
    ds.setUsername("root");
    ds.setPassword("123456");
    return ds;
}
```

8. 使用嵌入式的数据源:嵌入式数据库（embedded database）。嵌入式数据库作为应用的一部分运行，而不是应用连接的独立数据库服务器。对于开发和测试来讲，嵌入式数据库是很好的可选方案。
```java
@Bean
public DataSource dataSource(){
    return new EmbeddedDatabaseBuilder()
             .setType(EmbeddedDatabaseType.H2)
             .addScript("classpath:schema.sql")
             .addScript("classpath:test-data.sql")
             .build();
}
```

9. 使用profile选择数据源:我们很可能面临这样一种需求，那就是在某种环境下需要其中一种数据源，而在另外的环境中需要不同的数据源
```java
@Configuration
public class DataSourceConfig {
  
  @Bean(destroyMethod = "shutdown")
  @Profile("dev")
  public DataSource embeddedDataSource() {
    ...
  }

  @Bean
  @Profile("prod")
  public DataSource jndiDataSource() {
    ...
  }


   @Bean()
   @Profile("qa")
   public DataSource dataSource(){
       ...
   }
}
```

10. JdbcTemplate主要提供以下五类方法：
- execute方法：可以用于执行任何SQL语句，一般用于执行DDL语句
- update方法及batchUpdate方法：update方法用于执行新增、修改、删除等语句；batchUpdate方法用于执行批处理相关语句**
- query方法及queryForXXX方法：用于执行查询相关语句
- call方法：用于执行存储过程、函数相关语句

11. 数据状态相关概念：
- 瞬时状态：在程序运行的时候，有些数据保存在内存中，当程序退出后，这些数据就不复存在了，称这些数据的状态是瞬时的。
- 持久状态：数据以文件形式保存在辅存中，这样，程序退出后，数据依然存在，这种状态称之为持久的。
- 持久化 ：即在程序中的瞬时状态和持久状态之间转换的机制。
实际上，我们通常所说的持久化，一般指的持久化数据到数据库中。

12. 数据库的几个特性：
- 延迟加载（ Lazy loading ）：随着我们的对象关系变得越来越复杂，有时候我们并不希望立即获取完整的对象间关系。举一个典型的例子，假设我们在查询一组 PurchaseOrder 对象，而每个对象中都包含一个 LineItem 对象集合。如果我们只关心 PurchaseOrder 的属性，那查询出 LineItem 的数据就毫无意义。而且这可能是开销很大的操作。延迟加载允许我们只在需要的时候获取数据。
- 预先抓取（ Eager fetching ）：这与延迟加载是相对的。借助于预先抓取，我们可以使用一个查询获取完整的关联对象。如果我们需要 PurchaseOrder 及其关联的 LineItem 对象，预先抓取的功能可以在一个操作中将它们全部从数据库中取出来，节省了多次查询的成本。
- 级联（ Cascading ）：有时，更改数据库中的表会同时修改其他表。回到我们订购单的例子中，当删除 Order 对象时，我们希望同时在数据库中删除关联的 LineItem 一些可用框架提供了上述服务，这些服务的通用名称是对象 / 关系映射（ object-relational mapping ， ORM ）。在持久层使用 ORM 工具，可以节省数千行的代码和大量的开发时间。 ORM 工具能够把你的注意力从容易出错的 SQL 代码转向如何实现应用程序的真正需求。

13. 当创建Repository实现的时候， Spring Data会检查Repository接口的所有方法， 解析方法的名称， 并基于被持久化的对象来试图推测方法的目的。 本质上， Spring Data定义了一组小型的领域特定语言（domain-specific language ， DSL） ， 在这里， 持久化的细节都是通过Repository方法的签名来描述的

14. Repository方法的命名遵循模式：查询动词、主题、断言
- Spring Data允许在方法名中使用四种动词： get、 read、 find和count。 其中， 动词get、 read和find是同义的， 这三个动词对应的Repository方法都会查询数据并返回对象。 而动词count则会返回匹配对象的数量， 而不是对象本身；
- Repository方法的主题是可选的。 它的主要目的是让你在命名方法的时候， 有更多的灵活性。对于大部分场景来说， 主题会被省略掉。在省略主题的时候， 有一种例外情况。 如果主题的名称以Distinct开头的话， 那么在生成查询的时候会确保所返回结果集中不包含重复记录。
- 断言是方法名称中最为有意思的部分， 它指定了限制结果集的属性。注意， 参数的名称是无关紧要的， 但是它们的顺序必须要与方法名称中的操作符相匹配。

15. 如果所需的数据无法通过方法名称进行恰当地描述， 那么我们可以使用@Query注解， 为Spring Data提供要执行的查询。 对于findAllGmailSpitters()方法， 我们可以按照如下的方式来使用@Query注解：
```java
@Query("update User bean set bean.recommend=?2,bean.birthDate=?3 where bean.id=?1")
public void recommended(Integer id,Integer recommend,Date birth);
```
对于Spring Data JPA的接口来说， @Query是一种添加自定义查询的便利方式。 但是， 它仅限于单个JPA查询。 

16. 有些数据具有明显的关联关系， 文档型数据库并没有针对存储这样的数据进行优化。 例如， 社交网络表现了应用中不同的用户之间是如何建立关联的， 这种情况就不适合放到文档型数据库中。 在文档数据库中存储具有丰富关联关系的数据也并非完全不可能， 但这样做的话， 你通常会发现遇到的挑战要多于所带来的收益。

17. 在使用MongoDB之前，我们首先要配置Spring Data MongoBD ，在Spring配置中添加几个必要的bean。
- 配置MongoClient，以便于访问MongoDB数据库；
- 配置MongoTemplate bean，实现基于模板的数据库访问；
- 启用Spring Data MongoDB的自动化Repository生成功能（不是必须，但强烈推荐）。
```java
@Configuration
@EnableMongoRepositories(basePackages = "orders.db")  // 启动MongoDB的Repository功能
public class MongoConfig {
    @Bean
    public MongoClientFactoryBean mongo(){    // MongoClient Bean (MongoFactoryBean已废弃)
        MongoClientFactoryBean mongo = new MongoClientFactoryBean();
        mongo.setHost("localhost");
        return mongo;
    }
    @Bean
    public MongoOperations mongoTemplate(Mongo mongo){  // MongoTemplate bean
        return new MongoTemplate(mongo, "OrdersDB");
    }
}
```

18. 无状态的组件一般来讲扩展性会更好一些，但它们也会更加倾向于一遍遍地问相同的问题。因为它们是无状态的，所以一旦当前的任务完成，就会丢弃掉已经获取到的所有解答，下一次需要相同的答案时，它们就不得不再问一遍这个问题。对于所提出的问题，有时候需要一点时间进行获取或计算才能得到答案。我们可能需要在数据库中获取数据，调用远程服务或者执行复杂的计算。为了得到答案，这就会花费时间和资源。如果问题的答案变更不那么频繁（或者根本不会发生变化），那么按照相同的方式再去获取一遍就是一种浪费了。除此之外，这样做还可能会对应用的性能产生负面的影响。一遍又一遍地问相同的问题,而每次得到的答案都是一样的，与其这样，我们还不如只问一遍并将答案记住，以便稍后再次需要时使用。**缓存（Caching）** 可以存储经常会用到的信息， 这样每次需要的时候， 这些信息都是立即可用的。 在本章中， 我们将会了解到Spring的缓存抽象。 尽管Spring自身并没有实现缓存解决方案， 但是它对缓存功能提供了声明式的支持， 能够与多种流行的缓存实现进行集成。

19. @EnableCaching和```<cache:annotation-driven />```的工作方式是相同的。它们都会创建一个切面（aspect）并触发Spring缓存注解的切点（pointcut）。根据所使用的注解以及缓存的状态，这个切面会从缓存中获取数据，将数据添加到缓存之中或者从缓存中移除某个值。

20. 下表的所有注解都能运用在方法或类上。当将其放在单个方法上时，注解所描述的缓存行为只会运用到这个方法上。如果注解放在类级别的话，那么缓存行为就会应用到这个类的所有方法上。
|注解|描述|
|:-----:|:------:|
|@Cacheable|表明Spring在调用方法之前，首先应该在缓存中查找方法的返回值。如果这个值能够找到，就会返回缓存的值。否则的话，这个方法就会被调用，返回值会放到缓存之中。|
|@CachePut|表明Spring应该将方法的返回值放到缓存中。在方法的调用前并不会检查缓存，方法始终都会被调用|
|@CacheEvict|表明Spring应该在缓存中清除一个或多个条目|
|@Caching|这是一个分组的注解，能够同时应用多个其他的缓存注解|

21. @Cacheable和@CachePut提供了两个属性用以实现条件化缓存：unless和condition，表面上来看，unless和condition属性做的是相同的事情。但是，这里有一点细微的差别。unless属性只能阻止将对象放进缓存，但是这个方法调用的时候，依然会去缓存中进行查找，如果找到了匹配的值，就会返回找到的值。与之不同，如果condition的表达式计算结果为false，那么在这个方法调用的过程中，缓存是被禁用的。就是说，不会去缓存进行查找，同时返回值也不会放进缓存中。

22. 配置将缓存条目存储在Redis服务器的缓存管理器
```java
@Configuration
@EnableCaching   // 启用缓存
public class CachingConfig {

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory redisConnectionFactory){
        return RedisCacheManager.create(redisConnectionFactory);
    }

    @Bean
    public JedisConnectionFactory jedisConnectionFactory() {
        JedisConnectionFactory jedisConnectionFactory = new JedisConnectionFactory();
        jedisConnectionFactory.afterPropertiesSet();
        return jedisConnectionFactory;
    }

    @Bean
    public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, String> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }  
}
```



