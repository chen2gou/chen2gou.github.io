---
title: freemarker+itext实现超级简历pdf
date: 2022-01-07 09:56:18
tags:
---
> 思路：尝试实现超级简历的pdf样式，分析是通过html模板结合freemarker占位循环，配合itext3支持css3高级特性渲染富文本生成pdf。

<!--more-->

### 1.设计html模板，关键词部分用freemarker标签占位处理

```
<!DOCTYPE html>
<html lang="zh">
<head>
    <style type="text/css">
       // 样式省略
    </style>

</head>
<body>
<div id="head" class="space">
    <div class="title center">
        ${username}
    </div>
    <div class="center">
        ${phone} 丨${email}
    </div>
    <div class="center">
        ${address}
    </div>
    <div class="center">
        ${homepage}
    </div>
    <div class="center">
        ${birthday?string('yyyy-MM-dd')} 丨${gender}
    </div>
    <div class="center">
         ${currentJob} 丨${expJob}
    </div>
</div>
<#if skill?? &&skill!=''>
    <div class="small-title">专业技能</div>
    <hr/>
    <div class="content space">
        ${skill}
    </div>
</#if>
<#if works?? && (works?size > 0)>
    <div class="small-title">工作经历</div>
    <hr/>
    <div class="content space">
        <#list works as t>

            <p class="left company">${t.company}</p>
            <p class="right">${t.startAt} - ${t.endAt}</p>
            <p class="clear"></p>
            <p>${t.dept} |${t.job}</p>
            ${t.description}

        </#list>
    </div>
</#if>
<#if edus?? && (edus?size > 0)>
    <div class="small-title">教育经历</div>
    <hr/>
    <div class="content space">
        <#list edus as t>

            <p class="left company">${t.university}</p>
            <p class="right">${t.startAt?string('yyyy-MM')} - ${t.endAt?string('yyyy-MM')}</p>
            <p class="clear"></p>
            <p>${t.major} |${t.degree}</p>
            ${t.description}

        </#list>
    </div>
</#if>
<#if summary?? &&summary!=''>
    <div class="small-title">个人总结</div>
    <hr/>
    <div class="content space">
        ${summary}
    </div>
</#if>
</body>
</html>
```
### 2.freemarker渲染html

```
// 获取模板,并设置编码方式
Template template = freemarkerCfg.getTemplate(htmlTmp);
template.setEncoding("UTF-8");
// 合并数据模型与模板
template.process(data, out); //将合并后的数据和模板写入到流中，这里使用的字符流
out.flush();
createPdf(out.toString(),outputStream);
```
### 3.itext生成pdf
```
//css3高级特性支持
ITextRenderer render = new ITextRenderer();
ITextFontResolver fontResolver = render.getFontResolver();
fontResolver.addFont(FONT, BaseFont.IDENTITY_H, BaseFont.EMBEDDED);
// 解析html生成pdf
render.setDocumentFromString(content);
render.layout();
render.createPDF(outputStream);
```
### 4.前端接收文件流展示

```
response.setHeader("Content-Type", "application/pdf ");
response.setContentType("application/pdf ;charset=utf-8");
//不加👇就直接预览
response.setHeader("Content-Disposition", "attachment;filename=" + URLEncoder.encode("test.pdf", "UTF-8"));
```

我这里是通过小程序展示

![image](http://img.achiguo.com/blog/2091641521649_.pic.jpg)