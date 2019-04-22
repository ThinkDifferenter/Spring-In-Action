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
```
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
```
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
```
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
```
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
```
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

10. 



