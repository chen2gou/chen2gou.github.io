---
title: 【微服务专题】springboot springsecurity oauth2自定义异常
date: 2022-04-11 10:28:34 +0800
tags: springboot springsecurity oauth2
---

## oauth2认证流程
调用认证中心的`/oauth/token`，header中带上basic认证的clientid和secret;用户名密码模式对应的`DaoAuthenticationProvider`，定义了`UserDetailsService`来获取用户信息和密码模式`passwordEncoder`;获取信息失败或者认证错误用`DefaultWebResponseExceptionTranslator`来进行异常处理，实现的接口`WebResponseExceptionTranslator`，`translate`方法用于异常处理。要实现自定义异常，我们继承后重写方法即可。

![认证流程](/img/2022/security%E8%AE%A4%E8%AF%81%E6%B5%81%E7%A8%8B.png)

## 自定义异常输出
1.重写translate来自定义异常

```
/**
 * @author TAO
 * @description: SpringSecurity OAuth2 异常翻译器，统一捕获授权时异常信息处理
 * @date 2021/10/1 16:14
 */
@Slf4j
@Component
public class Auth2ResponseExceptionTranslator implements WebResponseExceptionTranslator {

    private ThrowableAnalyzer throwableAnalyzer = new DefaultThrowableAnalyzer();

    @Override
    public ResponseEntity translate(Exception e) throws Exception {

        Throwable[] causeChain = throwableAnalyzer.determineCauseChain(e);
        Exception ase = null;// 异常栈获取 OAuth2Exception 异常

        ase= (AuthenticationException) throwableAnalyzer.getFirstThrowableOfType(AuthenticationException.class, causeChain);
        if (ase != null) {
            return handleOAuth2Exception(new ExtendOAuth2Exception(e.getMessage(), e));
        }

        ase = (UnsupportedGrantTypeException) throwableAnalyzer
                .getFirstThrowableOfType(UnsupportedGrantTypeException.class, causeChain);
        if (ase != null) {
            return handleOAuth2Exception(new ExtendOAuth2Exception(e.getMessage(), e));
        }

        ase = (AccessDeniedException) throwableAnalyzer.getFirstThrowableOfType(AccessDeniedException.class,
                causeChain);
        if (ase != null) {
            return handleOAuth2Exception(new ExtendOAuth2Exception(ase.getMessage(), ase));
        }

        ase = (HttpRequestMethodNotSupportedException) throwableAnalyzer
                .getFirstThrowableOfType(HttpRequestMethodNotSupportedException.class, causeChain);
        if (ase != null) {
            return handleOAuth2Exception(new ExtendOAuth2Exception(ase.getMessage(), ase));
        }

        ase = (OAuth2Exception) throwableAnalyzer.getFirstThrowableOfType(OAuth2Exception.class, causeChain);

        if (ase != null) {
            return handleOAuth2Exception((OAuth2Exception) ase);
        }

        // 不包含上述异常则服务器内部错误
        return handleOAuth2Exception(new ExtendOAuth2Exception(HttpStatus.INTERNAL_SERVER_ERROR.getReasonPhrase(), e));

    }

    private ResponseEntity<OAuth2Exception> handleOAuth2Exception(OAuth2Exception e) {

        int status = e.getHttpErrorCode();
        HttpHeaders headers = new HttpHeaders();
        headers.set("Cache-Control", "no-store");
        headers.set("Pragma", "no-cache");
        if (status == HttpStatus.UNAUTHORIZED.value() || (e instanceof InsufficientScopeException)) {
            headers.set("WWW-Authenticate", String.format("%s %s", OAuth2AccessToken.BEARER_TYPE, e.getSummary()));
        }
        return new ResponseEntity<>(new ExtendOAuth2Exception(e.getMessage(),e), headers, HttpStatus.valueOf(status));
    }


}

```
2.异常继承类以及序列化类

扩展OAuth2Exception
```
/**
* @description: 扩展OAuth2Exception
* @author TAO
* @date 2021/9/29 22:14
*/
@JsonSerialize(using = ExtendOAuth2ExceptionSerializer.class)
public class ExtendOAuth2Exception extends OAuth2Exception {

    @Getter
    private String dataMsg;

    public ExtendOAuth2Exception(String msg) {
        super(msg);
    }

    public ExtendOAuth2Exception(String msg, Throwable t) {
        super(msg, t);
    }

    public ExtendOAuth2Exception(String msg, String dataMsg) {
        super(msg);
        this.dataMsg = dataMsg;

    }

}


```
ExtendOAuth2Exception返回格式序列化
```
/**
* @description: ExtendOAuth2Exception返回格式序列化
* @author TAO
* @date 2021/9/29 22:42
*/
public class ExtendOAuth2ExceptionSerializer extends StdSerializer<ExtendOAuth2Exception> {

    public ExtendOAuth2ExceptionSerializer() {
        super(ExtendOAuth2Exception.class);
    }

    @Override
    public void serialize(ExtendOAuth2Exception e, JsonGenerator gen, SerializerProvider serializerProvider) throws IOException {
        gen.writeStartObject();
        gen.writeObjectField("code", String.valueOf(e.getHttpErrorCode()));
        gen.writeStringField("msg", e.getMessage());
        gen.writeStringField("data", e.getDataMsg());
        gen.writeEndObject();

    }
}
```

3.配置自定义异常解析器

```
    /**
     * 访问端点配置。tokenStore、tokenEnhancer服务
     */
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        TokenEnhancerChain tokenEnhancerChain = new TokenEnhancerChain();
        List<TokenEnhancer> tokenEnhancers = new ArrayList<>();
        tokenEnhancers.add(tokenEnhancer());
        tokenEnhancerChain.setTokenEnhancers(tokenEnhancers);

        endpoints.allowedTokenEndpointRequestMethods(HttpMethod.GET, HttpMethod.POST) //支持请求
                .tokenEnhancer(tokenEnhancerChain)//token增强
                .authenticationManager(authenticationManager)//密码模式
                .tokenStore(tokenStore())//token存入redis
                .exceptionTranslator(new Auth2ResponseExceptionTranslator()) // 设置自定义的异常解析器
                .userDetailsService(userDetailsService);
    }
```
至此，security oauth2认证已经能返回指定格式的返回体
```
{
    "code": "400",
    "msg": "Unsupported grant type: password1",
    "data": null
}
```

## UsernameNotFoundException无法抛出的问题

经过对源码的分析 ，`hideUserNotFoundExceptions`决定了是否转换异常类型。
```
AbstractUserDetailsAuthenticationProvider.class

 try {
      user = retrieveUser(username,
              (UsernamePasswordAuthenticationToken) authentication);
  }
  catch (UsernameNotFoundException notFound) {
      logger.debug("User '" + username + "' not found");

      if (hideUserNotFoundExceptions) {
          throw new BadCredentialsException(messages.getMessage(
                  "AbstractUserDetailsAuthenticationProvider.badCredentials",
                  "Bad credentials"));
      }
      else {
          throw notFound;
      }
  }

```


解决方法就是把实现类中`hideUserNotFoundExceptions`修改值为false，并且替换掉默认的。
```
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    private UserDetailsService userDetailsService;


    /**
     * 用户验证
     */
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
       //替换默认DaoAuthenticationProvider
        auth.authenticationProvider(authenticationProvider());
    }

    /**
    *  用于抛出UsernameNotFoundException
    */
    @Bean
    public DaoAuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setHideUserNotFoundExceptions(false);
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }

```

如此一来，我们在loadUserByUsername中抛出的异常可以正常反馈
```
{
    "code": "400",
    "msg": "用户不存在",
    "data": null
}
```