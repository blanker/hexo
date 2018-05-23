---
title: 《Effective Java 3rd Edition》第2章-第3条 使用私有构造方法或枚举类型强制实现单例属性 
date: 2018-05-21 13:33:01
categories:
    - Effective Java 3rd edition
tags: 
    - effective 
    - java 
    - builder 
    - constrauctor
---

# Item 3: Enforce the singleton property with a private constructor or an enum type

# 第3条：使用私有构造方法或枚举类型强制实现单例属性 

<!-- more -->

> A singleton is simply a class that is instantiated exactly once [Gamma95]. Singletons typically represent either a stateless object such as a function (Item 24) or a system component that is intrinsically unique. **Making a class a singleton can make it difficult to test its clients** because it’s impossible to substitute a mock implementation for a singleton unless it implements an interface that serves as its type.

单例是一种简单的类，只能被实例化一次[Gamma95]。单例通常用来表示一个无状态的对象，比如函数，或者表示一个具有唯一性的系统组件。**单例类的客户端代码比较难以测试**，因为不可能模仿出单例的替代品，除非这个单例实现了一个接口，该接口对外提供服务。

>    There are two common ways to implement singletons. Both are based on keeping the constructor private and exporting a public static member to provide access to the sole instance. In one approach, the member is a final field:

实现单例的方法通常有两种，它们的都是基于私有构造方法，并对外暴露一个公开的静态成员来访问唯一的那个实例。在其中一种实现中，静态成员是一个final修饰的字段：

```java
// Singleton with public final field
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }

    public void leaveTheBuilding() { ... }
}
```

>    The private constructor is called only once, to initialize the public static final field Elvis.INSTANCE. The lack of a public or protected constructor guaranttees a “monoelvistic” universe: exactly one Elvis instance will exist once the Elvis class is initialized—no more, no less. Nothing that a client does can change this, with one caveat: a privileged client can invoke the private constructor reflectively (Item 65) with the aid of the AccessibleObject.setAccessible method. If you need to defend against this attack, modify the constructor to make it throw an exception if it’s asked to create a second instance.
    
要初始化公开的static final字段Elvis.INSTANCE，私有构造方法只会被调用一次。由于没有公开的或受保护的构造方法，可以确保这是一个"单一"世界：一旦Elvis类被初始化，只会存在一个Elvis实例，不多也不少。客户端没法改变这一点，不过有一点需要注意，获得授权的客户端借助AccessibleObject.setAccessible方法，通过反射可以调用私有构造方法。如果想要防御这种攻击，可以修改构造方法，当被请求创建第二个实例的时候，就抛出异常。

>    In the second approach to implementing singletons, the public member is a static factory method:

单例的第二种实现中，公开成员是一个静态工厂方法：

```java
// Singleton with static factory
// 使用静态工厂的单例
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() { ... }
}
```

>    All calls to Elvis.getInstance return the same object reference, and no other Elvis instance will ever be created (with the same caveat mentioned earlier).

所有对Elvis.getInstance方法的调用都返回同一个对象的引用，决不会有其它Elvis类的实例被创建（同样需要注意之前提到的那个问题）。

>    The main advantage of the public field approach is that the API makes it clear that the class is a singleton: the public static field is final, so it will always contain the same object reference. The second advantage is that it’s simpler.

使用公开字段实现单例的主要优点是，API明确表明该类是单例的：公开的静态字段以final修饰，因此总是包含相同对象的引用；第二个优点是更简单。

>    One advantage of the static factory approach is that it gives you the flexibility to change your mind about whether the class is a singleton without changing its API. The factory method returns the sole instance, but it could be modified to return, say, a separate instance for each thread that invokes it. A second advantage is that you can write a generic singleton factory if your application requires it (Item 30). A final advantage of using a static factory is that a method reference can be used as a supplier, for example Elvis::instance is a Supplier<Elvis>. Unless one of these advantages is relevant, the public field appproach is preferable.
    
使用静态工厂方法实现单例的一个优点是，它非常灵活，将来你想改变主意决定该类还是不是单例的时候，并不需要修改API。现在的工厂方法返回那个唯一的实例，但是将来可以把它修改为给每个调用该方法的线程分别返回一个实例。第二个好处是，如果应用有需要，你可以编写一个支持泛型的单例工厂方法。使用静态工厂方法的最后一个好处是，可以把方法引用用作生产者，比如Elvis::instance就是一个Supplier<Elvis>。除非的确用到这些好处，否则应首选使用公开字段实现单例。

>    To make a singleton class that uses either of these approaches serializable (Chapter 12), it is not sufficient merely to add implements Serializable to its declaration. To maintain the singleton guarantee, declare all instance fields transient and provide a readResolve method (Item 89). Otherwise, each time a serialized instance is deserialized, a new instance will be created, leading, in the case of our example, to spurious Elvis sightings. To prevent this from happening, add this readResolve method to the Elvis class:

要让两种实现的单例类可序列化（第12章），仅仅在类的声明上添加实现Serializable接口是不够的，为了保证单例，应该用transient修饰所有的实例字段，并提供一个readResolve方法（第89条），否则的话，每次序列化的实例被反序列化的时候，都会创建一个新的实例，应用到我们的例子里面，就会导致出现错误的多Elvis实例。为避免这种情况，需要给Elvis类添加readResolve方法：

```java
// readResolve method to preserve singleton property
// 通过readResole方法来保持单例属性
private Object readResolve() {
     // Return the one true Elvis and let the garbage collector
     // take care of the Elvis impersonator.
    return INSTANCE;
}
```

>    A third way to implement a singleton is to declare a single-element enum:
第三种实现单例的方法是声明一个只有一个元素的枚举：

```java
// Enum singleton - the preferred approach
// 枚举型单例——首选的实现方式
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() { ... }
}
```

> This approach is similar to the publlic field approach, but it is more concise, provides the serialization machinery for free, and provides an ironclad guarantee against multiple instantiation, even in the face of sophisticated serialization or reflection attacks. This approach may feel a bit unnatural, but **a single-element enum type is often the best way to implement a singleton**. Note that you can’t use this approach if your singleton must extend a superclass other than Enum (though you can declare an enum to implement interfaces).
 
这种实现方式和公开字段的实现方式很相似，不过相对来说更简洁，并且免费提供了序列化机制，提供了免受多实例化的屏障，即使面对复杂的序列化场景或利用反射技术的攻击。虽然感觉有点不自然，但是 **使用单个元素的枚举类型来实现总是最好的方式**。值得注意的是，如果你的单例必须继承自一个非Enum类的父类，就不能使用这种方式（不过可以声明一个枚举来实现接口）。
