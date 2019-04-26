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

18. 














