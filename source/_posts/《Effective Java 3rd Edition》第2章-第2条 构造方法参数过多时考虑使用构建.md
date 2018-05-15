---
title: 《Effective Java 3rd Edition》第2章-第2条 构造方法参数过多时考虑使用构建 
date: 2018-05-15 16:33:01
categories:
    - Effective Java 3rd edition
tags: 
    - effective 
    - java 
    - builder 
    - constrauctor
---

# Item 2: Consider a builder when faced with many constructor parameters

# 第2条：构造方法参数过多时考虑使用构建器

<!-- more -->

> Static factories and constructors share a limitation: they do not scale well to large numbers of optional parameters. Consider the case of a class representing the Nutrition Facts label that appears on packaged foods. These labels have a few required fields—serving size, servings per container, and calories per serving—and more than twenty optional fields—total fat, saturated fat, trans fat, cholesterol, sodium, and so on. Most products have nonzero values for only a few of these optional fields.

静态工厂和构造方法都有一个限制：当参数数量较多且参数值可选的时候，其扩展性较差。假设有一个类用来表示包装食品上的营养成分标签，这些标签有一些必须的字段——大小，每包份数，每份包含的卡路里，此外还有二十多个可选字段——总脂肪，饱和脂肪，反式脂肪，胆固醇，钠，诸如此类。对于这些可选字段，大部分商品只会有很少几个非零值。

> What sort of constructors or static factories should you write for such a class? Traditionally, programmers have used the telescoping constructor pattern, in which you provide a constructor with only the required parameters, another with a single optional parameter, a third with two optional parameters, and so on, culminating in a constructor with all the optional parameters. Here’s how it looks in practice. For brevity’s sake, only four optional fields are shown:

编写这样的类，你会选择哪种类型的构造方法或静态工厂方法？按照传统的做法，码农们会使用一种伸缩构造模式，首先提供一个所有必需字段作为参数的构造方法，然后在此基础上添加一个可选字段作为第二个构造方法，再然后添加两个可选字段作为第三个构造方法，以此类推，不断累加直到所有可选字段都包含在构造方法中。下面是一个实际的例子，为了版面简洁，这里只展示了四个可选字段：

```java
// Telescoping constructor pattern - does not scale well!
// 伸缩构造模式——扩展性差

public class NutritionFacts {
    private final int servingSize;  // (mL)            required
    private final int servings;     // (per container) required
    private final int calories;     // (per serving)   optional
    private final int fat;          // (g/serving)     optional
    private final int sodium;       // (mg/serving)    optional
    private final int carbohydrate; // (g/serving)     optional

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSSize, int servings,
            int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings,
            int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings,
            int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }
	public NutritionFacts(int servingSize, int servings,
           int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize  = servingSize;
        this.servings     = servings;
        this.calories     = calories;
        this.fat          = fat;
        this.sodium       = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```

> When you want to create an instance, you use the constructor with the shortest parameter list containing all the parameters you want to set:

当你想要创建一个实例的时候，可以使用包含了所有要设置的参数的那个最短参数列表的构造方法：

```java
NutritionFacts cocaCola =new NutritionFacts(240, 8, 100, 0, 35, 27);
```

> Typically this constructor invocation will require many parameters that you don’t want to set, but you’re forced to pass a value for them anyway. In this case, we passed a value of 0 for fat. With “only” six parameters this may not seem so bad, but it quickly gets out of hand as the number of parameters increases.

一般来说，这样调用构造方法的时候，会有一些你不想赋值的参数，但你又不得不传入一个值，比如在上面的代码中，我们给fat参数传了一个0值。只有六个参数的时候，这种情况看起来还不太糟，但是等到参数数量不断增长，情况很快就失去控制了。

> In short, the telescoping constructor pattern works, but it is hard to write client code when there are many parameters, and harder still to read it. The reader is left wondering what all those values mean and must carefully count parameters to find out. Long sequences of identically typed parameters can cause subtle bugs. If the client accidentally reverses two such parameters, the compiler won’t complain, but the program will misbehave at runtime (Item 51).

简单来说，伸缩构造方法模式能解决问题，但是参数数量很多的时候，编写客户端代码就变得困难了，并且代码阅读起来更困难。读者得弄清楚所有这些值的含义，并且需要小心计数来识别每个参数。一长串类型相同的参数可能会引发不易察觉的错误，假如客户端不小心颠倒了两个这样的参数，编译器并不会报错，但是在运行期会得到不一样的结果（第51条）。

> A second alternative when you’re faced with many optional parameters in a constructor is the JavaBeans pattern, in which you call a parameterless constructor to create the object and then call setter methods to set each required parameter and each optional parameter of interest:

当构造方法中有许多可选参数的时候，另一个可以选择的方案是JavaBeans模式，这种模式下，首先调用无参构造方法创建一个对象，然后调用setter方法给每个必选参数赋值，并给每个感兴趣的可选参数赋值：

```java
// JavaBeans Pattern - allows inconsistency, mandates mutability
// JavaBeans模式——允许不一致性，可变性
public class NutritionFacts {
    // Parameters initialized to default values (if any)
    private int servingSize  = -1; // Required; no default value
    private int servings     = -1; // Required; no default value
    private int calories     = 0;
    private int fat          = 0;
    private int sodium       = 0;
    private int carbohydrate = 0;

    public NutritionFacts() { }
    // Setters
    public void setServingSize(int val)  { servingSize = val; }
    public void setServings(int val)    { servings = val; }
    public void setCalories(int val)    { calories = val; }
	public void setFat(int val)         { fat = val; }
    public void setSodium(int val)      { sodium = val; }
    public void setCarbohydrate(int val) { carbohydrate = val; }
}
```

> This pattern has none of the disadvantages of the telescoping constructor pattern. It is easy, if a bit wordy, to create instances, and easy to read the resulting code:
 
这种模式没有伸缩构造方法的缺陷，用这种模式创建实例比较简单，虽然有一点点啰嗦，目标代码也易于阅读：

```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

> Unfortunately, the JavaBeans pattern has serious disadvantages of its own. Because construction is split across multiple calls, a JavaBean may be in an inconsistent state partway through its construction. The class does not have the option of enforcing consistency merely by checking the validity of the constructor parameters. Attempting to use an object when it’s in an inconsistent state may cause failures that are far removed from the code containing the bug and hence difficult to debug. A related disadvantage is that the JavaBeans pattern precludes the possibility of making a class immutable (Item 17) and requires added effort on the part of the programmer to ensure thread safety.

不过，JavaBeans也有它本身严重的问题，由于对象构造过程被拆分成了多次调用，一个JavaBean在创建过程中可能处于一种中间的不一致状态。仅仅通过检查构造器参数的有效性，类也不能强制使对象处于一致性状态。使用一个处于非一致性状态下的对象可能引起错误，而程序失败的地方，和包含错误的代码相隔太远，以至于难以进行调试。JavaBeans模式另一个相关的问题是，使用该模式没法构建不可变类，并且在多线程环境下程序员必须加倍小心以确保线程安全。

> It is possible to reduce these disadvantages by manually “freezing” the object when its construction is complete and not allowing it to be used until frozen, but this variant is unwieldy and rarely used in practice.

有一个方法可以缓解这种问题，在构造方法调用完成时，把对象手工冻结起来， 直到解冻时才允许使用，但是这种变体比较怪异，在实践中很少使用。

> Moreover, it can cause errors at runtime because the compiler cannot ensure that the programmer calls the freeze method on an object before using it.

另外，由于编译器没法确保程序员在使用对象之前调用冻结方法，这样就可能在运行期引发错误。

> Luckily, there is a third alternative that combines the safety of the telescoping constructor pattern with the readability of the JavaBeans pattern. It is a form of the Builder pattern [Gamma95]. Instead of making the desired object directly, the client calls a constructor (or static factory) with all of the required parameters and gets a builder object. Then the client calls setter-like methods on the builder object to set each optional parameter of interest. Finally, the client calls a parameterless build method to generate the object, which is typically immutable. The builder is typically a static member class (Item 24) of the class it builds. Here’s how it looks in practice:

幸运的是，还有第三种方案可供选择，这种方案结合了伸缩构造模式的安全性和JavaBeans模式的可读性，是构建器模式的一种。和直接创建所需对象不同，客户端首先传入所有必需参数调用一个构造方法（或者静态工厂方法），获得一个构建器，然后调用构建器上类似于setter的方法，给每个感兴趣的参数赋值，最后调用构建器的无参build方法生成对象，这个对象通常是不可变的。构造器通常是它要构造的类的一个静态成员类。在实践中看起来是这样的：

```java
// 构建器模式
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // Required parameters
        // 必需的参数
        private final int servingSize;
        private final int servings;

        // Optional parameters - initialized to default values
        private int calories      = 0;
        private int fat           = 0;
        private int sodium        = 0;
        private int carbohydrate  = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings    = servings;
        }

        public Builder calories(int val)
            { calories = val;      return this; }
        public Builder fat(int val)
            { fat = val;           return this; }
        public Builder sodium(int val)
            { sodium = val;        return this; }
        public Builder carbohydrate(int val)
            { carbohydrate = val;  return this; }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }
   private NutritionFacts(Builder builder) {
        servingSize  = builder.servingSize;
        servings     = builder.servings;
        calories     = builder.calories;
        fat          = builder.fat;
        sodium       = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```

> The NutritionFacts class is immutable, and all parameter default values are in one place. The builder’s setter methods return the builder itself so that invocations can be chained, resulting in a fluent API. Here’s how the client code looks:

NutritionFacts类是不可变的，所有默认值都在同一个地方设置。构造器的setter方法返回构造器对象本身，因此可以被链式调用，形成一个流式API。客户端代码看起来是这样的：

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
        .calories(100).sodium(35).carbohydrate(27).build();
```

> This client code is easy to write and, more importantly, easy to read. The Builder pattern simulates named optional parameters as found in Python and Scala.

这样的代码易于编写，更重要的是，易于阅读。构建器模式模仿的是Python和Scala语言中的命名可选参数特性。

> Validity checks were omitted for brevity. To detect invalid parameters as soon as possible, check parameter validity in the builder’s constructor and methods. Check invariants involving multiple parameters in the constructor invoked by the build method. To ensure these invariants against attack, do the checks on object fields after copying parameters from the builder (Item 50). If a check fails, throw an IllegalArgumentException (Item 72) whose detail message indicates which parameters are invalid (Item 75).

为了保持简洁，此处省略了有效性检查。为尽早检测到无效参数，可以在构建器的构造方法和setter方法中做参数有效性检查，在build方法中调用的构造方法中做涉及到多个参数的不变性检查。为了确保这些不变性不会受到破坏，从构建器复制参数到对象后，需要对对象字段做检查。一旦检查失败，就抛出一个IllegalArgumentException异常，该异常的详细消息表明了哪个参数是无效的。

> The Builder pattern is well suited to class hierarchies. Use a parallel hierarchy of builders, each nested in the corresponding class. Abstract classes have abstract builders; concrete classes have concrete builders. For example, consider an abstract class at the root of a hierarchy representing various kinds of pizza:

对于具有层次关系的类来说，构建器模式能进行很好的适配。可以使用一个平行层次结构的构建器，每个构建器嵌套在对应的类里。抽象类使用抽象构建器，具体类使用具体的构建器。举个例子，假设有一个抽象的根类，代表各种各样的披萨：

```java
// Builder pattern for class hierarchies
public abstract class Pizza {
   public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
   final Set<Topping> toppings;

   abstract static class Builder<T extends Builder<T>> {
      EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
      public T addTopping(Topping topping) {
         toppings.add(Objects.requireNonNull(topping));
         return self();
      }

      abstract Pizza build();
      // Subclasses must override this method to return "this"
      protected abstract T self();
   }
   Pizza(Builder<?> builder) {
      toppings = builder.toppings.clone(); // See Item  50
   }
}
```

> Note that Pizza.Builder is a generic type with a recursive type parameter (Item 30). This, along with the abstract self method, allows method chaining to work properly in subclasses, without the need for casts. This workaround for the fact that Java lacks a self type is known as the simulated self-type idiom.

值得注意的是，Pizza.Builder类是带有一个递归类型参数的泛型类（第30条），配合上抽象的self方法，可以让子类进行链式方法调用，而不需要做强制转换。由于java语言缺乏self类型，这是一种解决方法，这种方法被称为模拟self类型。

> Here are two concrete subclasses of Pizza, one of which represents a standard New-York-style pizza, the other a calzone. The former has a required size parameter, while the latter lets you specify whether sauce should be inside or out:
 
下面是两个Pizza类的具体子类，其中一个用来表示标准的纽约式披萨，另一个表示半圆形烤披萨，前者有一个必需的参数：大小，后者有一个必需的参数：披萨中是否带酱汁。

```java
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override public NyPizza build() {
            return new NyPizza(this);
        }

        @Override protected Builder self() { return this; }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}

public class Calzone extends Pizza     private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false; // Default

        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }

        @Override public Calzone build() {
            return new Calzone(this);
        }

        @Override protected Builder self() { return this; }
    }
    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }
}
```

> Note that the build method in each subclass’s builder is declared to return the correct subclass: the build method of NyPizza.Builder returns NyPizza, while the one in Calzone.Builder returns Calzone. This technique, wherein a subclass method is declared to return a subtype of the return type declared in the super-class, is known as covariant return typing. It allows clients to use these builders without the need for casting.

需要注意的是，每个子类构建器的build方法都声明返回适当的子类类型：NyPizza.Builder类的build方法返回NyPizza类型，而Calzone.Builder类则返回Calzone类型。这种技术被称为协变，指的是子类方法可以返回父类方法定义中返回类型的子类型，客户端使用这些构建器的时候不需要做强制转换。

> The client code for these “hierarchical builders” is essentially identical to the code for the simple NutritionFacts builder. The example client code shown next assumes static imports on enum constants for brevity:

调用层次结构构建器的客户端代码，和前面调用简单结构NutritionFacts构造器代码是一样的。接下来展示的客户端代码，为了版面简洁，假定已经静态导入了枚举常量：

```java
NyPizza pizza = new NyPizza.Builder(SMALL)
        .addTopping(SAUSAGE).addTopping(ONION).build();
Calzone calzone = new Calzone.Builder()
        .addTopping(HAM).sauceInside().build();
```

> A minor advantage of builders over constructors is that builders can have multiple varargs parameters because each parameter is specified in its own method. Alternatively, builders can aggregate the parameters passed into multiple calls to a method into a single field, as demonstrated in the addTopping method earlier.

和构造方法相比，构建器模式另一个次要点的优势是，构建器可以给多个不同参数赋值，因为每个参数通过各自的方法进行赋值。另外，构建器还可以把多次调用一个方法的参数聚合到一个单一的字段中，就像前面代码中的addTopping方法所演示的那样。

> The Builder pattern is quite flexible. A single builder can be used repeatedly to build multiple objects. The parameters of the builder can be tweaked between invocations of the build method to vary the objects that are created. A builder can fill in some fields automatically upon object creation, such as a serial number that increases each time an object is created.

构建器模式非常灵活，可以重复使用单个构建器来创建多个对象。调节构建器参数的方法很多，从build方法调用，到改变要创建的对象。在创建对象的时候，可以自动把构建器填充到某些字段中，比如，在每次创建对象的时候递增序列号。

> The Builder pattern has disadvantages as well. In order to create an object, you must first create its builder. While the cost of creating this builder is unlikely to be noticeable in practice, it could be a problem in performance-critical situations. Also, the Builder pattern is more verbose than the telescoping constructor pattern, so it should be used only if there are enough parameters to make it worthwhile, say four or more. But keep in mind that you may want to add more parameters in the future. But if you start out with constructors or static factories and switch to a builder when the class evolves to the point where the number of parameters gets out of hand, the obsolete constructors or static factories will stick out like a sore thumb. Therefore, it’s often better to start with a builder in the first place.

构建器模式也有它的缺点。为了创建对象，必须先创建它的构建器，虽然在实际使用时，创建构建器的成本不太引人注目，但在比较注重性能的场景中，这可能会成为问题。另外，和伸缩构造方法比起来，构建器模式要显得冗长很多，所以参数较多的时候使用它才有价值，也就是说，有四个以上参数的时候。但是应该时刻记住，类在将来可能会添加更多参数。如果一开始使用的是构造方法或静态工厂方法，等到类的参数数量发展到失去控制，此时切换到构建器模式，废弃的构造方法和静态工厂方法就像疼痛的大拇指一样引人注目。因此，通常更好的做法是一开始就使用构建器。

> In summary, the Builder pattern is a good choice when designing classes whose constructors or static factories would have more than a handful of parameters, especially if many of the parameters are optional or of identical type. Client code is much easier to read and write with builders than with telescoping constructors, and builders are much safer than JavaBeans.

总结：在设计类的时候，如果其构造方法或静态工厂方法拥有大量参数，使用构建器模式是个不错的选择，尤其是当很多参数值是可选的，或者很多参数类型相同的时候。相比于伸缩构造方法，使用构建器模式的客户端代码更容易阅读和编写，并且比JavaBeans模式更安全。
