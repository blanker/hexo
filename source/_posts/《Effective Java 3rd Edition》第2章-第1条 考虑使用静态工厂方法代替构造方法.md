---
title: 《Effective Java 3rd Edition》第2章-第1条 考虑使用静态工厂方法代替构造方法 
date: 2018-05-13 22:33:01
categories:
    - Effective Java 3rd edition
tags: 
    - effective 
    - java 
    - static factory method 
    - constrauctor
---

# Chapter 2. Creating and Destroying Objects

> THIS chapter concerns creating and destroying objects: when and how to create them, when and how to avoid creating them, how to ensure they are destroyed in a timely manner, and how to manage any cleanup actions that must precede their destruction.

<!-- more -->

# 第二章 创建和销毁对象
本章涉及创建和销毁对象的方方面面：什么时候创建对象，怎样创建对象，怎样避免创建对象；怎样确保在适当的时机销毁对象，并且在销毁前做好清理工作。


## Item 1: Consider static factory methods instead of constructors

> The traditional way for a class to allow a client to obtain an instance is to provide a public constructor. There is another technique that should be a part of every programmer’s toolkit. A class can provide a public static factory method, which is simply a static method that returns an instance of the class. Here’s a simple example from Boolean (the boxed primitive class for boolean). This method translates a boolean primitive value into a Boolean object reference:

## 第1条：考虑使用静态工厂方法代替构造方法

客户端获取一个类的实例的典型方法是通过public修饰的构造方法，此外还有另一种技术，应该放进每个码农的工具箱中，类可以提供public修饰的静态工厂方法，该方法返回一个类的实例。下面是Boolean类（原生布尔类型的包装类型）中的简单例子，该方法把原生布尔类型转换为Boolean对象。

```java
public static Boolean valueOf(boolean b){
    return b?Boolean.TRUE:Boolean.FALSE;
}
```

> Note that a static factory method is not the same as the Factory Method pattern from Design
Patterns [Gamma95]. The static factory method described in this item has no direct
equivalent in Design Patterns.


要注意的是，静态工厂方法跟设计模式[Gamma95]里的工厂方法是不同的，本条目中描述的静态工厂方法在设计模式里没有直接等价物。

> A class can provide its clients with static factory methods instead of, or in addition to, public constructors. Providing a static factory method instead of a public constructor has both advantages and disadvantages.

类提供静态工厂方法给客户端，来代替public修饰的构造方法，或者作为构造方法的一种补充，这样做有优点也有缺点。

> One advantage of static factory methods is that, unlike constructors,they have names. If the parameters to a constructor do not, in and of themselves, describe the object being returned, a static factory with a well-chosen name is easier to use and the resulting client code easier to read. For example, the constructor BigInteger(int, int, Random), which returns a BigInteger that is probably prime, would have been better expressed as a static factory method named BigInteger.probablePrime. (This method was added in Java 4.)

静态工厂方法的第一个优点是，它们有名字，这一点是和构造方法不一样的。构造方法不能通过参数来描述返回的对象的含义，相反，精心取名的静态工厂方法更容易使用，客户端代码也易于阅读。举个例子，构造方法BigInteger(int, int, Random)返回的是概素数，对于这一点，静态工厂方法BigInteger.ProbablePrime可以表述得更好。(这个方法是java4添加的)

> A class can have only a single constructor with a given signature. Programmers have been known to get around this restriction by providing two constructors whose parameter lists differ only in the order of their parameter types. This is a really bad idea. The user of such an API will never be able to remember which constructor is which and will end up calling the wrong one by mistake. People reading code that uses these constructors will not know what the code does without referring to the class documentation.

对于给定的方法签名，类只能有一个构造方法，为了解决这个限制，码农们常用的做法是提供两个构造方法，只是通过不同顺序的参数类型加以区别，这种做法真是糟透了。这种API的使用者总是记不住哪个构造方法是哪个，结果就是造成错误的调用。如果不关联类文档，阅读使用这些构造方法的代码将没法明白代码的含义。

> Because they have names, static factory methods don’t share the restriction discussed in the previous paragraph. In cases where a class seems to require multiple constructors with the same signature, replace the constructors with static factory methods and carefully chosen names to highlight their differences.

因为有不同名称，静态工厂方法没有上一节所讨论的这种限制。如果一个类看起来需要多个方法签名都相同的构造方法，此时可以用静态工厂方法来代替构造方法，并通过精心命名的方法名称来区别不同含义。

> A second advantage of static factory methods is that, unlike constructors, they are not required to create a new object each time they’re invoked. This allows immutable classes (Item 17) to use preconstructed instances, or to cache instances as they’re constructed, and dispense them repeatedly to avoid creating unnecessary duplicate objects. The Boolean.valueOf(boolean) method illustrates this technique: it never creates an object. This technique is similar to the Flyweight pattern [Gamma95]. It can greatly improve performance if equivalent objects are requested often,especially if they are expensive to create.

静态工厂方法的第二个优点是，每次被调用的时候，并非都需要创建一个新对象，这是和构造方法不一样的。这样就能让不可变类（第17条）使用预先创建好的实例对象，或者在创建实例的时候把它缓存起来，后面就能重复使用，避免创建不必要的重复对象。比如，Boolean类的valueOf(boolean)方法就描述了这样的技术：该方法并不会创建一个新对象。这种技术和Flyweight模式有点类似。当需要频繁请求等价对象的时候，这个技术能极大提高程序的性能，特别是创建对象的成本比较高的时候。

> The ability of static factory methods to return the same object from repeated invocations allows classes to maintain strict control over what instances exist at any time. Classes that do this are said to be instance-controlled. There are several reasons to write instance-controlled classes. Instance control allows a class to guarantee that it is a singleton (Item 3) or noninstantiable (Item 4). Also, it allows an immutable value class (Item 17) to make the guarantee that no two equal instances exist: a.equals(b) if and only if a == b.This is the basis of the Flyweight pattern [Gamma95]. Enum types (Item 34) provide this guarantee.

对静态工厂方法的多次调用，都返回同一个对象，使用这种能力，使得类能够严格控制在任何时刻拥有的实例数量。拥有这种行为的类被称作可控制实例的类。下面的一些原因描述了这种类存在的原因：确保类是单实例（第3条）并且不能被实例化的（第4条）；确保不可变类（第17条）的两个不同实例，它们的值肯定不相等，也就是说，只有两个实例完全相同时，它们的值才相等；这也是Flyweight模式的基础。枚举类型（第34条）也提供这样的特性。

> A third advantage of static factory methods is that, unlike constructors, they can return an object of any subtype of their return type. This gives you great flexibility in choosing the class of the returned object.

静态工厂方法的第三个优点是，和构造方法不一样，静态工厂方法可以返回所定义的返回类型的任意子类型，这为我们在选择返回对象类型的时候，提供了巨大的灵活性。

> One application of this flexibility is that an API can return objects without making their classes public. Hiding implementation classes in this fashion leads to a very compact API. This technique lends itself to interface-based frameworks (Item 20), where interfaces provide natural return types for static factory methods.

对这种灵活性的一种应用是，API能够返回非public类的对象。通过这种方式把实现类隐藏起来，使得API非常紧凑。基于接口的框架（第20条）就是借助这种技术来建立的，一般通过静态工厂方法返回接口作为原始返回类型。

> Prior to Java 8, interfaces couldn’t have static methods. By convention, static factory methods for an interface named Type were put in a noninstantiable companion class (Item 4) named Types. For example, the Java Collections Framework has forty-five utility implementations of its interfaces, providing unmodifiable collections, synchronized collections, and the like. Nearly all of these implementations are exported via static factory methods in one noninstantiable class (java.util.Collections). The classes of the returned objects are all nonpublic.

在Java8之前，接口不能拥有静态方法。按照约定，一个名称为Type的接口，为它设计的静态工厂方法，会放在一个不允许实例化的伴生类里，该类取名为Types。举个例子，Java集合框架为它的接口提供了45个辅助实现类，包括不可修改集合、同步集合等等，几乎所有这些实现都是在一个不能实例化的类（java.util.Collections）里，通过静态工厂方法暴露出来的，这些返回对象的类都是非public修饰的。

> The Collections Framework API is much smaller than it would have been had it exported forty-five separate public classes, one for each convenience impplementation. It is not just the bulk of the API that is reduced but the conceptual weight: the number and difficulty of the concepts that programmers must master in order to use the API. The programmer knows that the returned object has precisely the API specified by its interface, so there is no need to read additional class documentation for the implementation class. Furthermore, using such a static factory method requires the client to refer to the returned object by interface rather than implementation class, which is generally good practice (Item 64).

相对于为每个接口提供一个单独实现类，总共暴露45个public类，目前的集合框架的API精简太多了。这里说的精简不仅是大量API的减少，还是概念上的负担精简：码农们为使用这些API必须掌握的概念，从数量和复杂度上来说，都得到了精简。码农们已经通过接口描述知道了确切的返回类型，因此根本没必要再去阅读具体实现类的文档。另外，使用静态工厂方法要求客户端通过接口关联返回类型，而不是通过具体实现类，这普遍认为是一种良好的实践。

> As of Java 8, the restriction that interfaces cannot contain static methods was eliminated, so there is typically little reason to provide a noninstantiable companion class for an interface. Many public static members that would have been at home in such a class should instead be put in the interface itself. Note, however, that it may still be necessary to put the bulk of the implementation code behind these static methods in a separate package-private class. This is because Java 8 requires all static members of an interface to be public. Java 9 allows private static methods, but static fields and static member classes are still required to be public.

Java8取消了接口中不允许有静态方法的限制，因此，通过为接口提供一个不能实例化的伴生类的传统做法，显得不再那么合理。以前在这种伴生类里的public修饰的静态成员，应该放在接口里。然而，值得注意的是，还是有必要把大量隐藏在那些静态方法背后的实现代码放在一个单独的包级别私有类里。因为Java8规定接口中的所有静态成员都必须用public修饰，Java9倒是允许private来修饰静态方法，但是静态字段和静态类还是需要public修饰。

> A fourth advantage of static factories is that the class of the returned object can vary from call to call as a function of the input parameters. Any subtype of the declared return type is permissible. The class of the returned object can also vary from release to release.

静态工厂的第四个优点是，随着方法调用时输入参数的不同，返回对象的类型可以不同。返回定义的返回类型的任意子类型都是允许的，不同版本发布时，也可以选择不同的返回类型。

> The EnumSet class (Item 36) has no public constructors, only static factories. In the OpenJDK implementation, they return an instance of one of two subclasses, depending on the size of the underlying enum type: if it has sixty-four or fewer elements, as most enum types do, the static factories return a RegularEnumSet instance, which is backed by a single long; if the enum type has sixty-five or more elements, the factorries return a JumboEnumSet instance, backed by a long array.

EnumSet类（第36条）没有public构造方法，只有一些静态工厂方法。在OpenJDK的实现中，取决于底层枚举类型的大小，这些方法返回两个子类中的一个：绝大部分枚举类型的元素个数低于64（含64），此时工厂方法返回的是RegularEnumSet的实例，其底层是一个简单的长整型；当枚举类型的元素个数超过65（含65）的时候，工厂方法返回的是一个JumboEnumSet的实例，其底层是一个长整型数组。

> The existence of these two implementation classes is invisible to clients. If RegularEnumSet ceased to offer performance advantages for small enum types, it could be eliminated from a future release with no ill effects. Similarly, a future release could add a third or fourth implementation of EnumSet if it proved beneficial for performance. Clients neither know nor care about the class of the object they get back from the factory; they care only that it is some subclass of EnumSet.

这两个实现类的存在对客户端程序是不可见的，如果RegularEnumSet类的停用能带来对小枚举类型性能的提高，就可以在将来发布新版本的时候取消它，而不会造成不良影响。同样的道理，将来的发行版可以为EnumSet类添加第三个、第四个实现类，以带来性能的提升。客户端并不知道也不用关心工厂方法返回对象的类型，只需要知道该返回对象的类型是EnumSet类的子类。

> A fifth advantage of static factories is that the class of the returned object need not exist when the class containing the method is written. Such flexible static factory methods form the basis of service provider frameworks, like the Java Database Connectivity API (JDBC). A service provider framework is a system in which providers implement a service, and the system makes the implementations available to clients, decoupling the clients from the implementations.

静态工厂方法的第五个优点是，编写包含静态方法类代码的时候，方法返回对象的类并不需要实际存在。这种灵活的静态工厂方法为实现服务提供者框架提供了基础，比如Java数据库连接API（简称JDBC）。服务提供者框架中，厂商实现一个服务，框架让客户端能使用该具体实现，并将客户端与具体实现进行解耦。

> There are three essential components in a service provider framework: a service interface, which represents an implementation; a provider registration API, which providers use to register implementations; and a service access API, which clients use to obtain instances of the service. The service access API may allow clients to specify criteria for choosing an implementation. In the absence of such criteria, the API returns an instance of a default implementation, or allows the client to cycle through all available implementations. The service access API is the flexible static factory that forms the basis of the service provider framework.

服务提供者框架中有三个基本组件：一个代表具体实现的服务接口；一个为提供者用以注册具体实现的提供者注册API；一个为客户端用以获取服务实例的服务访问API。服务访问API允许客户端通过特定规则选择具体的实现，不指定规则的时候，API会返回一个默认具体实现的实例，或者允许客户端循环使用所有可用的具体实现。服务访问API是构成服务提供者基础中最灵活的静态工厂。

> An optional fourth component of a service provider framework is a service provider interface, which describes a factory object that produce instances of the service interface. In the absence of a service provider interface, implementattions must be instantiated reflectively (Item 65). In the case of JDBC, Connection plays the part of the service interface, DriverManager.registerDriver is the provider registration API, DriverManager.getConnection is the service access API, and Driver is the service provider interface.

服务提供者框架中还有第四个可选的组件，这个组件就是服务提供者接口，它是生成服务接口实例的工厂对象。缺少服务提供者接口的话，具体实现必须使用反射进行实例化（第65条）。在JDBC的场景中，Connection类扮演的是服务接口的角色，DriverManager.registerDriver是服务注册API，DriverManager.getConnection是服务访问API，Driver是服务提供者接口。

> There are many variants of the service provider framework pattern. For example, the service access API can return a richer service interface to clients than the one furnished by providers. This is the Bridge pattern [Gamma95]. Dependency injection frameworks (Item 5) can be viewed as powerful service providers. Since Java 6, the platform includes a general-purpose service provider framework, java.util.ServiceLoader, so you needn’t, and generally shouldn’t, write your own (Item 59). JDBC doesn’t use ServiceLoader, as the former predates the latter.

服务提供者框架有很多种变体，举个例子，服务访问API可以给客户端返回一个比厂商提供的更丰富的增强版服务接口，这就是桥接模式。依赖注入可以看作是一种强大的服务提供者。从Java6开始，平台包含了一个通用的服务提供者框架，就是java.util.ServiceLoader，所以没有必要，也不用自己实现一个这样的框架，JDBC没有使用ServiceLoader，因为前者出现的时间比后者早。

> The main limitation of providing only static factory methods is that classes without public or protected constructors cannot be subclassed. For example, it is impossible to subclass any of the convenience implementation classes in the Collections Framework. Arguably this can be a blessing in disguise because it encourages programmers to use composition instead of inheritance (Item 18), and is required for immutable types (Item 17).

一个类如果没有public或protected修饰的构造方法，仅仅提供静态工厂方法，最主要的限制就是该类不能被子类继承。比如，在集合框架中，想要生成任何一个合适的子类都是不可能的。按理来说，这也可以因为隐蔽性而被认为是一件好事，因为这种情况会鼓励程序员们使用组合而不是继承方式（第18条），并且不可变类型也需要这样实现（第17条）。

> A second shortcoming of static factory methods is that they are hard for programmers to find. They do not stand out in API documentation in the way that constructors do, so it can be difficult to figure out how to instantiate a class that provides static factory methods instead of constructors. The Javadoc tool may someday draw attention to static factory methods. In the meantime, you can reduce this problem by drawing attention to static factories in class or interface documentation and by adhering to common naming conventions. Here are some common names for static factory methods. This list is far from exhaustive:

静态工厂方法的第二个缺点是，它们不易被程序员找到。在API文档中，它们没有构造方法那么突出的地位，因此，当提供静态工厂方法而不是构造方法来实例化一个类的时候，要找到实例化类的方法会困难一些。也许有朝一日生成Javadoc的工具会多考虑一下静态工厂方法，就目前来说，为减少这种问题的出现，可以在类和接口文档中对静态工厂方法多着笔墨，同时遵守常见的命名约定。下面是一些常见的静态工厂方法名称，这里并没有列出全部：

> from—A type-conversion method that takes a single parameter and returns a corresponding instance of this type, for example:
```java
Date d = Date.from(instant);
```

from —— 接收一个参数的类型转换方法，返回该类型相对应的一个，例如
```java
Date d = Date.from(instant);
```

> of—An aggregation method that takes multiple parameters and returns an instance of this type that incorporates them, for example:
```java
Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
```

of——接收多个参数的聚合方法，返回一个包含这些参数值的该类型的实例，例如：
```java 
Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
```

> valueOf—A more verbose alternative to from and of, for example:
```java
BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
```

valueOf——是from和of方法的冗长版，举个例子：

```java
BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
```

> instance or getInstance—Returns an instance that is described by its parameters (if any) but cannot be said to have the same value, for example:
```java
StackWalker luke = StackWalker.getInstance(options);
```

> instance或getInstance——返回一个实例，该实例基于方法参数的值而创建，不过不一定和参数拥有相同的值，举个例子：
```java
StackWalker luke = StackWalker.getInstance(options);
```

> create or newInstance—Like instance or getInstance, except that the method guarantees that each call returns a new instance, for example:
``` java
Object newArray = Array.newInstance(classObject, arrayLen);
```

create或newInstance——和instance、getInstance类似，不同的是，该方法确保每次调用都返回一个新的实例，举个例子：
```java
Object newArray = Array.newInstance(classObject, arrayLen)
```

> getType—Like getInstance, but used if the factory method is in a different class. Type is the type of object returned by the factory method, for example:
``` java
FileStore fs = Files.getFileStore(path);
```

getType——和getInstance类似，但是用在工厂方法位于不同类中的情况，其中Type是方法返回对象的类型，举个例子：
``` java
FileStore fs = Files.getFileStore(path);
```

> newType—Like newInstance, but used if the factory method is in a different class. Type is the type of object returned by the factory method, for example:
```java
BufferedReader br = Files.newBufferedReader(path);
```

> newType——和newInstance类似，但是用在工厂方法位于不同类中的情况，其中Type是方法 返回对象的类型，举个例子：
```java
BufferedReader br = Files.newBufferedReader(path);
```

> type—A concise alternative to getType and newType, for example:
```java
List<Complaint> litany = Collections.list(legacyLitany);
```

type——是getType和newType的简洁替代版，举个例子：
```java
List<Complaint> litany = Collections.list(legacyLitany)
```

> In summary, static factory methods and public constructors both have their uses, and it pays
to understand their relative merits. Often static factories are preferable, so avoid the reflex to
provide public constructors without first considering static factories.

总结：静态工厂方法和public构造方法都有它们的用途，我们应理解各自的优点。通常情况下，应优先考虑使用静态工厂方法。因此，不要有这样的习惯行为；提供public构造方法，而不是首先考虑静态工厂方法。