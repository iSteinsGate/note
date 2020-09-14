一级缓存

使用PerpetualCache



二级缓存

使用SerializedCache

### 一级缓存与二级缓存

mybatis默认开启一级缓存

开启二级缓存：

 第一步：在mybatis-config.xml全局配置中启用二级缓存

```xml
<setting name="caheEnabled" value="true">
```

第二步：在对应的mapper.xml中添加启用二级缓存配置

```xml
<cache></cache>
```



测试一级缓存和二级缓存

1.使用一级缓存

```java
//条件e，
Entity example = new Entity();
List<Entity> list1 = dao.findBy(e);
//同一个条件e
List<Entity> list2 = dao.findBy(e);
```

**结果**：list1与list2返回同一个对象即地址一样

2.取消一级缓存：对应的statement设置flushCache为true即可

```xml
<select id="selectBy" flushCache="true">
```

**结果**：list1与list2返回不同的对象

3.使用二级缓存：

**结果：**list1与list2返回不同的对象  

**原因分析：**

二级缓存使用的是SerializedCache缓存，缓存对象时会进行序列化和反序列化，导致第一次和第二次获取到的是不同的对象。

![image-20200911161519209](https://raw.githubusercontent.com/iSteinsGate/picture/master/assets/image-20200911161519209.png)

 而一级缓存使用的是PerpetualCache缓存   ，直接使用的是HashMap

![image-20200911165638970](https://raw.githubusercontent.com/iSteinsGate/picture/master/assets/image-20200911165638970.png)                     

注意：**

（1）当为select语句时：

flushCache默认为false，表示任何时候语句被调用，都不会去清空本地缓存和二级缓存。

useCache默认为true，表示会将本条语句的结果进行二级缓存。

（2）当为insert、update、delete语句时：

flushCache默认为true，表示任何时候语句被调用，都会导致本地缓存和二级缓存被清空。

useCache属性在该情况下没有。

当为select语句的时候，如果没有去配置flushCache、useCache，那么默认是启用缓存的，所以，如果有必要，那么就需要人工修改配置



### 参考文档

[美团mybatis缓存机制](https://tech.meituan.com/2018/01/19/mybatis-cache.html)