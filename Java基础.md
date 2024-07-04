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