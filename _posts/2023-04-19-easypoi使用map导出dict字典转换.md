---
title: easypoi使用map模板导出dict字典转换
date: 2023-04-19 16:19:45 +0800
tags: easypoi map dict
---
## 背景
有一个导出原来用的map导出，现在有字段需要做数值转换
## 模板处理

```
申请人公司	申请时间	申请单号	产品名称	申请用户	状态	回收标记
$fe:map1 t.companyName	 t.create_time	t.id	t.productName	t.creator	dict:applyStatus;t.status	t.recycleFlag
```
以上为excel模板内容 其中状态字段 dict:applyStatus;t.status，dict 代表这个是字段处理,applyStatus 字典key


## 代码处理
需要自定义字典转换接口，实现`IExcelDictHandler` 接口，然后在模板参数中设置一下

我们来看一下源码
```
public interface IExcelDictHandler {
    //获取字典所有值，带入下拉
    default List<Map> getList(String dict) {
        return null;
    }
    //值->名称
    String toName(String var1, Object var2, String var3, Object var4);
    //名称->值
    String toValue(String var1, Object var2, String var3, Object var4);
}
```

例如我们自己实现的接口
```
 @Override
    public String toName(String dict, Object obj, String name, Object value) {
        if ("applyStatus".equals(dict)) {
            if (Integer.parseInt(String.valueOf(value))==0) {
                return "申请中";
            }else if (Integer.parseInt(String.valueOf(value))==1) {
                return "审核通过";
            }else if (Integer.parseInt(String.valueOf(value))==2) {
                return "不通过";
            }
        }
        return "-";
    }
```
然后设置
```
TemplateExportParams params = new TemplateExportParams(
                "test.xlsx");
        params.setDictHandler(new IExcelDictHandlerImpl());
```

即可完成值到名称的字典转换，也可以获取系统字典值进行查询转换