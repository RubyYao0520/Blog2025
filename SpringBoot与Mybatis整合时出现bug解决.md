今天在Spring Boot与MySQL数据库进行连接的时候出现了以下报错(直接眼前一黑😢)，

在网上进行搜索之后发现是maven依赖中的MyBatis与Spring Boot的版本不匹配

****

解决方法如下：

将MyBatis依赖的version改为2.2.2版本（与SpringBoot不发生冲突的版本）即可

小小的感悟：出现爆红时先不要慌，记得复制报错信息搜索一下，说不定就是个很简单的BUG。以及maven依赖的报错真是数不胜数，简直是噩梦。

```xml
<dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.2.2</version>
</dependency>
```

图片报错信息如下

![image-20250302130724513](http://image.slugyao.top/java/image-20250302130724513.png)