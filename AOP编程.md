### 什么是AOP？

AOP就是面向切面编程，与OOP(面向对象编程)一样，是一种编程思想。AOP的思想运用在编程中可以在不改动原有设计的基础上进行功能的增强。

### AOP中的主要概念

通知：简单来说就是抽取出的共性功能。

切面：描述通知与切入点的关系，通俗来说就是哪些方法要追加哪些功能。

连接点：就是方法，因为共性功能是要追加到方法里面的，因此连接点就是方法。

切点：要抽取共性功能的方法称为切点，切点是连接点的子集。

织入：将要添加的共性功能添加到的目标对象中的过程。



### AOP的原理

其实对于初学者来说，不需要太清楚AOP的实现原理，但我还是决定写一下。知道它底层实现的原理也许能在未来解决一些bug。

#### 代理模式

要讲AOP的底层原理，首先要讲什么是“代理”。假设有这样一个场景：你想要买一双鞋，但是一般情况下你是从商店里面买到这双鞋，而不是从这双鞋生产的地方买的。商店就在这一过程中充当了“代理”的角色。在买鞋的时候，你是很难直接接触到工厂的，而商店，这个“代理”者，能帮我们从工厂那里拿到鞋子。那么在购买这双鞋的过程中，商店就充当了“代理”的角色。



### 示例

这里我们使用aspectj来实现AOP

项目背景：在一个项目中，调用接口时打印日志

首先梳理一下大致的思路

导入坐标到pom.xml文件中

一般来说添加以下这两个依赖即可

```xml
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
        </dependency>
```

首先创建一个打印日志的注解MyLog

```java
package com.blog.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)//运用在方法上
@Retention(RetentionPolicy.RUNTIME)//在运行时执行
public @interface MyLog {
    String title() default "";
}
```

然后制作共性功能

这里的共性功能就是打印日志，我们就简单写一下

在aspect包下创建一个切面类

@AfterReturning注解在方法返回后进入切面的通知方法中

```Java
package com.blog.aspect;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

import java.util.Date;

@Aspect
@Component
public class LogAspect {
    @AfterReturning(pointcut = "@annotation(com.blog.annotation.MyLog)")
    public void log(JoinPoint joinPoint) {
        System.out.println("log:");
        handler(joinPoint);
    }

    protected void handler(JoinPoint joinPoint) {
        Date date = new Date();
        String className = joinPoint.getTarget().getClass().getName();
        System.out.println(className);
        System.out.println(date.getTime());
    }
}

```

这里面的pointcut指定为添加了MyLog注解的方法，也就是添加了@MyLog注解的方法才会进入AOP的逻辑中打印日志。

