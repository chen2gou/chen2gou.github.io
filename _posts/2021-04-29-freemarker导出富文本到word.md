---
title: freemarker导出富文本到word
date: 2021-04-29 16:03:55
tags:
  - java
---
> 场景：富文本编辑器中的文本信息存储到数据库，为html语言，需要把富文本解析然后导出到word中。由于不存在图片信息，故此文不做讨论。


####  尝试方法
1. 使用`easy-poi`导出word，单个字段方便展示，但是富文本信息无法解析，list类型数据需要解析也没有良好的解决方法，尽管有遍历，但是仅限于表格之中。归根结底是由于poi对于word的支持不太行。
2. 使用freemarker标签占位word对应位置，然后另存为`mht`格式，类似html的一种文件格式。


<!-- more -->

####  思路解析
1. 编辑word模板，freemarker标签占位
2. idea中打开对应文件，另存之后会产生格式错乱，需要修改占位符号正确，最后另存为ftl格式。
3. **（*重要*）解析需要展示的文本为3Dus-asci编码值，使用freemarker渲染模板。**

##### 核心转换，将对象中所有字符换成3Dus-asci，十进制Accsii码，mht可以识别转换为html格式
```
public static String string2Ascii(String source){
		if(source==null || source==""){
			return null;
		}
		StringBuilder sb=new StringBuilder();
		
		char[] c=source.toCharArray();
		for(char item : c){
			String itemascii="";
			if(item>=19968 && item<40623){
				itemascii=itemascii="&#"+(item & 0xffff)+";";
			}else{
				itemascii=item+"";
			}
			sb.append(itemascii);
		}
		
		return sb.toString();
		
	}
```


#### 参考文档
[chaofanHb/ExpordWord  java填充word文档（带有富文本）](https://github.com/chaofanHb/ExpordWord)