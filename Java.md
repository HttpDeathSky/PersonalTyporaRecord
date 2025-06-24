# Java基础

## 注解

引用：https://zhuanlan.zhihu.com/p/562451020

### 元注解

| 元注解      | 作用               |
| ----------- | ------------------ |
| @Target     | 设置目标范围       |
| @Retention  | 设置保持性         |
| @Inherited  | 注解继承           |
| @Repeatable | 此注解可以重复修饰 |
| @Documented | 注解可入文档       |

- @Retention

内有成员变量RetentionPolicy可用于指定注解保留的时间。**SOURCE**，注解仅存在源码中，不会保留在class文件中；**CLASS**，默认的方式，注解会存在于class文件(字节码)中，但不会被JVM加载；**RUNTIME**，注解保留在class文件中，而且会被JVM运行时访问到，这意味着后续能够结合反射技术将注解中的值获取出来。

- @Target

内有成员变量ElementType可用于限定目标注解作用与什么位置。

**METHOD**限定修饰方法，**ANNOTATION_TYPE**限定修饰注解，**CONSTRUCTOR**限定修饰构造器，**FIELD**限定修饰成员变量，**LOCAL_VARIABLE**限定修饰局部变量，**PACKAGE**限定修饰包，**PARAMETER**限定修饰参数，**TYPE**限定修饰上面提到的类型都可以修饰。

- @Iherited

使得一个类和它的子类都包含某个注解，也就是说子类能够继承父类具有的这个注解。换句话说，注解原本为父类所特有，如果子类想要具有父类一样的注解，那么需要再次在子类中书写，但现在如果这个注解如果受元注解@Iherited的注解，那么可以不用在子类中也写这个注解，子类就具有这个注解了。

- Documented

指明这个注解可以被Javadoc工具解析，形成帮助文档。

- @Repeatable

自JDK1.8引入，表示被修饰的注解可以重复应用注解，需要定义注解和容器注解。

```java
package test;

import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Repeatable(RepeatableAnnotations.class)  // 指定重复注解容器
public @interface RepeatableAnnotation {
    int a() default 1;
    int b() default 0;
    int c() default 0;
}
```

```java
package test;

import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Retention(RetentionPolicy.RUNTIME)
public @interface RepeatableAnnotations {
    RepeatableAnnotation[] value();
}
```

假设现在要对Teacher中的add进行测试，需要传入多个用例，那么我们可以把用例作为重复注解传入。

```java
package test;

public class Teacher<T> {
    T subject;
    public Teacher(T subject){
        this.subject = subject;
    }

    public void setSubject(T subject) {
        this.subject = subject;
    }

    public T getSubject() {
        return subject;
    }

    @RepeatableAnnotation(a=1,b=2,c=3)
    @RepeatableAnnotation(a=1,b=2,c=4)
    public static void add(int a, int b, int c) {
        if (c != a+b) throw new ArithmeticException("Wrong");
    }
}
```

下面展示，在Main函数中如何运行

```java
public class TestMain{
    public static void main(String[] args) {
        String className = "test.Teacher";
        for (Method m : Class.forName(className).getMethods()) {
            // 注意用的是容器，因为实际上RepeatableAnnotation是放在这个容器里的
            if (m.isAnnotationPresent(RepeatableAnnotations.class)) { 
                System.out.println(m.getName());
                RepeatableAnnotation[] annos = m.getAnnotationsByType(RepeatableAnnotation.class);
                for (RepeatableAnnotation anno : annos) {
                    System.out.println(anno.a() + " " + anno.b() + " " + anno.c());
                    try {
                        m.invoke(null, anno.a(), anno.b(), anno.c());
                    }catch (Throwable ex) {
                        System.out.printf("Test %s failed: %s %n", m, ex.getCause());
                        System.out.println();

                    }
                }
            }
        }
    }
}
```

## 数据实体命名规范

Domain
Domain这个包国外很多项目经常用到，字面意思是域的意思。

DAO
DAO（Data Access Object）数据访问对象，它是一个面向对象的数据库接口，负责持久层的操作，为业务层提供接口，主要用来封装对数据库的访问，常见操作无外乎 CURD。我们也可以认为一个 DAO 对应一个 POJO 的对象，它位于业务逻辑与数据库资源中间，可以结合 PO 对数据库进行相关的操作。(不怎么用)

POJO
POJO（Plain Ordinary Java Object）简单的 Java 对象，实际就是普通的 JavaBeans，是为了避免和 EJB（Enterprise JavaBean）混淆所创造的简称。POJO 实质上可以理解为简单的实体类，其中有一些属性及其getter和setter方法的类，没有业务逻辑，也不允许有业务方法，也不能携带有connection之类的方法。POJO 是 JavaEE 世界里面最灵活的对象，在简单系统中，如果从数据库到页面展示都是 POJO 的话，它可以是 DTO；如果从数据库中到业务处理中都是 POJO 的话，它可以是 BO；如果从数据库到整个页面的展示的话，它也可以是 VO。

PO
PO（Persistent Object）持久层对象，它是由一组属性和属性的get和set方法组成，最简单的 PO 就是对应数据库中某个表中的一条记录（也就是说，我们可以将数据库表中的一条记录理解为一个持久层对象），多个记录可以用 PO 的集合，PO 中应该不包含任何对数据库的操作。PO 的属性是跟数据库表的字段一一对应的，此外 PO 对象需要实现序列化接口。

BO
BO（Business Object）业务层对象，是简单的真实世界的软件抽象，通常位于中间层。BO 的主要作用是把业务逻辑封装为一个对象，这个对象可以包括一个或多个其它的对象。举一个求职简历的例子，每份简历都包括教育经历、项目经历等，我们可以让教育经历和项目经历分别对应一个 PO，这样在我们建立对应求职简历的 BO 对象处理简历的时候，让每个 BO 都包含这些 PO 即可。

VO
VO（View Object）值对象，通常用于业务层之间的数据传递，和 PO 一样也是仅仅包含数据而已，但 VO 应该是抽象出的业务对象，可以和表对应，也可以不对应，这根据业务的需要。 如果锅碗瓢盆分别为对应的业务对象的话，那么整个碗柜就是一个值对象。此外，VO 也可以称为页面对象，如果称为页面对象的话，那么它所代表的将是整个页面展示层的对象，也可以由需要的业务对象进行组装而来。

DTO
DTO（Data Transfer Object）数据传输对象，主要用于远程调用等需要大量传输对象的地方，比如我们有一个交易订单表，含有 25 个字段，那么其对应的 PO 就有 25 个属性，但我们的页面上只需要显示 5 个字段，因此没有必要把整个 PO 对象传递给客户端，这时我们只需把仅有 5 个属性的 DTO 把结果传递给客户端即可，而且如果用这个对象来对应界面的显示对象，那此时它的身份就转为 VO。使用 DTO 的好处有两个，一是能避免传递过多的无用数据，提高数据的传输速度；二是能隐藏后端的表结构。常见的用法是：将请求的数据或属性组装成一个 RequestDTO，再将响应的数据或属性组装成一个 ResponseDTO.