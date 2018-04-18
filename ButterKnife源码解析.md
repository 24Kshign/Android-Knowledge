### ButterKnife源码解析

#### 1、介绍
butterknife主要是为了解决findViewById和setOnClickListener

#### 2、原理
主要采用编译时注解，就是采用APT生成代码。在编译源文件时，会分析扫描注解，当扫描到butterknife定义的@BindView、@OnClick等注解时，会使用JavaPoet来生成代码。生成后的文件会再次分析，直到没有分析到需要处理的注解位置。关键是AbstractProcessor（编译时注解处理器）

#### 4、什么是AbstractProcessor
在程序编译时编译器会检查AbstractProcessor的子类，并且调用该类型的process函数，然后将添加了该处理器支持注解的所有元素都传递到process函数中，使得开发人员可以在编译器进行相应的处理。

#### 3、什么是APT
APT(Annotation Processing Tool 的简称)，可以在代码编译期解析注解，并且生成新的 Java 文件，减少手动的代码输入。

#### 4、源码解析

```
@Target(FIELD)
@Retention(CLASS)
public @interface BindView {
  /** View ID to which the field will be bound. */
  int[] value();
}
```
@Retention(CLASS)表示编译时注解，在编译的时候所使用的。


```
Class<?> bindingClass = cls.getClassLoader().loadClass(clsName + "_ViewBinding");
//noinspection unchecked
bindingCtor = (Constructor<? extends Unbinder>) bindingClass.getConstructor(cls, View.class);
```

编译的时候通过上述代码生成一个.java文件--->.class文件

#### 5、定义注解的元注解

**`@Target`**：注解的作用目标

```
@Target(ElementType.TYPE)   //接口、类、枚举、注解
@Target(ElementType.FIELD) //字段、枚举的常量
@Target(ElementType.METHOD) //方法
@Target(ElementType.PARAMETER) //方法参数
@Target(ElementType.CONSTRUCTOR)  //构造函数
@Target(ElementType.LOCAL_VARIABLE)//局部变量
@Target(ElementType.ANNOTATION_TYPE)//注解
@Target(ElementType.PACKAGE) ///包   
```

**`@ Reteniton `**：定义被它所注解的注解保留多久，一共有三种策略，定义在RetentionPolicy枚举中

```
public enum RetentionPolicy {
    SOURCE,
    CLASS,
    RUNTIME
}
```

**1、SOURCE**：被编译器忽略

**2、CLASS**：(编译时)注解将会被保留在Class文件中，但在运行时并不会被VM保留。这是默认行为，所有没有用Retention注解的注解，都会采用这种策略。

**3、RUNTIME**：(运行时)保留至运行时。所以我们可以通过反射去获取注解信息。














