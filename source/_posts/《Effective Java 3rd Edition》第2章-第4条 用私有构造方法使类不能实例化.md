---
title: 《Effective Java 3rd Edition》第2章-第4条：利用私有构造方法强制类不能被实例 
date: 2018-05-23 16:33:01
categories:
    - Effective Java 3rd edition
tags: 
    - effective 
    - java 
    - noninstantiability 
    - constrauctor
---

# Item 4: Enforce noninstantiability with a private constructor

# 第4条：利用私有构造方法强制类不能被实例 

<!-- more -->

> Occasionally you’ll want to write a class that is just a grouping of static methods and static fields. Such classes have acquired a bad reputation because some people abuse them to avoid thinking in terms of objects, but they do have valid uses. They can be used to group related methods on primitive values or arrays, in the manner of java.lang.Math or java.util.Arrays. They can also be used to group static methods, including factories (Item 1), for objects that implement some interface, in the manner of java.util.Collections. (As of Java 8, you can also put such methods in the interface, assuming it’s yours to modify.) Lastly, such classes can be used to group methods on a final class, since you can’t put them in a subclass.
    
有时候你想编写这样一个类，仅仅包含一组静态方法和静态字段。这样的类名声不佳，源于部分人对它们的滥用，导致没有以对象的方式进行思考，但它们确实有合理的用途。这种类可以用来组合对原生类型或数组类型的相关方法，就像java.lang.Math或java.util.Arrays；它们还可以用来组合静态方法，包括静态工厂方法（第1条），专门针对实现了某些接口的对象，就像java.util.Collections这样的类。（在Java8里，假定这些类是你的，可以修改，你还可以把这些方法写在接口中。）最后一点，这样的类还可以用来把方法组合到一个final修饰的类里，因为你不能把它们放在子类里。

>    Such utility classes were not designed to be instantiated: an instance would be nonsensical. In the absence of explicit constructors, however, the compiler provides a public, parameterless default constructor. To a user, this constructor is indistinguishable from any other. It is not uncommon to see unintentionally instantiable classes in published APIs.

这种辅助类被设计成不可实例化的：它们的实例没有意义。然而，假如缺少显式的构造方法，编译器会提供一个public修饰无参的默认构造方法，对于用户来说，这样的构造方法不能和任何其他构造方法区分开来，在已发布的API中，无意识地实例化类的情况并不罕见。

>    Attempting to enforce noninstantiability by making a class abstract does not work. The class can be subclassed and the subclass instantiated. Furthermore, it misleads the user into thinking the class was designed for inheritance (Item 19). There is, however, a simple idiom to ensure noninstantiability. A default constructor is generated only if a class contains no explicit constructors, so a class can be made noninstantiable by including a private constructor:
    
试图通过编写一个抽象类来强制使类不能被实例化的做法是行不通的。抽象类可以有子类，而子类能够被实例化。另外，抽象类会误导用户认为该类这样设计的目的是被继承使用（第19条）。然而，有一个简单的惯用做法来确保类不能被实例化，由于只有在没有显式构造方法的时候，编译器才会为类生成默认构造方法，因此在类里包含一个私有的构造方法，类就不能被实例化：

```java
// Noninstantiable utility class
// 不能实例化的辅助类
public class UtilityClass {
    // Suppress default constructor for noninstantiability
    // 抑制默认构造方法，使类不能实例化
private UtilityClass() {
        throw new AssertionError();
    }
    ... // Remainder omitted
}
```

>    Because the explicit constructor is private, it is inaccessible outside the class. The AssertionError isn’t strictly required, but it provides insurance in case the constructor is accidentally invoked from within the class. It guarantees the class will never be instantiated under any circumstances. This idiom is mildly counterintuitive because the constructor is provided expressly so that it cannot be invoked. It is therefore wise to include a comment, as shown earlier.

因为显式声明的构造方法是私有的，所以在类的外部不能访问。该方法中并不是严格要求需要抛出AssertionError异常，不过这样做为防止从类的内部意外调用构造方法提供了保障，这样就确保了在任何情况下，类都不能被实例化。这种惯用做法有点违反直觉，因为明确提供构造方法专门用于不能被调用，所以比较明智的做法是包含一些注释来说明情况，如前述代码所示。

>    As a side effect, this idiom also prevents the class from being subclassed. All constructors must invoke a superclass constructor, explicitly or implicitly, and a subclass would have no accessible superclass constructor to invoke.
    
这种做法有一个副作用，该类不能被子类继承使用。因为所有子类构造方法都必须显式或隐式地调用父类的构造方法，而该类的子类却不能访问父类的构造方法。
