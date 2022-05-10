---
title: kkFileView实现文件优雅预览水印
date: 2022-04-27 09:22:31 +0800
tags: 开源项目
---

## 起因
项目中包含大量office文件以及pdf需要实现在线预览，前端预览只能实现pdf预览且效果不佳

## 项目介绍
springboot搭建的文件预览服务，需要作为第三方服务器启动，原理是将文件通过本地安装的openoffice或者libreoffice转换成pdf，进而可以转换成图片，输出到前端模板中。且可以添加自定义水印，配置文件中也有多种选项可以配置
```
#######################################不可动态配置，需要重启生效#######################################
server.port = ${KK_SERVER_PORT:8012}
server.servlet.context-path= ${KK_CONTEXT_PATH:/}
server.servlet.encoding.charset = utf-8
#文件上传限制
spring.servlet.multipart.max-file-size=500MB
spring.servlet.multipart.max-request-size=500MB
## Freemarker 配置
spring.freemarker.template-loader-path = classpath:/web/
spring.freemarker.cache = false
spring.freemarker.charset = UTF-8
spring.freemarker.check-template-location = true
spring.freemarker.content-type = text/html
spring.freemarker.expose-request-attributes = true
spring.freemarker.expose-session-attributes = true
spring.freemarker.request-context-attribute = request
spring.freemarker.suffix = .ftl

# office-plugin
## office转换服务的进程数，默认开启两个进程
office.plugin.server.ports = 2001,2002
## office 转换服务 task 超时时间，默认五分钟
office.plugin.task.timeout = 5m

#预览生成资源路径（默认为打包根路径下的file目录下）
#file.dir = D:\\kkFileview\\
file.dir = ${KK_FILE_DIR:default}

#允许预览的本地文件夹 默认不允许任何本地文件被预览
#file.dir = D:\\kkFileview\\
local.preview.dir = ${KK_LOCAL_PREVIEW_DIR:default}


#openoffice home路径
#office.home = C:\\Program Files (x86)\\OpenOffice 4
office.home = ${KK_OFFICE_HOME:default}

#缓存实现类型，不配默认为内嵌RocksDB(type = default)实现，可配置为redis(type = redis)实现（需要配置spring.redisson.address等参数）和 JDK 内置对象实现（type = jdk）,
cache.type =  ${KK_CACHE_TYPE:jdk}
#redis连接，只有当cache.type = redis时才有用
spring.redisson.address = ${KK_SPRING_REDISSON_ADDRESS:127.0.0.1:6379}
spring.redisson.password = ${KK_SPRING_REDISSON_PASSWORD:}
#缓存是否自动清理 true 为开启，注释掉或其他值都为关闭
cache.clean.enabled = ${KK_CACHE_CLEAN_ENABLED:true}
#缓存自动清理时间，cache.clean.enabled = true时才有用，cron表达式，基于Quartz cron
cache.clean.cron = ${KK_CACHE_CLEAN_CRON:0 0 3 * * ?}

#######################################可在运行时动态配置#######################################
#提供预览服务的地址，默认从请求url读，如果使用nginx等反向代理，需要手动设置
#base.url = https://file.keking.cn
base.url = ${KK_BASE_URL:default}

#信任站点，多个用','隔开，设置了之后，会限制只能预览来自信任站点列表的文件，默认不限制
#trust.host = file.keking.cn,kkfileview.keking.cn
trust.host = ${KK_TRUST_HOST:default}

#是否启用缓存
cache.enabled = ${KK_CACHE_ENABLED:true}

#文本类型，默认如下，可自定义添加
simText = ${KK_SIMTEXT:txt,html,htm,asp,jsp,xml,json,properties,md,gitignore,log,java,py,c,cpp,sql,sh,bat,m,bas,prg,cmd}
#多媒体类型，默认如下，可自定义添加
media = ${KK_MEDIA:mp3,wav,mp4,flv}
#是否开启多媒体类型转视频格式转换,目前可转换视频格式有：avi,mov,wmv,3gp,rm
#请谨慎开启此功能，建议异步调用添加到处理队列，并且增加任务队列处理线程，防止视频转换占用完线程资源，转换比较耗费时间,并且控制了只能串行处理转换任务
media.convert.disable = ${KK_MEDIA_CONVERT_DISABLE:false}
#支持转换的视频类型
convertMedias = ${KK_CONVERTMEDIAS:avi,mov,wmv,mkv,3gp,rm}		
#office类型文档(word ppt)样式，默认为图片(image)，可配置为pdf（预览时也有按钮切换）
office.preview.type = ${KK_OFFICE_PREVIEW_TYPE:image}
#是否关闭office预览切换开关，默认为false，可配置为true关闭
office.preview.switch.disabled = ${KK_OFFICE_PREVIEW_SWITCH_DISABLED:false}

#是否禁止下载转换生成的pdf文件
pdf.download.disable = ${KK_PDF_DOWNLOAD_DISABLE:false}
#是否禁用首页文件上传
file.upload.disable = ${KK_FILE_UPLOAD_ENABLED:false}

#预览源为FTP时 FTP用户名，可在ftp url后面加参数ftp.username=ftpuser指定，ä¸指定默认用配置的
ftp.username = ${KK_FTP_USERNAME:ftpuser}
#预览源为FTP时 FTP密码，可在ftp url后面加参数ftp.password=123456指定，不指定默认用配置的
ftp.password = ${KK_FTP_PASSWORD:123456}
#预览源为FTP时, FTP连接默认ControlEncoding(根据FTP服务器操作系统选择，Linux一般为UTF-8，Windows一般为GBK)，可在ftp url后面加参数ftp.control.encoding=UTF-8指定，不指定默认用配置的
ftp.control.encoding = ${KK_FTP_CONTROL_ENCODING:UTF-8}

#水印内容
#例：watermark.txt = ${WATERMARK_TXT:凯京科技内部文件，严禁外泄}
#如需取消水印，内容设置为空即可，例：watermark.txt = ${WATERMARK_TXT:}
watermark.txt = ${WATERMARK_TXT:}
#水印x轴间隔
watermark.x.space = ${WATERMARK_X_SPACE:10}
#水印y轴间隔
watermark.y.space = ${WATERMARK_Y_SPACE:10}
#水印字体
watermark.font = ${WATERMARK_FONT:微软雅黑}
#水印字体大小
watermark.fontsize = ${WATERMARK_FONTSIZE:18px}
#水印字体颜色
watermark.color = ${WATERMARK_COLOR:black}
#水印透明度，要求设置在大于等于0.005，小于1
watermark.alpha = ${WATERMARK_ALPHA:0.2}
#水印宽度
watermark.width = ${WATERMARK_WIDTH:180}
#水印高度
watermark.height = ${WATERMARK_HEIGHT:80}
#水印倾斜度数，要求设置在大于等于0，小于90
watermark.angle = ${WATERMARK_ANGLE:10}
```

## 项目整合
1. 系统内下载采用的是http/https下载流url预览
很多系统内不是直接暴露文件下载地址，而是请求通过id、code等参数到通过统一的接口，后端通过id或code等参数定位文件，再通过OutputStream输出下载，此时下载url是不带文件后缀名的，预览时需要拿到文件名，传一个参数fullfilename=xxx.xxx来指定文件名，示例如下
```
var originUrl = 'http://127.0.0.1:8080/filedownload?fileId=1'; //要预览文件的访问地址
var previewUrl = originUrl + '&fullfilename=test.txt'
window.open('http://127.0.0.1:8012/onlinePreview?url='+encodeURIComponent(Base64.encode(previewUrl)));
```

2. 项目还支持自定义传入水印，在预览url后面加上参数&watermarkTxt即可
```
var url = 'http://127.0.0.1:8080/file/test.txt'; //要预览文件的访问地址
window.open('http://127.0.0.1:8012/onlinePreview?url=' + encodeURIComponent(url) + '&watermarkTxt=' + encodeURIComponent('动态水印'));
```

由于需要自定义传入水印，所以对原来的水印处理做了修改,代码拦截器位于`AttributeSetFilter`,其中拦截了watermarkTxt参数，并将其注入到当前请求request对象中。我们修改了参数值并且对齐进行base64转码操作，当然防君子不防小人，参数放在url请求中本来就是不安全的，只是做了些许转码能降低别人去除水印的难度。
```
String watermarkTxt = new String(Base64.decodeBase64(request.getParameter("wmc")), StandardCharsets.UTF_8);
request.setAttribute("watermarkTxt", watermarkTxt != null ? watermarkTxt : WatermarkConfigConstants.getWatermarkTxt());
```
## 项目打包启动
1.docker安装

```
//maven编译，
mvn clean package -DskipTests

//进行打包操作 然后生成image
docker build -t keking/kkfileview:v4.0.0 .

//docker启动 并且可以传入配置文件中对应变量
docker run --restart=always -d \
 -e KK_BASE_URL="https://预览地址/kkfile" \
 -e KK_CONTEXT_PATH="/kkfile" \
 -it -p 8012:8012  --name kkfile keking/kkfileview:v4.0.0
 
 //KK_BASE_URL作为网站预览域名，
 //KK_CONTEXT_PATH 对应nginx中配置的location地址，如作为单独端口映射无二级域名则可以不填
 server {
        listen 20303;
        server_name localhost;
         location /kkfile {
                        proxy_pass http://localhost:8012;
                }
}

```
#### docker拓展：
每次打包时间较长，可以采用映射文件夹，替换jar包方式来进行更新。通过查看Dockerfile可以确定容器内`/opt/kkFileView-4.1.0-SNAPSHOT`,存放jar包和配置文件。
    
```
ENTRYPOINT ["java","-Dfile.encoding=UTF-8","-Dspring.config.location=/opt/kkFileView-4.1.0-SNAPSHOT/config/application.properties","-jar","/opt/kkFileView-4.1.0-SNAPSHOT/bin/kkFileView-4.1.0-SNAPSHOT.jar"]
```

在宿主机内新建文件夹并在内部创建bin和config文件夹，并在创建容器的时候进行映射，之后每次编译后将jar包放入bin，配置文件放入config即可，可随时方便的修改配置文件了，同时下载和转换后的文件存在于同目录的file文件夹下。
```
docker run --restart=always -d \
 -e KK_BASE_URL="https://预览地址/kkfile" \
 -e KK_CONTEXT_PATH="/kkfile" \
 -v 宿主机文件夹:/opt/kkFileView-4.1.0-SNAPSHOT \
 -it -p 8012:8012  --name kkfile keking/kkfileview:v4.0.0
```  

2.源码编译上传，修改配置文件，运行bin中的start.sh启动，会自动安装libreoffice以及启动项目。（目前还未尝试）

## 效果展示

![](/media/16503324978050/16510243592444.jpg)


## 后续问题
1. 虽然禁用下载按钮了，但是页面可以ctrl+s下载问题，定位到是pdf.js的问题，修改view.js源码,注释掉对应代码。
```
 if (cmd === 1 || cmd === 8) {
    switch (evt.keyCode) {
      case 83:
        // eventBus.dispatch("download", {
        //   source: window
        // });
        // handled = true;
        break;
```