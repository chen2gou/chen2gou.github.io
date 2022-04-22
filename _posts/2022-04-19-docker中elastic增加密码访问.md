---
title: docker中elastic增加密码访问
date: 2022-04-19 09:40:19 +0800
tags: docker elasticsearch
---

## 配置文件修改
进入容器内部
```
sudo docker exec -it elastic /bin/bash
```
vi打开config 下 elasticsearch.yml
增加安全配置
```
# 配置X-Pack
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-headers: Authorization
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
```
重启docker中elastic
`docker restart elastic`


## 修改密码

`bin/elasticsearch-setup-passwords interactive`

因为需要设置 elastic，apm_system，kibana，kibana_system，logstash_system，beats_system，remote_monitoring_user 这些用户的密码

## 验证
`curl localhost:9200`
出现如下信息
```
{"error":{"root_cause":[{"type":"security_exception","reason":"missing authentication credentials for REST request [/]","header":{"WWW-Authenticate":"Basic realm=\"security\" charset=\"UTF-8\""}}],"type":"security_exception","reason":"missing authentication credentials for REST request [/]","header":{"WWW-Authenticate":"Basic realm=\"security\" charset=\"UTF-8\""}},"status":401}
```
带上用户密码信息访问则返回正常信息
`curl localhost:9200 -u elastic:{password}`

## RestHighLevelClient配置修改

后端生产环境增加CredentialsProvider用于配置账号密码访问
```
 HttpHost[] hh = clusterNodes.stream().map(x -> new HttpHost(x.split(":")[0], Integer.parseInt(x.split(":")[1]), "http")).toArray(HttpHost[]::new);
        RestClientBuilder builder = RestClient.builder(hh);
        //增加账号密码配置
        if (StringUtils.isNotBlank(userName) && StringUtils.isNotBlank(password)) {
            CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
            credentialsProvider.setCredentials(AuthScope.ANY, new UsernamePasswordCredentials(userName, password));
            builder.setHttpClientConfigCallback(f -> f.setDefaultCredentialsProvider(credentialsProvider));
        }
        RestHighLevelClient client = new RestHighLevelClient(builder);
        return client;
```

前端访问9200端口需要在header中增加basic认证拼接用户名密码信息
