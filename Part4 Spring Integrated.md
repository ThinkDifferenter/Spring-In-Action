1. **远程调用**是客户端应用和服务端之间的会话。在客户端，它所需要的一些共功能并不在该应用的实现范围之内，所以应用要向能提供这些功能的其他系统寻求帮助。而远程应用通过远程服务暴露这些功能。下表概述了每个一个Spring支持的RPC模型，并简要讨论了它们所适用的不同场景。

|RPC模型|适用场景|
|:-----:|:-----:|
|远程方法调用（RMI）| 	不考虑网络限制时（例如防火墙），访问/发布基于Java的服务|
|Hessian或Burlap |	考虑网络限制时，通过HTTP访问/发布基于Java的服务。Hessian是二进制协议，而Burlap是基于XML的(Spring 5.0不支持Burlap了)|
|HTTP invoker |	考虑网络限制，并希望使用基于XML或专有的序列化机制实现Java序列化时，访问/发布基于Spring的服务|
|JAX-PRC和JAX-WS |	访问/发布平台独立的、基于SOAP的Web服务|
在所有的模型中，服务都作为Spring所管理的bean配置到我们的应用中。这是通过一个代理工厂bean实现的，这个bean能够把远程服务像本地对象一样装配到其他bean的属性中去。

2. 使用RMI：RMI是一种实现远程服务交互的好办法，但是它存在某些限制。首先，RMI很难穿越防火墙，这是因为RMI使用任意端口来交互—这是防火墙通常所不允许的。另外一件需要考虑的事情是RMI是基于Java的。这意味这客户端和服务端必须都是用Java开发的。因为RMI使用了Java的序列化机制，所以通过网络传输的对象类型必须要保证在调用两端的Java运行时中是完全相同的版本。

3. RMI
- 优点：RMI使用的是Java本身的序列化机制
- 缺点：由于不是HTTP协议，会有防火墙渗透问题。而且服务端和客户端都必须用Java语言。传输速度慢。

4. 使用Hessian发布远程服务：Hessian和Burlap是Caucho Technology提供的两种基于HTTP的轻量级远程服务解决方案。
- Hessian，基于二进制消息进行交互。可在Java、PHP、Python、C++和C#。由于二进制交互，带宽上更具优势。
- Burlap，基于XML的远程调用技术。它的消息结构尽可能的简单，不需要额外的外部定义语句（例如WSDL或IDL），Spring 5.0不支持Burlap了。

5. HessianServiceExporter是一个Spring MVC控制器，它接收Hessian请求，并把这些请求转换成对被POJO的调用从而将POJO导出为一个Hessian服务。在如下Spring的声明中，HessianServiceExporter会把spitterService bean导出为Hessian服务：
```java
@Bean
public HessianServiceExporter hessianExportedSpitterService(SpitterService service) {
    HessianServiceExporter exporter = new HessianServiceExporter();
    exporter.setService(service);
    exporter.setServiceInterface(SpitterService.class);
    return exporter;
}
```

6. Hessian和Burlap
- 优点：基于HTTP的，解决了防火墙渗透问题。服务端和客户端基本上支持常用语言。传输速度快。
- 缺点：Hessian和Burlap采用了私有的序列化机制

7. 使用Spring的HttpInvoker：Spring的HttpInvoker 是一个新的远程调用模型，作为Spring框架的一部分，能够执行基于HTTP的远程调用（让防火墙不为难），并使用Java的序列化机制。

8. 要记住HTTP invoker有一个重大的限制：客户端和服务端必须都是Spring应用，并且都要基于java。另外，因为使用Java的序列化机制，客户端与服务端必须使用相同版本的类（与RMI类似）。

***

9. Rest含义：
- 表述性(Representational)：REST资源实际上可以用各种形式来进行表述，包括XML、JSON（JavaScript Object Notation）甚至HTML—最适合资源使用者的任意形式；
- 状态(State)：当使用REST的时候，我们更关注资源的状态而不是对资源采取的行为；
- 转移(Transfer)：REST涉及到转移资源数据，它以某种表述性形式从一个应用转移到另一个应用。
更简洁地讲， REST就是将资源的状态以最适合客户端或服务端的形式从服务器端转移到客户端（或者反过来）。

10. REST的动作（HTTP的方法）以及匹配的CRUD动作：
- Create: POST
- Read: GET
- Update: PUT或PATCH
- DELETE: DELETE

11. Spring支持以下方式来创建REST资源：
- 控制器可以处理所有的HTTP方法，包含四个主要的REST方法：GET、PUT、DELETE以及POST。Spring 3.2及以上版本还支持PATCH方法；
- 借助@PathVariable注解，控制器能够处理参数化的URL（将变量输入作为URL的一部分）；
- 借助Spring的视图和视图解析器，资源能够以多种方式进行表述，包括将模型数据渲染为XML、JSON、Atom以及RSS的View实现；
- 可以使用ContentNegotiatingViewResolver来选择最适合客户端的表述；
- 借助@ResponseBody注解和和各种HttpMethodConverter实现，能够替换基于视图的渲染方式；
- 类似地，@RequestBody注解以及HttpMethodConverter实现可以将传入的HTTP数据转化为传入控制器处理方法的Java对象；
- 借助RestTemplate，Spring应用能够方便地使用Rest资源。

12. Spring提供了两种方法将资源的Java表述转换为发送给客户端的表述形式：
- 内容协商（Content negotiation）:选择一个视图，它能够将模型渲染为呈现给客户端表述形式。不过由于它只能决定资源该如何渲染到客户端，并没有涉及到客户端要发送什么样的表述给控制器使用，比如客户端发送JSON或XML，它就无法提供帮助了。而且还有其他限制，**不建议使用**。
- 消息转换器（Message conversion）：通过一个消息转换器将控制器所返回的对象转换为呈现给客户端的表述形式。

13. 使用HTTP信息转换器：如果使用了消息转换功能的话，我们需要告诉Spring跳过正常的模型/视图流程，并使用消息转换器。最简单的方式是为控制器方法添加@ResponseBody注解。例如，如下程序：
```java
@RequestMapping(method=RequestMethod.GET, produces="application/json")
public @ResponseBody List<Spittle> spittles (
@RequestParam(value="max",defaultValue=MAX_LONG_AS_SPRING)) long max,
@RequestParam(value="count",defaultValue="20") int count) {
      return spittleRepository.findSpittles(max, count);
}
```
@ResponseBody注解会告知Spring，我们要将返回的对象作为资源发送给客户端，并将其转换为客户端可接受的表述形式。更具体地讲，DispatcherServlet将会考虑到请求中Accept头部信息，并查找能够为客户端提供所需表述形式的消息转换器（根据类路径下实现库）。

14. 与@ResponseBody类似，@RequestBody也能告诉Spring查找一个消息转换器，将来自客户端的资源表述为对象。例如：
```java
@RequestMapping(method=RequestMethod.POST, consumes="application/json")
public @ResponseBody Spittle saveSpittle(@RequestBody Spittle spittle) {
    return spittleRepository.save(spittle);
}
```
通过使用注解，@RequestMapping表明它只能处理“/spittles”（在类级别的@RequestMapping中进行了声明）的POST请求。POST请求体中预期要包含一个Spittle的资源表述。因为Spittle参数上使用了@RequestBody，所以Spring将会查看请求中的Content-Type头部信息，并查找能够将请求转换为Spittle的消息转换器。

15. Spring 4.0引入了@RestController注解。如果在控制器类上使用@RestController来代替@Controller的话，Spring将会为该控制器的所有处理方法应用消息转换功能。我们不必为每个方法都添加@ResponseBody了。添加@RestController注解，此类中所有处理器方法都不需要使用@ResponseBody注解了，因为控制器使用了@RestController，所有它的方法所返回的对象将会通过**消息转换机制**，产生客户端所需的资源表述。

16. 如果一个处理器方法本应返回一个对象，但由于查找不到相应的对象而返回null。我们考虑一下在这种场景下应该发生什么。至少，状态码不应是200，而应该是404，告诉客户端它们所要求的内容没有找到。如果响应体中能够包含错误信息而不是空的话就更好了。Spring提供了多种方式来处理这样的场景：
- 使用@ResponseStatus注解可以指定状态码；
- 控制器方法可以返回ResponseEntity对象，该对象能够包含更多响应相关的元数据；
- 异常处理器能够应对错误场景，这样处理器方法就能关注于正常的状况。

17. RestTemplate可以减少我们使用HttpClient创建客户端所带来的样板式代码。它定义了36个（只有11个独立方法，其他都是重载这些方法）与REST资源交互的方法，其中的大多数都对应于HTTP的方法。下表展示了这11个独立方法

|方法|描述|
|:---:|:---:|
|delete()|在特定的URL上对资源执行HTTP DELETE操作|
|exchange()||在URL上执行特定的HTTP方法，返回包含对象的ResponseEntity，这个对象是从响应体中映射得到的|
|execute()||在URL上执行特定的HTTP方法，返回一个从响应体映射得到的对象|
|getForEntity()||发送一个HTTP GET请求，返回的ResponseEntity包含了响应体所映射成的对象|
|getForObject()|发送一个HTTP GET请求，返回的请求体将映射为一个对象|
|headForHeaders()|发送HTTP HEAD请求，返回包含特定资源URL的HTTP头|
|optionsForAllow()|发送HTTP OPTIONS请求，返回对特定的URL的Allow头信息|
|postForEntity()|POST数据到一个URL，返回包含一个对象的ResponseEntity，这个对象是从响应体中映射得到的|
|postForLocation()|POST数据到一个URL，返回新创建资源的URL|
|postForObject()||POST数据到一个URL，返回根据响应体匹配形成的对象|
|put()||PUT资源到特定的URL|

***

18. 远程同步调用与消息异步发送 
- 像RMI和Hessian/Burlap这样的远程调用机制是同步的。当客户端调用远程方法时， 客户端必须等到远程方法完成后， 才能继续执行。 即使远程方法不向客户端返回任何信息， 客户端也要被阻塞直到服务完成。
- 消息则是异步发送的，客户端不需要等待服务处理消息， 甚至不需要等待消息投递完成。 客户端发送消息， 然后继续执行， 这是因为客户端假定服务最终可以收到并处理这条消息。

19. 与邮局发送邮件类似， 间接性也是异步消息的关键所在。 当一个应用向另一个应用发送消息时， 两个应用之间没有直接的联系。 相反的是， 发送方的应用程序会将消息交给一个服务， 由服务确保将消息投递给接收方应用程序。在异步消息中有两个主要的概念： **消息代理（message broker）和目的地（destination）**。 当一个应用发送消息时， 会将消息交给一个消息代理。 消息代理实际上类似于邮局。 消息代理可以确保消息被投递到指定的目的地， 同时解放发送者， 使其能够继续进行其他的业务。

20. 尽管不同的消息系统会提供不同的消息路由模式， 但是有两种通用的目的地： 队列（queue） 和主题（topic）。每种类型都与特定的消息模型相关联，分别是点对点模型（队列）和发布/订阅模型（主题）。

21. 点对点消息模型：在点对点模型中， 每一条消息都有一个发送者和一个接收者，当消息代理得到消息时， 它将消息放入一个队列中。 当接收者请求队列中的下一条消息时， 消息会从队列中取出， 并投递给接收者。 因为消息投递后会从队列中删除， 这样就可以保证消息只能投递给一个接收者。尽管消息队列中的每一条消息只被投递给一个接收者， 但是并不意味着只能使用一个接收者从队列中获取消息。 事实上， 通常可以使用几个接收者来处理队列中的消息。在点对点的消息中， 如果有多个接收者监听队列， 我们也无法知道某条特定的消息会由哪一个接收者处理。 这种不确定性实际上有很多好处， 因为我们只需要简单地为队列添加新的监听器就能提高应用的消息处理能力。

22. 发布—订阅消息模型：在发布—订阅消息模型中， 消息会发送给一个主题。 与队列类似， 多个接收者都可以监听一个主题。 但是， 与队列不同的是， 消息不再是只投递给一个接收者， 而是主题的所有订阅者都会接收到此消息的副本。因为对于异步消息来讲， 发布者并不知道谁订阅了它的消息。 发布者只知道它的消息要发送到一个特定的主题——而不知道有谁在监听这个主题。 也就是说， 发布者并不知道消息是如何被处理的。

23. 虽然同步通信比较容易理解， 建立起来也很简单， 但是采用同步通信机制访问远程服务的客户端存在几个限制， 最主要的是：
- 同步通信意味着等待。 当客户端调用远程服务的方法时， 它必须等待远程方法结束后才能继续执行。 如果客户端与远程服务频繁通信，或者远程服务响应很慢， 就会对客户端应用的性能带来负面影响。
- 客户端通过服务接口与远程服务相耦合。 如果服务的接口发生变化， 此服务的所有客户端都需要做相应的改变。
- 客户端与远程服务的位置耦合。 客户端必须配置服务的网络位置， 这样它才知道如何与远程服务进行交互。 如果网络拓扑进行调整， 客户端也需要重新配置新的网络位置。
- 客户端与服务的可用性相耦合。 如果远程服务不可用， 客户端实际上也无法正常运行。

24. 异步消息的优点：
- 无需等待：客户端不需要等待，可以继续执行其他任务。
- 面向消息和解耦：发送异步消息是以数据为中心的，客户端不必了解远程服务的任何规范。
- 位置独立：只要服务能够从队列或主题中获取消息即可，消息客户端根本不需要关注服务来自哪里。
- 确保投递：当发送异步消息时，客户端可以相信消息会被投递。即使在消息发送时，服务无法使用，消息也会被储存起来，直到服务重新可以使用为止。

25. Java消息服务（Java Message Service , JMS）是一个Java标准，定义了使用消息代理的通用API。Spring通过基于模版的抽象为JMS功能提供了支持，这个模版也就是JmsTemplate。Spring还提供了消息驱动POJO的理念：这是一个简单的Java对象，它能够以异步的方式响应队列或主题上的到达的消息。

26. 在Spring中搭建消息代理
```xml
<!-- 我们必须配置JMS连接工厂，让它知道如何连接到ActiveMQ。配置如下： -->
<bean id="connectionFactory" class="org.apache.activemq.spring.ActiveMQConnectionFactory" 
p:brokerURL="tcp:localhost:61616" />

<!-- 也可以使用ActiveMQ的命名空间配置 -->
<amq:connectionFactory id="connectionFactory" brokerURL="tcp://localhost:61616" />

<!-- 下面的<bean>声明定义了一个ActiveMQ队列： -->
<bean id="queue" class="org.apache.activemq.command.ActiveMQQueue"
           c:_="spitter.queue" />

<!-- 命名空间配置如下： -->
<amq:queue id="spittleQueue" physicalName="spitter.queue" />

<!-- 同样，下面<bean>声明定义了一个ActiveMQ主题： -->
<bean id="topic" class="org.apache.activemq.command.ActiveMQTopic"
      c:_="spitter.queue" />

<!-- 命名空间配置如下： -->
<amq:topic id="spittleTopic" physicalName="spitter.topic" />
```

27. 使用Spring的JMS模版:针对如何消除冗长和重复的JMS代码，Spring给出的解决方案是JmsTemplate。JmsTemplate可以创建连接、获得会话以及发送和接收消息。另外，JmsTemplate可以处理抛出的笨拙的JMSException异常。如果在使用JmsTemplate时抛出JMSException异常，JmsTemplate将捕获该异常，然后抛出一个非检查型异常，该异常是Spring自带的JmsException异常的子类。

28. 使用AMQP实现消息功能:AMQP具有多项JMS所不具备的优势。首先，AMQP为消息定义了线路层的协议，而JMS所定义的是API规范。JMS的API协议能够确保所有的实现都能通过的API来使用，但是并不能保证某个JMS实现所发送的消息能够被另外不同的JMS实现所使用。而AMQP的线路层协议规范了消息的格式，消息在生产者和消费者间传送的时候会遵循这个格式。这个AMQP在互相协作方面就要优于JMS—它不仅能跨不同的AMQP实现，还能跨语言和平台。

29. 相比JMS，AMQP另外一个明显的优势在于它具有更加灵活和透明的消息模型。使用JMS的话，只有两种消息模型可供选择：点对点和发布-订阅。这两种模型在AMQP当然都是可以实现的，但AMQP还能够让我们以其他的多种方式来发送消息，这是通过将消息的生产者与存放消息的队列解耦实现的。AMQP的生产者并不会直接将消息发布到队列中。AMQP在消息的生产者以及传递信息的队列之间引入了一种间接的机制：Exchange。消息的生产者将信息发布到一个Exchange。Exchange会绑定到一个或多个队列上它负责将信息路由到队列上。信息的消费者会从队列中提取数据并进行处理。AMQP定义了四种不同类型的Exchange，每一种都有不同的路由算法，这些算法决定了是否要将信息放到队列中。四种标准的AMQP Exchange如下所示：
- Direct: 如果消息的routing key与binding的routing key直接匹配的话，消息将会路由到该队列上；
- Topic: 如果消息的routing key与binding的routing key附和通配符匹配的话，消息将会路由到该队列上；
- Headers: 如果消息参数表中的头信息和值都与binding参数表中相匹配，消息将会路由到该队列上；
- Fanout: 不管消息的routing key和参数表和头信息/值是什么，消息将会路由到所有队列上。
生产者将信息发送给Exchange并带有一个routing key，消费者从队列中获取消息。

***

30. Spring 4.0为WebSocket通信提供了支持，包括：
- 发送和接收消息的低层级API；
- 发送和接收消息的高级API；
- 用来发送消息的模板；
- 支持SockJS， 用来解决浏览器端、 服务器以及代理不支持WebSocket的问题。

31. 使用Spring的低层级WebSocket API:按照其最简单的形式，WebSocket只是两个应用之间通信的通道。位于WebSocket一端的应用发送消息，另外一端处理消息。因为它是全双工的，所以每一端都可以发送和处理消息。WebSocket通信可以应用于任何类型的应用中，但是WebSocket最常见的应用场景是实现服务器和基于浏览器的应用之间的通信。

32. WebSocket是一个相对比较新的规范。虽然它早在2011年底就实现了规范化，但即便如此，在Web浏览器和应用服务器上依然没有得到一致的支持。Firefox和Chrome早就已经完整支持WebSocket了，但是其他的一些浏览器刚刚开始支持WebSocket。服务器端对WebSocket的支持也好不到哪里去。GlassFish在几年前就开始支持一定形式的WebSocket，但是很多其他的应用服务器在最近的版本中刚刚开始支持WebSocket。即便浏览器和应用服务器的版本都符合要求，两端都支持WebSocket，在这两者之间还有可能出现问题。防火墙代理通常会限制所有除HTTP以外的流量。它们有可能不支持或者（还）没有配置允许进行WebSocket通信。

33. SockJS让我们能够使用统一的编程模型，就好像在各个层面都完整支持WebSocket一样，SockJS在底层会提供备用方案。例如，为了在服务端启用SockJS通信，我们在Spring配置中可以很简单地要求添加该功能。

34. 使用STOMP消息:直接使用WebSocket（或SockJS）就很类似于使用TCP套接字来编写Web应用。因为没有高层级的线路协议（wire protocol），因此就需要我们定义应用之间所发送消息的语义，还需要确保连接的两端都能遵循这些语义。不过，好消息是我们并非必须要使用原生的WebSocket连接。就像HTTP在TCP套接字之上添加了请求-响应模型层一样，STOMP在WebSocket之上提供了一个基于帧的线路格式（frame-based wire format）层，用来定义消息的语义。

35. 在使用Spring和STOMP消息功能的时候，我们有三种方式利用认证用户：
- @MessageMapping和@SubscribeMapping标注的方法能够使用Principal来获取认证用户；
- @MessageMapping、@SubscribeMapping和@MessageException方法返回的值能够以消息的形式发送给认证用户；
- SimpMessagingTemplate能够发送消息给特定用户。

***

36. 使用Spring发送邮件
- Spring 自带了一个MailSender的实现也就是JavaMailSenderIpml，我们只需要装配这个实现即可。
- 我们只需要将JavaMailSenderIpml的对象通过@AutoWired注解注入我们发送提供邮件发送服务的Service类，然后在方法中调用即可。

37. 构建丰富内容的Email
- 添加附件：Spring提供了一个MimeMessageHelper，我们可以通过它构建带附件的Email。我们需要先实例化一个MimeMessgae作为构造器参数传给MimeMessageHelper。然后才能使用它。
- 发送富文本：同样是使用MimeMessageHelper，只是将它的setText方法的第二个参数设为true。

38. 使用Thymeleaf构建Email
- 使用SpringTemplateEngine作为Thymeleaf引擎，解析器除了要配置ServletContextTemplateResolver还要配置ClassLoaderTemplateResolver（从类路径解析模板），将这两个解析器注入到SpringTemplateEngine中，通过它的setTemplateResolvers方法。
- 在方法内要做的第一件事就是创建Thymeleaf Context实例，将模型数据set进去，然后在Thymeleaf视图层可以通过${}取出

***

40. Spring对DI的支持是通过在应用中配置bean属性， 这是一种非常不错的方法。 不过， 一旦应用已经部署并且正在运行， 单独使用DI并不能帮助我们改变应用的配置。JMX这项技术能够让我们管理、 监视和配置应用。使用JMX管理应用的核心组件是托管bean（managed bean， MBean） 。 所谓的MBean就是暴露特定方法的JavaBean， 这些方法定义了管理接口。 JMX规范定义了如下4种类型的MBean：
- 标准MBean： 标准MBean的管理接口是通过在固定的接口上执行反射确定的， bean类会实现这个接口；
- 动态MBean： 动态MBean的管理接口是在运行时通过调用DynamicMBean接口的方法来确定的。 因为管理接口不是通过静态接口定义的， 因此可以在运行时改变；
- 开放MBean： 开放MBean是一种特殊的动态MBean， 其属性和方法只限定于原始类型、 原始类型的包装类以及可以分解为原始类型或原始类型包装类的任意类型；
- 模型MBean： 模型MBean也是一种特殊的动态MBean， 用于充当管理接口与受管资源的中介。 模型Bean并不像它们所声明的那样来编写。 它们通常通过工厂生成， 工厂会使用元信息来组装管理接口。

41. 为了对MBean的属性和操作获得更细粒度的控制， Spring提供了几种选择， 包括：
- 通过名称来声明需要暴露或忽略的bean方法；
- 通过为bean增加接口来选择要暴露的方法；
- 通过注解标注bean来标识托管的属性和操作。 

42. 通过名称暴露方法
MBean信息装配器（MBean info assembler） 是限制哪些方法和属性将在MBean上暴露的关键。 其中有一个MBean信息装配器是MethodNameBasedMBean-InfoAssembler。 另一个基于方法名称的装配器是MethodExclusionMBeanInfoAssembler。 这个MBean信息装配器是MethodNameBaseMBeanInfoAssembler的反操作。 它不是指定哪些方法需要暴露为MBean的托管操作， MethodExclusionMBeanInfoAssembler指定了不需要暴露为MBean托管操作的方法名称列表。

43. 使用接口定义MBean的操作和属性 ：Spring的InterfaceBasedMBeanInfoAssembler是另一种MBean信息装配器， 可以让我们通过使用接口来选择bean的哪些方法需要暴露为MBean的托管操作。 InterfaceBasedMBeanInfoAssembler与基于方法名称的装配器很相似， 只不过不再通过罗列方法名称来确定暴
露哪些方法， 而是通过列出接口来声明哪些方法需要暴露。

44. 使用注解驱动的MBean：Spring还提供了另一种装配器——Metadata-MBeanInfoAssembler， 这种装配器可以使用注解标识哪些bean的方法需要暴露为MBean的托管操作和属性。

45. 在Spring家族中， Spring Boot是令人兴奋（也许我敢说它是改变游戏规则的） 的新项目。 它提供了四个主要的特性， 能够改变开发Spring应用程序的方式： 
- Spring Boot Starter： 它将常用的依赖分组进行了整合， 将其合并到一个依赖中， 这样就可以一次性添加到项目的Maven或Gradle构建中；
- 自动配置： Spring Boot的自动配置特性利用了Spring 4对条件化配置的支持， 合理地推测应用所需的bean并自动化配置它们；
- 命令行接口（Command-line interface， CLI） ： Spring Boot的CLI发挥了Groovy编程语言的优势， 并结合自动配置进一步简化Spring应用的开发；
- Actuator： 它为Spring Boot应用添加了一定的管理特性。