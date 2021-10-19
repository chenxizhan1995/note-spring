# RestTemplate
2021-10-15

[掌握 Spring 之 RestTemplate - 掘金](https://juejin.cn/post/6844903842065154061)
[RestTemplate 最详解 - 程序员自由之路 - 博客园](https://www.cnblogs.com/54chensongxia/p/11414923.html)
[github 代码：一起学 Spring 之 RestTemplate 使用指南 的 演示 Demo](https://github.com/developer-wenren/resttemplate/tree/master)
[RestTemplate (Spring Framework 5.3.11 API)](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html)

"Spring 官网教程对 RestTemplate的介绍"
[Integration](https://docs.spring.io/spring-framework/docs/5.1.6.RELEASE/spring-framework-reference/integration.html#rest-client-access)
## 基本概念
HTTP 客户端工具类。同步模式。自 Spring 3.0 引入，自Spring 5.0 废弃，5.0引入了替代物 WebClient，是下一代产品，异步。

该工具类提供的方法大致分为三组：
- getForObject,getForEntity... 分为一组，这类方法是常规的 Rest API（GET、POST、DELETE 等）方法调用；
- exchange：                  接收一个 RequestEntity 参数，可以自己设置 HTTP method，URL，headers 和 body，返回 ResponseEntity；
- execute：                   通过 callback 接口，可以对请求和返回做更加全面的自定义控制
一般情况下，我们使用第一组和第二组方法就够了。

### 类层次
```
java.lang.Object
  org.springframework.http.client.support.HttpAccessor
      org.springframework.http.client.support.InterceptingHttpAccessor
          org.springframework.web.client.RestTemplate
```
实现的接口：RestOperations。
这个接口方法很多，基础 Rest 操作方法。
> Interface specifying a basic set of RESTful operations. Implemented by RestTemplate. Not often used directly, but a useful option to enhance testability, as it can easily be mocked or stubbed.
### 方法1

据说 rest 风格的接口使用HTTP动词表达操作，那么 GET, POST, PUT, DELETE 是要支持的哈。
对每个 HTTP 方法重载了若干快捷操作方法。

对于有返回值的方法 GET、POST 都提供 xyzForObject() 和 xyzForEntity() 两种，
对于无返回值的方法 PUT、DELETE 都提供 xxx() 方法。
参数，对于无请求体的方法 GET，DELETE，参数为 url, Class, uriVariables。
对于有请求体的方法 POST、PUT 参数为 url，requet，Class, urlVariables。
url 为URI对象的时候，没有urivariables，大概是都包含在URI对象中了。

```

先是 get 系列
<T> ResponseEntity<T> 	getForEntity(String url, Class<T> responseType, Map<String,?> uriVariables)
<T> ResponseEntity<T> 	getForEntity(String url, Class<T> responseType, Object... uriVariables)
<T> ResponseEntity<T> 	getForEntity(URI url, Class<T> responseType)
<T> T 	                getForObject(String url, Class<T> responseType, Map<String,?> uriVariables)
<T> T 	                getForObject(String url, Class<T> responseType, Object... uriVariables)
<T> T 	                getForObject(URI url, Class<T> responseType)

接着是 POST 系列
<T> ResponseEntity<T> 	postForEntity(String url, Object request, Class<T> responseType, Map<String,?> uriVariables)
<T> ResponseEntity<T> 	postForEntity(String url, Object request, Class<T> responseType, Object... uriVariables)
<T> ResponseEntity<T> 	postForEntity(URI url, Object request, Class<T> responseType)
<T> T 	postForObject(String url, Object request, Class<T> responseType, Map<String,?> uriVariables)
<T> T 	postForObject(String url, Object request, Class<T> responseType, Object... uriVariables)
<T> T 	postForObject(URI url, Object request, Class<T> responseType)

然后是PUT
void 	put(String url, Object request, Map<String,?> uriVariables)
void 	put(String url, Object request, Object... uriVariables)
void 	put(URI url, Object request)

还有 delete
void 	delete(String url, Map<String,?> uriVariables)
void 	delete(String url, Object... uriVariables)
void 	delete(URI url)

```
PS：还看到了 HEADERS，OPTIONS, PATCH 方法。还有三个重载的 postForLocation 方法。后面再看。

这些方法的重载版本都很有规律，形参和方法名称。

ResponseEntity 则获取的内容多一些，包括状态码和媒体类型。
```java
ResponseEntity<String> entity = template.getForEntity("https://example.com", String.class);
String body = entity.getBody();
MediaType contentType = entity.getHeaders().getContentType();
HttpStatus statusCode = entity.getStatusCode();
```
### 方法2：exchange()
exchange 是比上述方法更底层的方法，它接收 RequestEntity(包含HTTP方法、首部、URL、请求体)作为参数，返回
ResponseEntiy 作为结果。

```
<T> ResponseEntity<T> 	exchange(RequestEntity<?> entity, Class<T> responseType)

<T> ResponseEntity<T> 	exchange(String url, HttpMethod method, HttpEntity<?> requestEntity, Class<T> responseType, Map<String,?> uriVariables)
<T> ResponseEntity<T> 	exchange(String url, HttpMethod method, HttpEntity<?> requestEntity, Class<T> responseType, Object... uriVariables)

<T> ResponseEntity<T> 	exchange(URI url, HttpMethod method, HttpEntity<?> requestEntity, Class<T> responseType)


<T> ResponseEntity<T> 	exchange(RequestEntity<?> entity, ParameterizedTypeReference<T> responseType)

<T> ResponseEntity<T> 	exchange(String url, HttpMethod method, HttpEntity<?> requestEntity, ParameterizedTypeReference<T> responseType, Map<String,?> uriVariables)
<T> ResponseEntity<T> 	exchange(String url, HttpMethod method, HttpEntity<?> requestEntity, ParameterizedTypeReference<T> responseType, Object... uriVariables)

<T> ResponseEntity<T> 	exchange(URI url, HttpMethod method, HttpEntity<?> requestEntity, ParameterizedTypeReference<T> responseType)
```
### 方法3：execute()
这个是最底层的方法，通过回调接口，可以完全控制整个请求/响应的方方面面。
```
<T> T 	execute(String url, HttpMethod method, RequestCallback requestCallback, ResponseExtractor<T> responseExtractor, Map<String,?> uriVariables)

<T> T 	execute(String url, HttpMethod method, RequestCallback requestCallback, ResponseExtractor<T> responseExtractor, Object... uriVariables)

<T> T 	execute(URI url, HttpMethod method, RequestCallback requestCallback, ResponseExtractor<T> responseExtractor)
```
### 线程安全性
API 文档并没有说是否是线程安全的。网上资料如下：

> 从概念上讲，它是非常相似的JdbcTemplate，JmsTemplate和Spring框架和其他投资项目中发现的各种其他模板。举例来说，这意味着RestTemplate一旦构建后，线程安全

> RestTemplate该类的对象不会更改其任何状态信息来处理HTTP：该类是Strategy设计模式的一个实例，而不是像一个连接对象。没有状态信息，如果它们共享一个RestTemplate对象，就不可能有不同的线程破坏或加速状态信息。这就是为什么线程可以共享这些对象的原因。

> 如果检查源代码，RestTemplate您将发现在构造对象之后，它不使用synchronized方法或volatile字段来提供线程安全。因此，它是不是安全的修改RestTemplate施工后的对象。特别是，添加消息转换器是不安全的。

ps：总结就是：新建对象，投入使用之后只要不修改对象，并发发送请求是没有问题的。

[关于java：RestTemplate线程安全吗？ | 码农家园](https://www.codenong.com/22989500/)
## 常见用例
### 简单get请求
### 简单post请求
### get 发送表单数据
### post 发送表单数据
### post 发送json数据
- 使用 postForObject/postForEntity 方法，可以应对常见需求，但控制力略有不足
  - 以实体类作为参数       --- 可以
  - 以 Map 对象作为参数    --- 可以
  - 以 json 字符串作为参数 --- 不行无法设置媒体类型
- 使用 exchange 方法，可以覆盖绝大多数需求
- 使用 execute ……

### post 上传文件

