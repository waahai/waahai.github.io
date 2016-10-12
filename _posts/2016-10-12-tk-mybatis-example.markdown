---
layout: post
title:  tk-mybatis中example的使用介绍
tags: ORM tk-mybatis example
categories: Java
---
tk-mybatis中的example可以进行简单的查询，它提供了类似全自动ORM框架的一些接口来设置查询条件，比如

{% highlight java %}
public PageInfo queryUsers(String keyword, int page, int pageSize) {
  PageHelper.startPage(page, pageSize);
  Example query = new Example(MyObject.class);
  query.or().andLike("field", "%"+keyword+"%");
  return new PageInfo(mapper.selectByExample(query));
}  
{% endhighlight %}

当搜索多个字段时，也可以方便的实现，如下

{% highlight java %}
public PageInfo queryUsers(String keyword, int page, int pageSize) {
  PageHelper.startPage(page, pageSize);
  Example query = new Example(MyObject.class);
  query.or().andLike("field1", "%"+keyword+"%");
  query.or().andLike("field2", "%"+keyword+"%");
  return new PageInfo(mapper.selectByExample(query));
}  
{% endhighlight %}

但是Example只支持顶层为or的并列条件，如`条件A or 条件B`, 并不能直接实现`(条件A or 条件B) and (条件1 or 条件2)`这种查询
研读源码[Example](https://github.com/abel533/Mapper/blob/master/src/main/java/tk/mybatis/mapper/entity/Example.java)后，发现它同时提供了`Example.Criteria`类， 但Criteria只支持and的并列条件`条件1 and 条件2`
有了这两个，要实现`(条件A or 条件B) and (条件1 or 条件2)`， 我们可以先把条件展开成or并列的条件`(条件A and 条件1) or (条件A and 条件2) or (条件B and 条件1) or (条件B and 条件2)`,然后用下面的代码实现在两组field中搜索对应关键字的功能

{% highlight java %}
public PageInfo queryUsers(String keyword1, String keyword2, int page, int pageSize) {
  PageHelper.startPage(page, pageSize);
  Example query = new Example(MyObject.class);
  String[] searchFields1 = new String[]{"field1", "field2"};
  String[] searchFields2 = new String[]{"field3", "field4"};

  for(String f1 : searchFields1) {
    for(String f2 : searchFields2) {
      Example.Criteria criteria = query.or();
      criteria.andLike(f1, "%"+keyword1+"%");
      criteria.andLike(f2, "%"+keyword2+"%");
    }
  }
  return new PageInfo(mapper.selectByExample(query));
}  
{% endhighlight %}
