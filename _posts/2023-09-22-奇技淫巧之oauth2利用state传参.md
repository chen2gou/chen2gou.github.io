---
title: 奇技淫巧之oauth2利用state传参
date: 2023-08-22 16:12:25 +0800
tags: oauth2
---
## 问题
公司多个子系统调用同一套oauth2认证，但是系统之间token是自定义的，现在需要A跳转到B指定页面，单点认证之后跳转的地址是固定的

## 解决办法
利用oauth2中的state参数，oauth官网上有这样一段 [关于state描述的内容](https://www.oauth.com/oauth2-servers/redirect-uris/redirect-uri-registration/#:~:text=If%2520a%2520client%2520wishes%2520to%2520include%2520request%252Dspecific%2520data%2520in%2520the%2520redirect%2520URL%252C%2520it%2520can%2520instead%2520use%2520the%2520%25E2%2580%259Cstate%25E2%2580%259D%2520parameter%2520to%2520store%2520data%2520that%2520will%2520be%2520included%2520after%2520the%2520user%2520is%2520redirected.%2520It%2520can%2520either%2520encode%2520the%2520data%2520in%2520the%2520state%2520parameter%2520itself%252C%2520or%2520use)
```
如果客户端希望在重定向 URL 中包含特定于请求的数据，则可以使用“state”参数来存储用户重定向后将包含的数据。 
它可以对状态参数本身中的数据进行编码，也可以使用状态参数作为会话 ID 将状态存储在服务器上。
```
## 步骤1
一般state用于状态认证，是用于防止跨站请求伪造攻击，通常，state 参数会随着授权请求一起发送到授权服务器，授权服务器会在回调时返回该参数，客户端需要验证该参数和发送时的是
否一致，以确保请求不是来自非法的攻击者。但是我们也可以利用state传递参数，我们在调用oauth2认证链接的时候将跳转地址传入`注意需要将链接进行url编码，例如:http%3A%2F%2Fwww.baidu.com`

```java
public String login(@RequestParam(required = false) String backUrl) {
        // 传递参数response_type、client_id、state、redirect_uri
        String url = baseUrl + authorizeUrl + "?response_type=code&" + "client_id=" + clientId + "&redirect_uri=" + callbackUrl;
        //state记录跳转url
        if (StrUtil.isNotBlank(backUrl)) {
            url = url + "&state=" + backUrl;
        }
        url = url + "&scope=ALL";
        return "redirect:" + url;
    }
```
## 步骤2
上一步我们传递了state参数，那么在回调的时候就会接收到state参数

```java
public String callback(@RequestParam(required = false) String code, @RequestParam(required = false) String state) {
        //……此处省略业务处理 换取token 获取用户信息
        //state不为空取出重定向url，携带token进行跳转 否则跳转默认首页
        if (StrUtil.isNotBlank(state)) {
            callbackFeUrl = state;
        }
        return "redirect:" + callbackFeUrl + "?token=" + r.get("token");
    }

```

## 步骤3
测试一下，B系统改造为以上代码，A系统前端页面挂载B系统跳转认证接口 `http://xxxx/sso/dologin?backUrl=http%3A%2F%2Fwww.baidu.com`,实现跳转到百度，此处百度替换为B系统指定页面。打完收工