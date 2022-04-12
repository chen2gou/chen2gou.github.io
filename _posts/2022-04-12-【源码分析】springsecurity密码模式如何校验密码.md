---
title: 【源码分析】springsecurity密码模式校验密码
date: 2022-04-12 09:39:00 +0800
tags: springsecurity oauth2
---

## oauth2认证流程
**TokenGranter**给用户分发OAuth2AccessToken,根据grantType(password,authorization-code)和TokenRequest（requestParameters,clientId,grantType）授予人OAuth2AccessToken令牌。


实现AbstractTokenGranter的类有5种。其中如果用password的方式进行验证，那么TokenGranter类型是**ResourceOwnerPasswordTokenGranter**，该类中重写了getOAuth2Authentication方法，里面调用了authenticationManager.authenticate()方法。

 ```
userAuth = this.authenticationManager.authenticate(userAuth);
 ```

用户可自行定义granter类继承AbstractTokenGranter，重写getOAuth2Authentication()方法，并将该granter类添加至CompositeTokenGranter中。五种实现类如下图：
![asd](/img/2022/granter实现.png)



## 密码模式源码分析

验证码用户名密码调用 **AbstractUserDetailsAuthenticationProvider**
中的authenticate方法，至于为什么调用这个服务，可以观察到其缺省实现 ProviderManager中一行代码`if (provider.supports(toTest))`
即表示如果服务支持 toTest 这个类型：` Class<? extends Authentication> toTest = authentication.getClass();`
那么它就会调用这个服务的认证方法：`result = provider.authenticate(authentication);`
`supports` 方法是接口`AuthenticationProvider`中的一个方法。

我们打开一个实现类，以**AbstractUserDetailsAuthenticationProvider**抽象类为例
这个方法的实现表示这个服务支持 `UsernamePasswordAuthenticationToken` 这个类型的Token
```
return UsernamePasswordAuthenticationToken.class.isAssignableFrom(authentication);
```
当传入的Token类型是`UsernamePasswordAuthenticationToken`
并且在服务列表中循环到这个服务的具体实现服务时，就会触发该实现的认证方法

## 密码校验
**AbstractUserDetailsAuthenticationProvider**中authenticate方法
```
    this.preAuthenticationChecks.check(user);
    #用于校验密码
    this.additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken)authentication);
```
调用了DaoAuthenticationProvider中的验证密码方法
```
    protected void additionalAuthenticationChecks(UserDetails userDetails, UsernamePasswordAuthenticationToken authentication) throws AuthenticationException {
        if (authentication.getCredentials() == null) {
            this.logger.debug("Authentication failed: no credentials provided");
            throw new BadCredentialsException(this.messages.getMessage("AbstractUserDetailsAuthenticationProvider.badCredentials", "Bad credentials"));
        } else {
            String presentedPassword = authentication.getCredentials().toString();
            if (!this.passwordEncoder.matches(presentedPassword, userDetails.getPassword())) {
                this.logger.debug("Authentication failed: password does not match stored value");
                throw new BadCredentialsException(this.messages.getMessage("AbstractUserDetailsAuthenticationProvider.badCredentials", "Bad credentials"));
            }
        }
    }
```

参考链接：[OAuth2 源码分析(一.核心类）](https://blog.csdn.net/qq_30905661/article/details/81112305)

[SpringCloud - Oauth2增加短信验证码验证登录
](https://blog.csdn.net/qq_40096897/article/details/122668061)