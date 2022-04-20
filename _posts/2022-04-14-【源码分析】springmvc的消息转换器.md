---
title: 【源码分析】springmvc的消息转换器
date: 2022-04-20 09:40:19 +0800
tags: java SpringMVC
---

## 疑问
为什么springboot的Controller带上`@RestController`注解就可以直接返回json呢

```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller
@ResponseBody
public @interface RestController {
    @AliasFor(
        annotation = Controller.class
    )
    String value() default "";
}
```
可以看到源码里带上了@ResonseBody注解，这就相当于给Controller里面每个方法增加了这个注解，该注解用于将Controller的方法返回的对象，通过适当的HttpMessageConverter转换为指定格式后，写入到Response对象的body数据区。

## 消息转换器 HttpMessageConverter

MVC会添加几个默认的转换器，HttpMessageConverters维护了一个HttpMessageConverter集合。WebMvcConfigurationSupport类的方法addDefaultHttpMessageConverters中可以看到。而要动用这些消息转换器，需要在特定的位置加上`@RequestBody`和`@ResponseBody`
![](/media/16503324978050/16504199075600.jpg)

SpringMVC会根据传入的Content-Type，对应的值就可以叫做是Media Type来确定消息转换器，如果没有合适的转换器，读则抛出HttpMediaTypeNotSupportedException，浏览器会收到一个415 Unsupported Media Type状态码；写则抛出HttpMediaTypeNotAcceptableException异常，浏览器会收到一个406 Not Acceptable状态码。
举个例子
```
    @PostMapping("/testRequestBody")
    @ResponseBody
    public Object testRequestBody(@RequestBody User user) {
        return user;
    }

```

在SpringMVC进入testRequestBody方法前，会根据@RequestBody注解选择适当的HttpMessageConverter实现类来将请求参数解析到User对象中，具体来说是使用了MappingJackson2HttpMessageConverter类，它的canRead()方法返回true，然后它的read()方法会从请求中读出请求参数，绑定到testRequestBody()方法的User对象中。

当SpringMVC执行testRequestBody方法后，由于返回值标识了@ResponseBody，SpringMVC将使用MappingJackson2HttpMessageConverter的write()方法，将结果作为json值写入响应报文，当然，此时canWrite()方法返回true。
## 运作流程
![](/media/16503324978050/16504173198694.jpg)


对于消息转换器的调用，都是在`RequestResponseBodyMethodProcessor`类中完成的，对于返回值的操作handleReturnValue调用了AbstractMessageConverterMethodProcessor中的`writeWithMessageConverters`方法，确定selectedMediaType后遍历消息转换器，根据canWrite来确定消息转换器，并对返回值进行处理；对于请求值操作调用AbstractMessageConverterMethodArgumentResolver的`readWithMessageConverters`方法，处理方式一致。

参考文章：

[springMVC的消息转换器（Message Converter）](https://www.jianshu.com/p/2f633cb817f5)

[Spring MVC系列（9）-HttpMessageConverter报文转换流程源码解析_云烟成雨TD的博客-CSDN博客](https://blog.csdn.net/qq_43437874/article/details/120262966)

[Spring MVC系列（10）-使用fastjson自定义消息转换器_云烟成雨TD的博客-CSDN博客_fastjson 转换器](https://blog.csdn.net/qq_43437874/article/details/120271673)

[SpringMVC源码剖析（五)-消息转换器HttpMessageConverter - 相见欢的个人空间 - OSCHINA - 中文开源技术交流社区](https://my.oschina.net/lichhao/blog/172562)