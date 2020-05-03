# [OOP六大设计原则](https://www.cnblogs.com/pony1223/p/7594803.html)

首先我们看下各个模式之间的关系图，下面这张图是网上比较典型的一个类图关系：

​           ![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170926072840714-1631147313.jpg)

 

从上面的类图之间可以看出，学习设计模式或者说学懂完全理解所有的设计模式还是挺难的，只能说不断的重复学习，不断的去领悟才是唯一的方法，当然不排除有些人是天才看一篇就学会了，可惜鄙人不是，所以必须不断重复学习来加深自己的理解。个人感觉，单例、工厂、装饰者、观察者、代理模式使用的频率比较高；当然不是说其他模糊就不使用，只是个人的看法而已，o(*￣︶￣*)o。

学习设计模式，首先要学习的就是设计原则，因此我从设计原则来开始第一个节。

**一、设计原则**                                                                         

## **1.单一职责**                                                                           

一个类，只有一个引起它变化的原因。应该只有一个职责。每一个职责都是变化的一个轴线，如果一个类有一个以上的职责，这些职责就耦合在了一起。这会导致脆弱的设计。当一个职责发生变化时，可能会影响其它的职责。另外，多个职责耦合在一起，会影响复用性。例如：要实现逻辑和界面的分离。

简单通俗的来说：一个类只负责一项职责。

**问题：**比如一个类T负责两个不同的职责：职责P1，职责P2。当由于职责P1需求发生改变而需要修改类T时，有可能会导致原本运行正常的职责P2功能发生故障。

解决方法：遵循单一职责原则。分别建立两个类T1、T2，使T1完成职责P1功能，T2完成职责P2功能。这样，当修改类T1时，不会使职责P2发生故障风险；同理，当修改T2时，也不会使职责P1发生故障风险。

扩展：说到单一职责原则，其实很多人不知不觉的都在使用，即使没有学习过设计模式的人，或者没有听过单一职责原则这个概念的人也会自觉的遵守这个重要原则，因为这是一个常识，比如你去在原有的项目上开发一个新的业务功能的时候，你肯定是会从新建立一个类，来实现一个新的功能，肯定不会基于原有的A功能身上直接写B业务的功能，肯定一般都是会新写一个类来实现B功能。在软件编程中，谁也不希望因为修改了一个功能导致其他的功能发生故障。而避免出现这一问题的方法便是遵循单一职责原则。虽然单一职责原则如此简单，并且被认为是常识，**但是**即便是经验丰富的程序员写出的程序，也会有违背这一原则的代码存在。为什么会出现这种现象呢？**因为有职责扩散。所谓职责扩散，就是因为某种原因，职责P被分化为粒度更细的职责P1和P2。**

**比如：类T只负责一个职责P，这样设计是符合单一职责原则的。后来由于某种原因，也许是需求变更了，也许是程序的设计者境界提高了，需要将职责P细分为粒度更细的职责P1，P2，这时如果要使程序遵循单一职责原则，需要将类T也分解为两个类T1和T2，分别负责P1、P2两个职责。但是在程序已经写好的情况下，这样做简直太费时间了。所以，简单的修改类T，用它来负责两个职责是一个比较不错的选择，虽然这样做有悖于单一职责原则。（这样做的风险在于职责扩散的不确定性，因为我们不会想到这个职责P，在未来可能会扩散为P1，P2，P3，P4……Pn。所以记住，在职责扩散到我们无法控制的程度之前，立刻对代码进行重构。）**

举例说明，用一个类描述动物呼吸这个场景：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
class Animal{
    public void breathe(String animal){
        System.out.println(animal+"呼吸空气");
    }
}
public class Client{
    public static void main(String[] args){
        Animal animal = new Animal();
        animal.breathe("牛");
        animal.breathe("羊");
        animal.breathe("猪");
    }
} 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

运行结果：

牛呼吸空气

羊呼吸空气

猪呼吸空气

程序上线后，发现问题了，并不是所有的动物都呼吸空气的，比如鱼就是呼吸水的。修改时如果遵循单一职责原则，需要将Animal类细分为陆生动物类Terrestrial，水生动物Aquatic，代码如下：

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
class Terrestrial{
    public void breathe(String animal){
        System.out.println(animal+"呼吸空气");
    }
}
class Aquatic{
    public void breathe(String animal){
        System.out.println(animal+"呼吸水");
    }
}

public class Client{
    public static void main(String[] args){
        Terrestrial terrestrial = new Terrestrial();
        terrestrial.breathe("牛");
        terrestrial.breathe("羊");
        terrestrial.breathe("猪");
        
        Aquatic aquatic = new Aquatic();
        aquatic.breathe("鱼");
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

运行结果：

牛呼吸空气

羊呼吸空气

猪呼吸空气

鱼呼吸水

我们会发现如果这样修改花销是很大的，除了将原来的类分解之外，还需要修改客户端。而直接修改类Animal来达成目的虽然违背了单一职责原则，但花销却小的多，代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
class Animal{
    public void breathe(String animal){
        if("鱼".equals(animal)){
            System.out.println(animal+"呼吸水");
        }else{
            System.out.println(animal+"呼吸空气");
        }
    }
}

public class Client{
    public static void main(String[] args){
        Animal animal = new Animal();
        animal.breathe("牛");
        animal.breathe("羊");
        animal.breathe("猪");
        animal.breathe("鱼");
    }
} 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

可以看到，这种修改方式要简单的多。但是却存在着隐患：有一天需要将鱼分为呼吸淡水的鱼和呼吸海水的鱼，则又需要修改Animal类的breathe方法，而对原有代码的修改会对调用“猪”“牛”“羊”等相关功能带来风险，也许某一天你会发现程序运行的结果变为“牛呼吸水”了。这种修改方式直接在代码级别上违背了单一职责原则，虽然修改起来最简单，但隐患却是最大的。还有一种修改方式：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
class Animal{
    public void breathe(String animal){
        System.out.println(animal+"呼吸空气");
    }

    public void breathe2(String animal){
        System.out.println(animal+"呼吸水");
    }
}

public class Client{
    public static void main(String[] args){
        Animal animal = new Animal();
        animal.breathe("牛");
        animal.breathe("羊");
        animal.breathe("猪");
        animal.breathe2("鱼");
    }
} 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

可以看到，这种修改方式没有改动原来的方法，而是在类中新加了一个方法，这样虽然也违背了单一职责原则，但在方法级别上却是符合单一职责原则的，因为它并没有动原来方法的代码。**这三种方式各有优缺点，那么在实际编程中，采用哪一中呢？其实这真的比较难说，需要根据实际情况来确定。我的原则是：只有逻辑足够简单，才可以在代码级别上违反单一职责原则；只有类中方法数量足够少，才可以在方法级别上违反单一职责原则；**

例如本文所举的这个例子，它太简单了，它只有一个方法，所以，无论是在代码级别上违反单一职责原则，还是在方法级别上违反，都不会造成太大的影响。实际应用中的类都要复杂的多，一旦发生职责扩散而需要修改类时，除非这个类本身非常简单，否则还是遵循单一职责原则的好。

**遵循单一职责原的优点有：**

- 可以降低类的复杂度，一个类只负责一项职责，其逻辑肯定要比负责多项职责简单的多；
- 提高类的可读性，提高系统的可维护性；
- 变更引起的风险降低，变更是必然的，如果单一职责原则遵守的好，当修改一个功能时，可以显著降低对其他功能的影响。

需要说明的一点是单一职责原则不只是面向对象编程思想所特有的，只要是模块化的程序设计，都适用单一职责原则。

单一职责看似简单，实际上在实际运用过程中，会发现真的会出现很多职责扩展的现象，这个时候采用直接违反还会方法上遵循还是完全遵循单一职责原则还是取决于当前业务开发的人员的技能水平和这个需求的时间，如果技能水平不足，肯定会简单的if else 去解决，不会想什么原则，直接实现功能就好了，这也是为什么在很多小公司会发现代码都是业务堆起来的，当然也有好的小公司代码是写的好的，这个也是不可否认的。不过不管采用什么方式解决，心中至少要知道有几种解决方法。

 

## 2.里氏替换原则 （Liskov Substitution Principle）                              

里氏代换原则(Liskov Substitution Principle LSP)面向对象设计的基本原则之一。 里氏代换原则中说，任何基类可以出现的地方，子类一定可以出现。 LSP是继承复用的基石，只有当衍生类可以替换掉基类，软件单位的功能不受到影响时，基类才能真正被复用，而衍生类也能够在基类的基础上增加新的行为。里氏代换原则是对“开-闭”原则的补充。实现“开-闭”原则的关键步骤就是抽象化。而基类与子类的继承关系就是抽象化的具体实现，所以里氏代换原则是对实现抽象化的具体步骤的规范。

看完上面的概念估计很多人都和我一样不是太理解，或者比较好奇，为什么叫里氏替换？其原因是：这项原则最早是在1988年，由麻省理工学院的一位姓里的女士（Barbara Liskov）提出来的。

解剖为下面的描述:

定义1：如果对每一个类型为 T1的对象 o1，都有类型为 T2 的对象o2，使得以 T1定义的所有程序 P 在所有的对象 o1 都代换成 o2 时，程序 P 的行为没有发生变化，那么类型 T2 是类型 T1 的子类型。

定义2：所有引用基类的地方必须能透明地使用其子类的对象。

通俗简单的说就是：子类可以扩展父类的功能，但不能改变父类原有的功能。

**问题由来：**有一功能P1，由类A完成。现需要将功能P1进行扩展，扩展后的功能为P，其中P由原有功能P1与新功能P2组成。新功能P由类A的子类B来完成，则子类B在完成新功能P2的同时，有可能会导致原有功能P1发生故障。

解决方案：**当使用继承时，遵循里氏替换原则。类B继承类A时，除添加新的方法完成新增功能P2外，尽量不要重写父类A的方法，也尽量不要重载父类A的方法。【由时候我们可以采用final的手段强制来遵循】**

继承包含这样一层含义：父类中凡是已经实现好的方法（相对于抽象方法而言），实际上是在设定一系列的规范和契约，虽然它不强制要求所有的子类必须遵从这些契约，但是如果子类对这些非抽象方法任意修改，就会对整个继承体系造成破坏。而里氏替换原则就是表达了这一层含义。

继承作为面向对象三大特性之一，在给程序设计带来巨大便利的同时，也带来了弊端。比如使用继承会给程序带来侵入性，程序的可移植性降低，增加了对象间的耦合性，如果一个类被其他的类所继承，则当这个类需要修改时，必须考虑到所有的子类，并且父类修改后，所有涉及到子类的功能都有可能会产生故障。

举例说明继承的风险，我们需要完成一个两数相减的功能，由类A来负责。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
class A{
    public int func1(int a, int b){
        return a-b;
    }
}

public class Client{
    public static void main(String[] args){
        A a = new A();
        System.out.println("100-50="+a.func1(100, 50));
        System.out.println("100-80="+a.func1(100, 80));
    }
} 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

运行结果：

100-50=50

100-80=20

后来，我们需要增加一个新的功能：完成两数相加，然后再与100求和，由类B来负责。即类B需要完成两个功能：

- 两数相减。
- 两数相加，然后再加100。

由于类A已经实现了第一个功能【两数相减】，所以类B继承类A后，只需要再完成第二个功能【两数相加，然后再加100】就可以了，代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
class B extends A{
    public int func1(int a, int b){
        return a+b;
    }
    
    public int func2(int a, int b){
        return func1(a,b)+100;
    }
}

public class Client{
    public static void main(String[] args){
        B b = new B();
        System.out.println("100-50="+b.func1(100, 50));
        System.out.println("100-80="+b.func1(100, 80));
        System.out.println("100+20+100="+b.func2(100, 20));
    }
} 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

类B完成后，运行结果：

100-50=150

100-80=180

100+20+100=220

我们发现原本运行正常的相减功能发生了错误。**原因就是类B在给方法起名时无意中重写了父类的方法**，造成所有运行相减功能的代码全部调用了类B重写后的方法，造成原本运行正常的功能出现了错误。在本例中，**引用基类A完成的功能，换成子类B之后，发生了异常**。**在实际编程中，我们常常会通过重写父类的方法来完成新的功能，这样写起来虽然简单，但是整个继承体系的可复用性会比较差，特别是运用多态比较频繁时，程序运行出错的几率非常大。如果非要重写父类的方法，比较通用的做法是：原来的父类和子类都继承一个更通俗的基类，原有的继承关系去掉，采用依赖、聚合，组合等关系代替。**

再次来理解里氏替换原则：子类可以扩展父类的功能，但不能改变父类原有的功能。它包含以下4层含义：

- 子类可以实现父类的抽象方法，但不能覆盖父类的非抽象方法。
- 子类中可以增加自己特有的方法。
- 当子类的方法重载父类的方法时，方法的前置条件（即方法的形参）要比父类方法的输入参数更宽松。【注意区分重载和重写】
- 当子类的方法实现父类的抽象方法时，方法的后置条件（即方法的返回值）要比父类更严格。

看上去很不可思议，因为我们会发现在自己编程中常常会违反里氏替换原则，程序照样跑的好好的。所以大家都会产生这样的疑问，假如我非要不遵循里氏替换原则会有什么后果？

后果就是：你写的代码出问题的几率将会大大增加。

##  3.依赖倒置原则 （Dependence Inversion Principle）

所谓依赖倒置原则（Dependence Inversion Principle）就是要依赖于抽象，不要依赖于具体。实现开闭原则的关键是抽象化，并且从抽象化导出具体化实现，如果说开闭原则是面向对象设计的目标的话，那么依赖倒转原则就是面向对象设计的主要手段。

定义：高层模块不应该依赖低层模块，二者都应该依赖其抽象；抽象不应该依赖细节；细节应该依赖抽象。

通俗点说：要求对抽象进行编程，不要对实现进行编程，这样就降低了客户与实现模块间的耦合。

**问题由来：**类A直接依赖类B，假如要将类A改为依赖类C，则必须通过修改类A的代码来达成。这种场景下，类A一般是高层模块，负责复杂的业务逻辑；类B和类C是低层模块，负责基本的原子操作；假如修改类A，会给程序带来不必要的风险。

解决方案：将类A修改为依赖接口I，类B和类C各自实现接口I，类A通过接口I间接与类B或者类C发生联系，则会大大降低修改类A的几率。

依赖倒置原则基于这样一个事实：相对于细节的多变性，抽象的东西要稳定的多。以抽象为基础搭建起来的架构比以细节为基础搭建起来的架构要稳定的多。在java中，抽象指的是接口或者抽象类，细节就是具体的实现类，**使用接口或者抽象类的目的是制定好规范和契约，而不去涉及任何具体的操作，把展现细节的任务交给他们的实现类去完成。**

**依赖倒置原则的核心思想是面向接口编程，**我们依旧用一个例子来说明面向接口编程比相对于面向实现编程好在什么地方。场景是这样的，母亲给孩子讲故事，只要给她一本书，她就可以照着书给孩子讲故事了。代码如下：

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
class Book{
    public String getContent(){
        return "很久很久以前有一个阿拉伯的故事……";
    }
}

class Mother{
    public void narrate(Book book){
        System.out.println("妈妈开始讲故事");
        System.out.println(book.getContent());
    }
}

public class Client{
    public static void main(String[] args){
        Mother mother = new Mother();
        mother.narrate(new Book());
    }
} 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

运行结果：

妈妈开始讲故事

很久很久以前有一个阿拉伯的故事……

上述是面向实现的编程，即依赖的是Book这个具体的实现类；看起来功能都很OK，也没有什么问题。

运行良好，假如有一天，需求变成这样：不是给书而是给一份报纸，让这位母亲讲一下报纸上的故事，报纸的代码如下：

```
class Newspaper{
    public String getContent(){
        return "林书豪38+7领导尼克斯击败湖人……";
    }
}
```

这位母亲却办不到，因为她居然不会读报纸上的故事，这太荒唐了，只是将书换成报纸，居然必须要修改Mother才能读。假如以后需求换成杂志呢？换成网页呢？还要不断地修改Mother，这显然不是好的设计。原因就是Mother与Book之间的耦合性太高了，必须降低他们之间的耦合度才行。

我们引入一个抽象的接口IReader。读物，只要是带字的都属于读物：

```
interface IReader{
    public String getContent();
} 
```

Mother类与接口IReader发生依赖关系，而Book和Newspaper都属于读物的范畴，他们各自都去实现IReader接口，这样就符合依赖倒置原则了，代码修改为：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
class Newspaper implements IReader {
    public String getContent(){
        return "林书豪17+9助尼克斯击败老鹰……";
    }
}
class Book implements IReader{
    public String getContent(){
        return "很久很久以前有一个阿拉伯的故事……";
    }
}

class Mother{
    public void narrate(IReader reader){
        System.out.println("妈妈开始讲故事");
        System.out.println(reader.getContent());
    }
}

public class Client{
    public static void main(String[] args){
        Mother mother = new Mother();
        mother.narrate(new Book());
        mother.narrate(new Newspaper());
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

运行结果：

妈妈开始讲故事

很久很久以前有一个阿拉伯的故事……

妈妈开始讲故事

林书豪17+9助尼克斯击败老鹰……

这样修改后，无论以后怎样扩展Client类，都不需要再修改Mother类了。这只是一个简单的例子，实际情况中，代表高层模块的Mother类将负责完成主要的业务逻辑，一旦需要对它进行修改，引入错误的风险极大。所以遵循依赖倒置原则可以降低类之间的耦合性，提高系统的稳定性，降低修改程序造成的风险。

采用依赖倒置原则给多人并行开发带来了极大的便利，比如上例中，原本Mother类与Book类直接耦合时，Mother类必须等Book类编码完成后才可以进行编码，因为Mother类依赖于Book类。修改后的程序则可以同时开工，互不影响，因为Mother与Book类一点关系也没有。参与协作开发的人越多、项目越庞大，采用依赖导致原则的意义就越重大。现在很流行的TDD开发模式就是依赖倒置原则最成功的应用。

传递依赖关系有三种方式，以上的例子中使用的方法是接口传递，另外还有两种传递方式：构造方法传递和setter方法传递，相信用过Spring框架的，对依赖的传递方式一定不会陌生。

**在实际编程中，我们一般需要做到如下3点：**

- **低层模块尽量都要有抽象类或接口，或者两者都有。【可能会被人用到的】**
- **变量的声明类型尽量是抽象类或接口。**
- **使用继承时遵循里氏替换原则。**

**依赖倒置原则的核心就是要我们面向接口编程，理解了面向接口编程，也就理解了依赖倒置。**

## 4.接口隔离原则 （Interface Segregation Principle）        

其原则字面的意思是：使用多个隔离的接口，比使用单个接口要好。本意降低类之间的耦合度，而设计模式就是一个软件的设计思想，从大型软件架构出发，为了升级和维护方便。所以上文中多次出现：降低依赖，降低耦合。

原定义：客户端不应该依赖它不需要的接口；一个类对另一个类的依赖应该建立在最小的接口上。 

问题由来：类A通过接口I依赖类B，类C通过接口I依赖类D，如果接口I对于类A和类B来说不是最小接口，则类B和类D必须去实现他们不需要的方法。

**解决方案：将臃肿的接口I拆分为独立的几个接口，类A和类C分别与他们需要的接口建立依赖关系。也就是采用接口隔离原则。**

举例来说明接口隔离原则：

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170927214256294-478349580.png)

 

上图就没有实现接口隔离，类B 和 类 D 中都会实现不是自己的方法。

具体来说：类A依赖接口I中的方法1、方法2、方法3，类B是对类A依赖的实现。类C依赖接口I中的方法1、方法4、方法5，类D是对类C依赖的实现。对于类B和类D来说，虽然他们都存在着用不到的方法（也就是图中红色字体标记的方法），但由于实现了接口I，所以也必须要实现这些用不到的方法。对类图不熟悉的可以参照程序代码来理解，代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
interface I {
    public void method1();
    public void method2();
    public void method3();
    public void method4();
    public void method5();
}

class A{
    public void depend1(I i){
        i.method1();
    }
    public void depend2(I i){
        i.method2();
    }
    public void depend3(I i){
        i.method3();
    }
}

class B implements I{
    public void method1() {
        System.out.println("类B实现接口I的方法1");
    }
    public void method2() {
        System.out.println("类B实现接口I的方法2");
    }
    public void method3() {
        System.out.println("类B实现接口I的方法3");
    }
    //对于类B来说，method4和method5不是必需的，但是由于接口A中有这两个方法，
    //所以在实现过程中即使这两个方法的方法体为空，也要将这两个没有作用的方法进行实现。
    public void method4() {}
    public void method5() {}
}

class C{
    public void depend1(I i){
        i.method1();
    }
    public void depend2(I i){
        i.method4();
    }
    public void depend3(I i){
        i.method5();
    }
}

class D implements I{
    public void method1() {
        System.out.println("类D实现接口I的方法1");
    }
    //对于类D来说，method2和method3不是必需的，但是由于接口A中有这两个方法，
    //所以在实现过程中即使这两个方法的方法体为空，也要将这两个没有作用的方法进行实现。
    public void method2() {}
    public void method3() {}

    public void method4() {
        System.out.println("类D实现接口I的方法4");
    }
    public void method5() {
        System.out.println("类D实现接口I的方法5");
    }
}

public class Client{
    public static void main(String[] args){
        A a = new A();
        a.depend1(new B());
        a.depend2(new B());
        a.depend3(new B());
        
        C c = new C();
        c.depend1(new D());
        c.depend2(new D());
        c.depend3(new D());
    }
} 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

可以看到，如果接口过于臃肿，只要接口中出现的方法，不管对依赖于它的类有没有用处，实现类中都必须去实现这些方法，这显然不是好的设计。如果将这个设计修改为符合接口隔离原则，就必须对接口I进行拆分。在这里我们将原有的接口I拆分为三个接口，拆分后的设计如图2所示：

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170927214612559-854563464.png)

 

上述为遵循接口隔离原则的设计，代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
interface I1 {
    public void method1();
}

interface I2 {
    public void method2();
    public void method3();
}

interface I3 {
    public void method4();
    public void method5();
}

class A{
    public void depend1(I1 i){
        i.method1();
    }
    public void depend2(I2 i){
        i.method2();
    }
    public void depend3(I2 i){
        i.method3();
    }
}

class B implements I1, I2{
    public void method1() {
        System.out.println("类B实现接口I1的方法1");
    }
    public void method2() {
        System.out.println("类B实现接口I2的方法2");
    }
    public void method3() {
        System.out.println("类B实现接口I2的方法3");
    }
}

class C{
    public void depend1(I1 i){
        i.method1();
    }
    public void depend2(I3 i){
        i.method4();
    }
    public void depend3(I3 i){
        i.method5();
    }
}

class D implements I1, I3{
    public void method1() {
        System.out.println("类D实现接口I1的方法1");
    }
    public void method4() {
        System.out.println("类D实现接口I3的方法4");
    }
    public void method5() {
        System.out.println("类D实现接口I3的方法5");
    }
} 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

**接口隔离原则的含义是：建立单一接口，不要建立庞大臃肿的接口，尽量细化接口，接口中的方法尽量少**。也就是说，我们要为各个类建立专用的接口，而不要试图去建立一个很庞大的接口供所有依赖它的类去调用。本文例子中，将一个庞大的接口变更为3个专用的接口所采用的就是接口隔离原则。在程序设计中，依赖几个专用的接口要比依赖一个综合的接口更灵活。接口是设计时对外部设定的“契约”，通过分散定义多个接口，可以预防外来变更的扩散，提高系统的灵活性和可维护性。

说到这里，很多人会觉的接口隔离原则跟之前的单一职责原则很相似，其实不然。**其一，单一职责原则原注重的是职责；而接口隔离原则注重对接口依赖的隔离。其二，单一职责原则主要是约束类，其次才是接口和方法，它针对的是程序中的实现和细节；而接口隔离原则主要约束接口接口，主要针对抽象，针对程序整体框架的构建。**

采用接口隔离原则对接口进行约束时，要注意以下几点：

- 接口尽量小，但是要有限度。对接口进行细化可以提高程序设计灵活性是不挣的事实，但是如果过小，则会造成接口数量过多，使设计复杂化。所以一定要适度。
- 为依赖接口的类定制服务，只暴露给调用的类它需要的方法，它不需要的方法则隐藏起来。只有专注地为一个模块提供定制服务，才能建立最小的依赖关系。
- 提高内聚，减少对外交互。使接口用最少的方法去完成最多的事情。

运用接口隔离原则，一定要适度，接口设计的过大或过小都不好。设计接口的时候，只有多花些时间去思考和筹划，才能准确地实践这一原则。

 

## 5.迪米特法则（最少知道原则） （Demeter Principle）

为什么叫最少知道原则，就是说：一个实体应当尽量少的与其他实体之间发生相互作用，使得系统功能模块相对独立。也就是说一个软件实体应当尽可能少的与其他实体发生相互作用。这样，当一个模块修改时，就会尽量少的影响其他的模块，扩展会相对容易，这是对软件实体之间通信的限制，它要求限制软件实体之间通信的宽度和深度。

**定义：一个对象应该对其他对象保持最少的了解。**

问题由来：类与类之间的关系越密切，耦合度越大，当一个类发生改变时，对另一个类的影响也越大。

解决方案：尽量降低类与类之间的耦合。

自从我们接触编程开始，就知道了软件编程的总的原则：低耦合，高内聚。无论是面向过程编程还是面向对象编程，只有使各个模块之间的耦合尽量的低，才能提高代码的复用率。低耦合的优点不言而喻，但是怎么样编程才能做到低耦合呢？那正是迪米特法则要去完成的。

迪米特法则又叫最少知道原则，最早是在1987年由美国Northeastern University的Ian Holland提出。**通俗的来讲，就是一个类对自己依赖的类知道的越少越好。也就是说，对于被依赖的类来说，无论逻辑多么复杂，都尽量地的将逻辑封装在类的内部，对外除了提供的public方法，不对外泄漏任何信息。**迪米特法则还有一个更简单的定义：只与直接的朋友通信。首先来解释一下什么是直接的朋友：每个对象都会与其他对象有耦合关系，只要两个对象之间有耦合关系，我们就说这两个对象之间是朋友关系。耦合的方式很多，依赖、关联、组合、聚合等。其中，我们称出现成员变量、方法参数、方法返回值中的类为直接的朋友，而出现在局部变量中的类则不是直接的朋友。也就是说，陌生的类最好不要作为局部变量的形式出现在类的内部。

举一个例子：有一个集团公司，下属单位有分公司和直属部门，现在要求打印出所有下属单位的员工ID。先来看一下违反迪米特法则的设计。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//总公司员工
class Employee{
    private String id;
    public void setId(String id){
        this.id = id;
    }
    public String getId(){
        return id;
    }
}

//分公司员工
class SubEmployee{
    private String id;
    public void setId(String id){
        this.id = id;
    }
    public String getId(){
        return id;
    }
}

class SubCompanyManager{
    public List<SubEmployee> getAllEmployee(){
        List<SubEmployee> list = new ArrayList<SubEmployee>();
        for(int i=0; i<100; i++){
            SubEmployee emp = new SubEmployee();
            //为分公司人员按顺序分配一个ID
            emp.setId("分公司"+i);
            list.add(emp);
        }
        return list;
    }
}

class CompanyManager{

    public List<Employee> getAllEmployee(){
        List<Employee> list = new ArrayList<Employee>();
        for(int i=0; i<30; i++){
            Employee emp = new Employee();
            //为总公司人员按顺序分配一个ID
            emp.setId("总公司"+i);
            list.add(emp);
        }
        return list;
    }
    
    public void printAllEmployee(SubCompanyManager sub){
        List<SubEmployee> list1 = sub.getAllEmployee();
        for(SubEmployee e:list1){
            System.out.println(e.getId());
        }

        List<Employee> list2 = this.getAllEmployee();
        for(Employee e:list2){
            System.out.println(e.getId());
        }
    }
}

public class Client{
    public static void main(String[] args){
        CompanyManager e = new CompanyManager();
        e.printAllEmployee(new SubCompanyManager());
    }
} 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

现在这个设计的主要问题出在CompanyManager中，根据迪米特法则，只与直接的朋友发生通信，而SubEmployee类并不是CompanyManager类的直接朋友（以局部变量出现的耦合不属于直接朋友），从逻辑上讲总公司只与他的分公司耦合就行了，与分公司的员工并没有任何联系，这样设计显然是增加了不必要的耦合。按照迪米特法则，应该避免类中出现这样非直接朋友关系的耦合。修改后的代码如下:

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
class SubCompanyManager{
    public List<SubEmployee> getAllEmployee(){
        List<SubEmployee> list = new ArrayList<SubEmployee>();
        for(int i=0; i<100; i++){
            SubEmployee emp = new SubEmployee();
            //为分公司人员按顺序分配一个ID
            emp.setId("分公司"+i);
            list.add(emp);
        }
        return list;
    }
    public void printEmployee(){
        List<SubEmployee> list = this.getAllEmployee();
        for(SubEmployee e:list){
            System.out.println(e.getId());
        }
    }
}

class CompanyManager{
    public List<Employee> getAllEmployee(){
        List<Employee> list = new ArrayList<Employee>();
        for(int i=0; i<30; i++){
            Employee emp = new Employee();
            //为总公司人员按顺序分配一个ID
            emp.setId("总公司"+i);
            list.add(emp);
        }
        return list;
    }
    
    public void printAllEmployee(SubCompanyManager sub){
        sub.printEmployee();
        List<Employee> list2 = this.getAllEmployee();
        for(Employee e:list2){
            System.out.println(e.getId());
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

修改后，为分公司增加了打印人员ID的方法，总公司直接调用来打印，从而避免了与分公司的员工发生耦合。

迪米特法则的初衷是降低类之间的耦合，由于每个类都减少了不必要的依赖，因此的确可以降低耦合关系。但是凡事都有度，虽然可以避免与非直接的类通信，但是要通信，必然会通过一个“中介”来发生联系，例如本例中，总公司就是通过分公司这个“中介”来与分公司的员工发生联系的。过分的使用迪米特原则，会产生大量这样的中介和传递类，导致系统复杂度变大。所以在采用迪米特法则时要反复权衡，既做到结构清晰，又要高内聚低耦合。

## 6.开闭原则（Open Close Principle）                                                

开闭原则就是说对扩展开放，对修改关闭。在程序需要进行拓展的时候，不能去修改原有的代码，实现一个热插拔的效果。所以一句话概括就是：为了使程序的扩展性好，易于维护和升级。想要达到这样的效果，需要面向接口编程。

定义：一个软件实体如类、模块和函数应该对扩展开放，对修改关闭。

问题由来：在软件的生命周期内，因为变化、升级和维护等原因需要对软件原有代码进行修改时，可能会给旧代码中引入错误，也可能会使我们不得不对整个功能进行重构，并且需要原有代码经过重新测试。

解决方案：当软件需要变化时，尽量通过扩展软件实体的行为来实现变化，而不是通过修改已有的代码来实现变化。

**开闭原则是面向对象设计中最基础的设计原则**，它指导我们如何建立稳定灵活的系统。开闭原则可能是设计模式六项原则中定义最模糊的一个了，它只告诉我们对扩展开放，对修改关闭，可是到底如何才能做到对扩展开放，对修改关闭，并没有明确的告诉我们。以前，如果有人告诉我“你进行设计的时候一定要遵守开闭原则”，我会觉的他什么都没说，但貌似又什么都说了。因为开闭原则真的太虚了。

如果仔细思考以及仔细阅读很多设计模式的文章后，会发现其实，我们遵循设计模式前面5大原则，以及使用23种设计模式的目的就是遵循开闭原则。也就是说，只要我们对前面5项原则遵守的好了，设计出的软件自然是符合开闭原则的，这个开闭原则更像是前面五项原则遵守程度的“平均得分”，前面5项原则遵守的好，平均分自然就高，说明软件设计开闭原则遵守的好；如果前面5项原则遵守的不好，则说明开闭原则遵守的不好。

开闭原则无非就是想表达这样一层意思：用抽象构建框架，用实现扩展细节。因为抽象灵活性好，适应性广，只要抽象的合理，可以基本保持软件架构的稳定。而软件中易变的细节，我们用从抽象派生的实现类来进行扩展，当软件需要发生变化时，我们只需要根据需求重新派生一个实现类来扩展就可以了。当然前提是我们的抽象要合理，要对需求的变更有前瞻性和预见性才行。

说到这里，再回想一下前面说的5项原则，恰恰是告诉我们**用抽象构建框架，用实现扩展细节的注意事项而已：单一职责原则告诉我们实现类要职责单一；里氏替换原则告诉我们不要破坏继承体系；依赖倒置原则告诉我们要面向接口编程；接口隔离原则告诉我们在设计接口的时候要精简单一；迪米特法则告诉我们要降低耦合。而开闭原则是总纲，他告诉我们要对扩展开放，对修改关闭。**

最后说明一下如何去遵守这六个原则。对这六个原则的遵守并不是是和否的问题，而是多和少的问题，也就是说，我们一般不会说有没有遵守，而是说遵守程度的多少。任何事都是过犹不及，设计模式的六个设计原则也是一样，制定这六个原则的目的并不是要我们刻板的遵守他们，而需要根据实际情况灵活运用。对他们的遵守程度只要在一个合理的范围内，就算是良好的设计。我们用一幅图来说明一下。

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170927223521856-615672071.png)

图中的每一条维度各代表一项原则，我们依据对这项原则的遵守程度在维度上画一个点，则如果对这项原则遵守的合理的话，这个点应该落在红色的同心圆内部；如果遵守的差，点将会在小圆内部；如果过度遵守，点将会落在大圆外部。一个良好的设计体现在图中，应该是六个顶点都在同心圆中的六边形。

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170927223703294-1177189859.png)

在上图中，设计1、设计2属于良好的设计，他们对六项原则的遵守程度都在合理的范围内；设计3、设计4设计虽然有些不足，但也基本可以接受；设计5则严重不足，对各项原则都没有很好的遵守；而设计6则遵守过渡了，设计5和设计6都是迫切需要重构的设计。

设计模式的六大原则主要如上，主要参考书籍有《设计模式》《设计模式之禅》《大话设计模式》、CSDN 的zhengzhb 博主以及网上一些零散的文章。下面将开始归纳总结下各个设计模式。



# 面向对象五大原则

## 开闭原则

### 什么是开放封闭原则

　　**开放封闭原则（OCP，Open Closed Principle）是所有面向对象原则的核心**。软件设计本身所追求的目标就是封装变化、降低耦合，而开放封闭原则正是对这一目标的最直接体现。其他的设计原则，很多时候是为实现这一目标服务的，例如以Liskov替换原则实现最佳的、正确的继承层次，就能保证不会违反开放封闭原则。　

　　**关于开放封闭原则，其核心的思想是：软件实体应该是可扩展，而不可修改的。也就是说，一个软件实体应当对扩展是开放的，而对修改是封闭的。**　

　　因此，开放封闭原则主要**体现在两个方面**：**对扩展开放**，意味着有新的需求或变化时，可以对现有代码进行扩展，以适应新的情况。**对修改封闭**，意味着类一旦设计完成，就可以独立完成其工作，而不要对类进行任何修改。

　　在设计一个模块时，应当使得这个模块可以在不被修改的前提下被扩展。也就是说，应当可以在不必修改源代码的情况下修改这个模块的行为。

　　设计的目的便在于面对需求的改变而保持系统的相对稳定，从而使得系统可以很容易的从一个版本升级到另一个版本。

　　“需求总是变化”、“世界上没有一个软件是不变的”，这些言论是对软件需求最经典的表白。从中透射出一个关键的意思就是，对于软件设计者来说，必须在不需要对原有的系统进行修改的情况下，实现灵活的系统扩展。而如何能做到这一点呢？

　　只有依赖于抽象。实现开放封闭的核心思想就是对抽象编程，而不对具体编程，因为抽象相对稳定。让类依赖于固定的抽象，所以对修改就是封闭的；而通过面向对象的继承和对多态机制，可以实现对抽象体的继承，通过覆写其方法来改变固有行为，实现新的扩展方法，所以对于扩展就是开放的。这是

实施开放封闭原则的基本思路，同时这种机制是建立在两个基本的设计原则的基础上，这就是Liskov替换原则和合成/聚合复用原则。

　　对于违反这一原则的类，必须进行重构来改善，常用于实现的设计模式主要有Template Method模式和Strategy模式。而封装变化，是实现这一原则的重要手段，将经常发生变化的状态封装为一个类。

### 怎样做到开放封闭原则

　　实际上，绝对封闭的系统是不存在的。无论模块是怎么封闭，到最后，总还是有一些无法封闭的变化。而我们的思路就是：既然不能做到完全封闭，那我们就应该对那些变化封闭，那些变化隔离做出选择。我们做出选择，然后将那些无法封闭的变化抽象出来，进行隔离，允许扩展，尽可能的减少系统的开发。当系统变化来临时，我们要及时的做出反应。

   我们并不害怕改变的到来。当变化到来时，我们首先需要做的不是修改代码，而是尽可能的将变化抽象出来进行隔离，然后进行扩展。面对需求的变化，对程序的修改应该是尽可能通过添加代码来实现，而不是通过修改代码来实现。

   实际上，变化或者可能的变化来的越早，抽象就越容易，相对的，代码的维护也就越容易；而当项目接近于完成而来的需求变化，则会使抽象变得很困难——这个困难，并不是抽象本身的困难，抽象本身并没有困难，困难在于系统的架构已经完成，修改牵扯的方面太多而使得抽象工作变得很困难。

　　我们举个银行业务的例子

　　银行有四项业务，存款，取款，转账和买基金

　　如果用普通的方式来写![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 /*
 2  * 银行业务员
 3  */
 4 public class BankWorker {
 5     //负责存款业务
 6     public void saving() {
 7         System.out.println("进行存款操作");
 8     }
 9     
10     //负责取款业务
11     public void drawing() {
12         System.out.println("进行取款操作");
13     }
14     
15     //负责转账
16     public void zhuanzhang() {
17         System.out.println("进行转账操作");
18     }
19     
20     //负责基金的申购
21     public void jijin() {
22         System.out.println("进行基金申购操作");
23     }
24 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这样等同于一个业务员负责了所有的业务，站在银行窗口焦急等待的用户，在长长的队伍面前显得无奈。所以，将这种无奈迁怒到银行的头上是理所当然的，因为银行业务的管理显然有不当之处。银行的业务人员面对蜂拥而至的客户需求，在排队等待的人们并非只有一种需求，有人存款、有人转账，

​	也有人申购基金，繁忙的业务员来回在不同的需求中穿梭，手忙脚乱的寻找各种处理单据，电脑系统的功能模块也在不同的需求要求下来回切换，这就是一个发生在银行窗口内外的无奈场景。而我每次面对统一排队的叫号系统时，都为前面长长的等待人群而叫苦，从梳理银行业务员的职责来看，在管理上他们负责的业务过于繁多，将其对应为软件设计来实现，这种拙劣的设计就上面例子中的方式。这样，如果要修改的话，如果银行新添加了一项业务，这个业务员就得新增这项业务，这样扩展行就很差，而且，只要新增业务就要修改源码。

　　所以下面我们用符合开放封闭原则的方式来编写代码

　　把不同的业务分配给不同的业务员，所以先编写抽象业务员父类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 /*
2  * 银行业务员接口，是所有银行业务员的抽象父类
3  */
4 public interface BankWorker {
5     public void operation();
6 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　然后，在编写各个负责各个业务的业务员

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 负责存款业务的业务员
 3  */
 4 public class SavingBankWorker implements BankWorker {
 5 
 6     public void operation() {
 7         System.out.println("进行存款操作");
 8     }
 9     
10 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 负责取款业务的业务员
 3  */
 4 public class WithdrawalsBankWorker  implements BankWorker{
 5 
 6     public void operation() {
 7         System.out.println("进行取款操作");
 8     }
 9     
10 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 负责转账业务的业务员
 3  */
 4 public class ZhuanZhangBankWorker implements BankWorker {
 5 
 6     public void operation() {
 7         System.out.println("进行转账操作");
 8     }
 9 
10 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 负责基金业务的业务员
 3  */
 4 public class JiJinBankWorker implements BankWorker {
 5 
 6     public void operation() {
 7         System.out.println("进行基金申购操作");
 8     }
 9 
10 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　可以看到，这样的形式就可以做到，增加业务只需新增业务员即可，不必对原有业务进行任何的修改，也符合了开放封闭原则　

### 开放封闭原则的优越性

　　1.通过扩展已有的软件系统，可以提供新的行为，以满足对软件的新需求，是变化中的软件有一定的适应性和灵活性。

　　2.已有的软件模块，特别是最重要的抽象模块不能再修改，这就使变化中的软件系统有一定的稳定性和延续性。

### 使用建议

　　1、开放封闭原则，是最为重要的设计原则，Liskov替换原则和合成/聚合复用原则为开放封闭原则的实现提供保证。

　　2、可以通过Template Method模式和Strategy模式进行重构，实现对修改封闭、对扩展开放的设计思路。

　　3、封装变化，是实现开放封闭原则的重要手段，对于经常发生变化的状态一般将其封装为一个抽象，例如银行业务中的IBankProcess接口。

　　4、拒绝滥用抽象，只将经常变化的部分进行抽象，这种经验可以从设计模式的学习与应用中获得。



## 单一职责原则

### 什么是单一职责原则

　　单一职责原则（SRP：Single responsibility principle）又称单一功能原则，面向对象五个基本原则（SOLID）之一。它规定一个类应该只有一个发生变化的原因。

　　所谓职责是指类变化的原因。如果一个类有多于一个的动机被改变，那么这个类就具有多于一个的职责。而单一职责原则就是指一个类或者模块应该有且只有一个改变的原因。

### 为什么使用单一职责原则

　　如果一个类承担的职责过多，就等于把这些职责耦合在一起了。一个职责的变化可能会削弱或者抑制这个类完成其他职责的能力。这种耦合会导致脆弱的设计，当发生变化时，设计会遭受到意想不到的破坏。而如果想要避免这种现象的发生，就要尽可能的遵守单一职责原则。此原则的核心就是解耦和增强内聚性。

　　就好比，现在的手机，拥有各种各样的功能，拍照，打电话，发短信，玩游戏，导航。但是每一个功能又都不是那么强大，远远比不上专业的。

　　拍照比不上专业单反，玩游戏比不上电脑或专业游戏机，导航比不上专业导航设备等等，这就是因为手机的职责太多了　　

　　一个类，只有一个引起它变化的原因。应该只有一个职责。每一个职责都是变化的一个轴线，如果一个类有一个以上的职责，这些职责就耦合在了一起。这会导致脆弱的设计。当一个职责发生变化时，可能会影响其它的职责。另外，多个职责耦合在一起，会影响复用性。例如：要实现逻辑和界面的分离。



## 里氏代换原则

### 什么是里氏代换原则　

　　里氏代换原则(Liskov Substitution Principle LSP)面向对象设计的基本原则之一。 **里氏代换原则中说，任何基类可以出现的地方，子类一定可以出现**。 LSP是继承复用的基石，只有当衍生类可以替换掉基类，软件单位的功能不受到影响时，基类才能真正被复用，而衍生类也能够在基类的基础上增加新的行为。里氏代换原则是对“开-闭”原则的补充。实现“开-闭”原则的关键步骤就是抽象化。而基类与子类的继承关系就是抽象化的具体实现，所以里氏代换原则是对实现抽象化的具体步骤的规范。

　　简单的理解为一个软件实体如果使用的是一个父类，那么一定适用于其子类，而且它察觉不出父类对象和子类对象的区别。也就是说，软件里面，把父类都替换成它的子类，程序的行为没有变化。

　　但是反过来的代换却不成立，里氏代换原则(Liskov Substitution Principle)：一个软件实体如果使用的是一个子类的话，那么它不能适用于其父类。

 　举个例子解释一下这个概念

　　先创建一个Person类

```
1 public class Person {
2     public void display() {
3         System.out.println("this is person");
4     }
5 }
```

　　再创建一个Man类，继承这个Person类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class Man extends Person {
2 
3     public void display() {
4         System.out.println("this is man");
5     }
6     
7 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　运行一下

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         Person person = new Person();//new一个Person实例
 4         display(person);
 5         
 6         Person man = new Man();//new一个Man实例
 7         display(man);
 8     }
 9     
10     public static void display(Person person) {
11         person.display();
12     }
13 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　可以看到

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180329091734851-1982402157.png)

　　运行没有影响，符合**一个软件实体如果使用的是一个父类的话，那么一定适用于其子类，而且它察觉不出父类和子类对象的区别**这句概念，这也就是java中的多态。

　　而反之，一个子类的话，那么它不能适用于其父类，这样，程序就会报错

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         Person person = new Person();
 4         display(person);//这里报错
 5         
 6         Man man = new Man();
 7         display(man);
 8     }
 9     
10     public static void display(Man man) {//传入一个子类
11         man.display();
12     }
13 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　继续再举一个很经典的例子，正方形与长方形是否符合里氏代换原则，也就是说正方形是否是长方形的一个子类，

 　以前，我们上学都说正方形是特殊的长方形，是宽高相等的长方形，所以我们认为正方形是长方形的子类，但真的是这样吗？

　　![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180329094003270-1221532256.png)

　　从图中，我们可以看到长方形有两个属性宽和高，而正方形则只有一个属性边长

　　所以，用代码如此实现

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 //长方形
 2 public class Changfangxing{
 3     private long width;
 4     private long height;
 5     
 6     public long getWidth() {
 7         return width;
 8     }
 9     public void setWidth(long width) {
10         this.width = width;
11     }
12     public long getHeight() {
13         return height;
14     }
15     public void setHeight(long height) {
16         this.height = height;
17     }
18 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 //正方形
 2 public class Zhengfangxing{
 3     private long side;
 4 
 5     public long getSide() {
 6         return side;
 7     }
 8 
 9     public void setSide(long side) {
10         this.side = side;
11     }
12 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　可以看到，它们的结构根本不同，所以正方形不是长方形的子类，所以长方形与正方形之间并不符合里氏代换原则。

　　当然我们也可以强行让正方形继承长方形

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 //正方形
 2 public class Zhengfangxing extends Changfangixng{
 3     private long side;
 4 
 5     public long getHeight() {
 6         return this.getSide();
 7     }
 8 
 9     public long getWidth() {
10         return this.getSide();
11     }
12 
13     public void setHeight(long height) {
14         this.setSide(height);
15     }
16 
17     public void setWidth(long width) {
18         this.setSide(width);
19     }
20 
21     public long getSide() {
22         return side;
23     }
24 
25     public void setSide(long side) {
26         this.side = side;
27     }
28 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　这个样子，编译器是可以通过的，也可以正常使用，但是这样就符合里氏代换原则了吗，肯定不是的。

　　我们不是为了继承而继承，只有真正符合继承条件的情况下我们才去继承，所以像这样为了继承而继承，强行实现继承关系的情况也是不符合里氏代换原则的。

　　但这是为什么呢？，我们运行一下

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         Changfangxing changfangxing = new Changfangxing();
 4         changfangxing.setHeight(10);
 5         changfangxing.setWidth(20);
 6         test(changfangxing);
 7         
 8         Changfangxing zhengfangxing = new Zhengfangxing();
 9         zhengfangxing.setHeight(10);
10         test(zhengfangxing);
11     }
12     
13     public static void test(Changfangxing changfangxing) {
14         System.out.println(changfangxing.getHeight());
15         System.out.println(changfangixng.getWidth());
16     }
17 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　结果：

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180329100252855-1181582385.png)

　　我们忽然发现，很正常啊，为什么不可以，但是我们继续修改

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         Changfangxing changfangxing = new Changfangxing();
 4         changfangxing.setHeight(10);
 5         changfangxing.setWidth(20);
 6         resize(changfangxing);
 7         
 8         Changfangxing zhengfangxing = new Zhengfangxing();
 9         zhengfangxing.setHeight(10);
10         resize(zhengfangxing);
11     }
12     
13     public static void test(Changfangxing changfangxing) {
14         System.out.println(changfangxing.getHeight());
15         System.out.println(changfangxing.getWidth());
16     }
17     
18     public static void resize(Changfangxing changfangxing) {
19         while(changfangxing.getHeight() <= changfangxing.getWidth()) {
20             changfangxing.setHeight(changfangxing.getHeight() + 1);
21             test(changfangxing);
22         }
23     }
24 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　当长方形运行时，可以正常运行，而正方形则会造成死循环，所以这种继承方式不一定恩能够适用于所有情况，所以不符合里氏代换原则。

　　还有一种形式，我们抽象出一个四边形接口，让长方形和正方形都实现这个接口

```
1 public interface Sibianxing {
2     public long getWidth();
3     public long getHeight();
4 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class Changfangxing implements Sibianxing{
 2     private long width;
 3     private long height;
 4     
 5     public long getWidth() {
 6         return width;
 7     }
 8     public void setWidth(long width) {
 9         this.width = width;
10     }
11     public long getHeight() {
12         return height;
13     }
14     public void setHeight(long height) {
15         this.height = height;
16     }
17 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 package com.ibeifeng.ex3;
 2 
 3 public class Zhengfangxing implements Sibianxing{
 4     private long side;
 5 
 6     public long getHeight() {
 7         return this.getSide();
 8     }
 9 
10     public long getWidth() {
11         return this.getSide();
12     }
13 
14     public void setHeight(long height) {
15         this.setSide(height);
16     }
17 
18     public void setWidth(long width) {
19         this.setSide(width);
20     }
21 
22     public long getSide() {
23         return side;
24     }
25 
26     public void setSide(long side) {
27         this.side = side;
28     }
29 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　运行

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         Changfangxing changfangxing = new Changfangxing();
 4         changfangxing.setHeight(10);
 5         changfangxing.setWidth(20);
 6         test(changfangxing);
 7         
 8         Zhengfangxing zhengfangxing = new Zhengfangxing();
 9         zhengfangxing.setHeight(10);
10         test(zhengfangxing);
11     }
12     
13     public static void test(Sibianxing sibianxing) {
14         System.out.println(sibianxing.getHeight());
15         System.out.println(sibianxing.getWidth());
16     }
17 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　对于长方形和正方形，取width和height是它们共同的行为，但是给width和height赋值，两者行为不同，因此，这个抽象的四边形的类只有取值方法，没有赋值方法。上面的例子中那个方法只会适用于不同的子类，LSP也就不会被破坏。

### 注意事项

　　在进行设计的时候，尽量从抽象类继承，而不是从具体类继承。如果从继承等级树来看，所有叶子节点应当是具体类，而所有的树枝节点应当是抽象类或者接口。当然这个只是一个一般性的指导原则，使用的时候还要具体情况具体分析。



## 依赖倒转原则

### 什么是依赖倒转原则

　　依赖倒转(Dependence Inversion Principle )：是**程序要依赖于抽象接口，不要依赖于具体实现**。简单的说就是要求对抽象进行编程，不要对实现进行编程，这样就降低了客户与实现模块间的耦合。

　　1.抽象不应该依赖于细节，细节应该依赖于抽象。

　　2.高层模块不依赖底层模块，两者都依赖抽象。　

　　我们举个例子：电脑有不同的组件，硬盘，内存，主板。

　　硬盘抽象类

```
1 //硬盘抽象类
2 public abstract class HardDisk {
3     public abstract void doSomething();
4 }
```

　　具体硬盘（希捷硬盘）

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 //希捷硬盘
2 public class XiJieHardDisk extends HardDisk {
3 
4     public void doSomething() {
5         System.out.println("希捷硬盘");
6     }
7 
8 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　具体硬盘（西数硬盘）

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class XiShuHardDisk extends HardDisk {
2 
3     public void doSomething() {
4         System.out.println("西数硬盘");
5     }
6 
7 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　主板抽象类　

```
1 //主板抽象类
2 public abstract class MainBoard {
3     public abstract void doSomething();
4 }
```

 　具体主板（华硕主板）

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class HuaShuoMainBoard extends MainBoard{
2 
3     public void doSomething() {
4         System.out.println("华硕主板");
5     }
6 
7 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　具体主板（微星主板）

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class WeiXingMainBoard extends MainBoard {
2 
3     public void doSomething() {
4         System.out.println("微星主板");
5     }
6 
7 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　内存抽象类　　

```
1 //内存抽象类
2 public abstract class Memory {
3     public abstract void doSomething();
4 }
```

　　具体内存（金士顿内存）

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class JinShiDunMemory extends Memory {
2 
3     public void doSomething() {
4         System.out.println("金士顿内存");
5     }
6 
7 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　具体内存（三星内存）

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class SanxingMemory extends Memory {
2 
3     public void doSomething() {
4         System.out.println("三星内存");
5     }
6 
7 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　现在，电脑的各个零部件都有了，只差电脑了。首先，我们不按照依赖倒转原则，按照传统模式

　　**传统的过程式设计倾向于使高层次的模块依赖于低层次的模块，抽象层依赖于具体的层次。**

**![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180329140516553-411474334.png)**

　　这样，电脑应该是这样的

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 //电脑
 2 public class Computer{
 3     private HuaShuoMainBoard huaShuoMainBoard;
 4     private JinShiDunMemory jinShiDunMemory;
 5     private XiJieHardDisk xiJieHardDisk;
 6     
 7     public HuaShuoMainBoard getHuaShuoMainBoard() {
 8         return huaShuoMainBoard;
 9     }
10     public void setHuaShuoMainBoard(HuaShuoMainBoard huaShuoMainBoard) {
11         this.huaShuoMainBoard = huaShuoMainBoard;
12     }
13     public JinShiDunMemory getJinShiDunMemory() {
14         return jinShiDunMemory;
15     }
16     public void setJinShiDunMemory(JinShiDunMemory jinShiDunMemory) {
17         this.jinShiDunMemory = jinShiDunMemory;
18     }
19     public XiJieHardDisk getXiJieHardDisk() {
20         return xiJieHardDisk;
21     }
22     public void setXiJieHardDisk(XiJieHardDisk xiJieHardDisk) {
23         this.xiJieHardDisk = xiJieHardDisk;
24     }
25 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这时，要组装一台电脑

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class MainClass {
    public static void main(String[] args) {
        Computer computer = new Computer();
        computer.setHuaShuoMainBoard(new HuaSuoMainBoard());
        computer.setJinShiDunMemory(new JinShiDunMemory());
        computer.setXiJieHardDisk(new XiJieHardDisk());
        
        computer.setHuaShuoMainBoard(new WeiXingMainBoard());//报错，无法安装
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　可以看到，这种情况下，这台电脑就只能安装华硕主板，金士顿内存和希捷硬盘了，这对用户肯定是不友好的，用户有了机箱肯定是想按照自己的喜好，选择自己喜欢的配件。

　　电脑就是高层业务逻辑，主板，内存，硬盘就是中层模块，还有更低的底层模块我们没有写那么细，但都是一个意思，这样的方式显然是不可取的。

　　下面，我们改造一下，让Computer依赖接口或抽象类，下面的模块同样如此

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180329143240833-506477182.png)

 

 

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180329143816487-1373350578.png)

 

　　Computer

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class Computer {
 2     private MainBoard mainBoard;
 3     private Memory memory;
 4     private HardDisk harddisk;
 5 
 6     public MainBoard getMainBoard() {
 7         return mainBoard;
 8     }
 9 
10     public void setMainBoard(MainBoard mainBoard) {
11         this.mainBoard = mainBoard;
12     }
13 
14     public Memory getMemory() {
15         return memory;
16     }
17 
18     public void setMemory(Memory memory) {
19         this.memory = memory;
20     }
21 
22     public HardDisk getHarddisk() {
23         return harddisk;
24     }
25 
26     public void setHarddisk(HardDisk harddisk) {
27         this.harddisk = harddisk;
28     }
29 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　这时，再组装

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         Computer computer = new Computer();
 4         computer.setMainBoard(new HuaSuoMainBoard());
 5         computer.setMemory(new JinShiDunMemory());
 6         computer.setHarddisk(new XiJieHardDisk());
 7         
 8         computer.setMainBoard(new WeiXingMainBoard());//完全没有问题
 9     }
10 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这样，用户就可以根据自己的喜好来选择自己喜欢的品牌，组装电脑了。

### 为什么采取依赖倒转

　　面向过程的开发，上层调用下层，上层依赖于下层，当下层剧烈变动时上层也要跟着变动，这就会导致模块的复用性降低而且大大提高了开发的成本。

　　面向对象的开发很好的解决了这个问题，一般情况下抽象的变化概率很小，让用户程序依赖于抽象，实现的细节也依赖于抽象。即使实现细节不断变动，只要抽象不变，客户程序就不需要变化。这大大降低了客户程序与实现细节的耦合度。

### 依赖倒转模式应用实例

　　1.[工厂方法模式](http://www.cnblogs.com/xiaobai1226/p/8482256.html)

　　2.[模板方法模式](http://www.cnblogs.com/xiaobai1226/p/8629441.html)

　　3.[迭代模式](http://www.cnblogs.com/xiaobai1226/p/8617511.html)

  综上所诉，我们可以看出一个应用中的重要策略决定及业务模型正是在这些高层的模块中。也正是这些模型包含着应用的特性。但是，当这些模块依赖于低层模块时，低层模块的修改将会直接影响到它们，迫使它们也去改变。这种境况是荒谬的。应该是处于高层的模块去迫使那些低层的模块发生改变。应该是处于高层的模块优先于低层的模块。无论如何高层的模块也不应依赖于低层的模块。而且，我们想能够复用的是高层的模块。通过子程序库的形式，我们已经可以很好地复用低层的模块了。当高层的模块依赖于低层的模块时，这些高层模块就很难在不同的环境中复用。但是，当那些高层模块独立于低层模块时，它们就能很简单地被复用了。这正是位于框架设计的最核心之处的原则。



## 迪米特法则

### 什么是迪米特法则

　　迪米特法则(Law of Demeter )又叫做**最少知识原则**，也就是说，**一个对象应当对其他对象尽可能少的了解**。不和陌生人说话。英文简写为: LoD。

　　迪米特法则最初是用来作为面向对象的系统设计风格的一种法则，于1987年秋天由lan holland在美国东北大学为一个叫做迪米特的项目设计提出的。

### 迪米特法则的模式与意义

　　迪米特法则可以简单说成：talk only to your immediate friends。 对于OOD来说，又被解释为下面几种方式：**一个软件实体应当尽可能少的与其他实体发生相互作用**。**每一个软件单位对其他的单位都只有最少的知识，而且局限于那些与本单位密切相关的软件单位**。

　　迪米特法则的初衷在于**降低类之间的耦合**。由于每个类尽量减少对其他类的依赖，因此，很容易使得**系统的功能模块功能独立**，相互之间不存在（或很少有）依赖关系。

　　迪米特法则不希望类之间建立直接的联系。如果真的有需要建立联系，也希望能通过它的友元类来转达。因此，应用迪米特法则有可能造成的一个后果就是：系统中存在大量的中介类，这些类之所以存在完全是为了传递类之间的相互调用关系——这在一定程度上增加了系统的复杂度。

### 狭义的迪米特法则

　　如果两个类不必彼此直接通信，那么这两个类就不应当发生直接的相互作用。如果其中一个类需要调用另一类的某一个方法的话，可以通过第三者转发这个调用。

　　这么看不太形象，我们来举个例子，和陌生人说话，甲和朋友认识，朋友和陌生人认识，而甲和陌生人不认识，这时甲可以直接和朋友说话，朋友可以直接和陌生人说话，而如果甲想和陌生人说话，就必须通过朋友

　　首先，第一种方式代码实现：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class Jia{
2     public void play(Friend friend){
3         friend.play();
4     }
5     
6     public void play(Stranger stranger) {
7         stranger.play();
8     }
9 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 //朋友
2 public class Friend {
3     public void play(){
4         System.out.println("朋友");
5     }
6 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 //陌生人
2 public class Stranger {
3     public void play(){
4         System.out.println("陌生人");
5     }
6 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　这种方式是肯定不对的，甲根本没有通过朋友，直接引用了陌生人的方法，不符合迪米特法则

　　第二种方式

 　![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180329153518084-1135867489.png)

 　代码实现

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 //甲
2 public class Jia{
3     public void play(Friend friend){
4         friend.play();
5         Stranger stranger = friend.getStranger();
6         stranger.play();
7     }
8 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 //朋友
 2 public class Friend {
 3     public void play(){
 4         System.out.println(朋友");
 5     }
 6     
 7     public Stranger getStranger() {
 8         return new Stranger();
 9     }
10 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 //陌生人
2 public class Stranger {
3     public void play(){
4         System.out.println("陌生人");
5     }
6 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这样的方式呢，看上去陌生人的实例是通过朋友来创建了，但还是不行，因为甲中包含的陌生人的引用，甲还是和陌生人直接关联上了，所以，不符合迪米特法则，我们要的是甲和陌生人没有一丁点直接关系

　　第三种方式

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180329155545617-939107079.png)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 //甲
 2 public class Jia{
 3     private Friend friend;
 4     
 5     public Friend getFriend() {
 6         return friend;
 7     }
 8 
 9     public void setFriend(Friend friend) {
10         this.friend = friend;
11     }
12 
13     public void play(Friend friend){
14         friend.play();
15     }
16 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 //朋友
 2 public class Friend {
 3     public void play(){
 4         System.out.println("朋友");
 5     }
 6     
 7     public void playWithStranger() {
 8         Stranger stranger = new Stranger();
 9         stranger.play();
10     }
11 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class Stranger {
2     public void play(){
3         System.out.println("陌生人");
4     }
5 }
```

 　这种方式，甲和陌生人之间就没有了任何直接联系，这样就避免了甲和陌生人的耦合度过高

　　当然还有一种更好的方式，与依赖倒转原则结合，为陌生人创建一个接口

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180329160712004-1751642978.png)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 //甲
 2 public class Jia{
 3     private Friend friend;
 4     private Stranger stranger;
 5 
 6     public Stranger getStranger() {
 7         return stranger;
 8     }
 9 
10     public void setStranger(Stranger stranger) {
11         this.stranger = stranger;
12     }
13 
14     public Friend getFriend() {
15         return friend;
16     }
17 
18     public void setFriend(Friend friend) {
19         this.friend = friend;
20     }
21 
22     public void play() {
23         System.out.println("someone play");
24         friend.play();
25         stranger.play();
26     }
27 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class Friend {
2     public void play(){
3         System.out.println("朋友");
4     }
5 }
1 //陌生人抽象类
2 public abstract class Stranger {
3     public abstract void play();
4 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 //具体陌生人
2 public class StrangerA extends Stranger {
3 
4     public void play() {
5         System.out.println("陌生人");
6     }
7 
8 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这样的方式，和甲直接通信的是陌生人的抽象父类，和具体陌生人没有直接关系，所以符合迪米特法则

### 狭义的迪米特法则的缺点

　　1、在系统里造出大量的小方法，这些方法仅仅是传递间接的调用，与系统的商务逻辑无关。

　　2、遵循类之间的迪米特法则会是一个系统的局部设计简化，因为每一个局部都不会和远距离的对象有直接的关联。但是，这也会造成系统的不同模块之间的通信效率降低，也会使系统的不同模块之间不容易协调。

###  迪米特法则应用实例

　　1.[外观模式](http://www.cnblogs.com/xiaobai1226/p/8566231.html)

　　2.[中介者模式](http://www.cnblogs.com/xiaobai1226/p/8609719.html)



## [JAVA23种设计模式总结](https://www.cnblogs.com/pony1223/p/7608955.html)

上一篇总结了设计模式的六大原则《[JAVA设计模式总结之六大设计原则](http://www.cnblogs.com/pony1223/p/7594803.html)》，这一篇，正式进入到介绍23种设计模式的归纳总结。

**一、什么是设计模式**                                                                    

设计模式（Design pattern）是一套被反复使用、多数人知晓的、经过分类编目的、代码设计经验的总结。使用设计模式是为了可重用代码、让代码更容易被他人理解、保证代码可靠性。 毫无疑问，设计模式于己于他人于系统都是多赢的，设计模式使代码编制真正工程化，设计模式是软件工程的基石，如同大厦的一块块砖石一样。项目中合理的运用设计模式可以完美的解决很多问题，每种模式在现在中都有相应的原理来与之对应，每一个模式描述了一个在我们周围不断重复发生的问题，以及该问题的核心解决方案，这也是它能被广泛应用的原因。简单说：

**模式：在某些场景下，针对某类问题的某种通用的解决方案。**

场景：项目所在的环境

问题：约束条件，项目目标等

解决方案：通用、可复用的设计，解决约束达到目标。

**二、设计模式的三个分类**                                                                 

**创建型模式：对象实例化的模式，创建型模式用于解耦对象的实例化过程。**

**结构型模式：把类或对象结合在一起形成一个更大的结构。**

**行为型模式：类和对象如何交互，及划分责任和算法。**

**如下图所示：**

**![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170928225241215-295252070.png)**

**三、各分类中模式的关键点**                                                               

单例模式：某个类只能有一个实例，提供一个全局的访问点。

简单工厂：一个工厂类根据传入的参量决定创建出那一种产品类的实例。

工厂方法：定义一个创建对象的接口，让子类决定实例化那个类。

抽象工厂：创建相关或依赖对象的家族，而无需明确指定具体类。

建造者模式：封装一个复杂对象的构建过程，并可以按步骤构造。

原型模式：通过复制现有的实例来创建新的实例。

 

适配器模式：将一个类的方法接口转换成客户希望的另外一个接口。

组合模式：将对象组合成树形结构以表示“”部分-整体“”的层次结构。

装饰模式：动态的给对象添加新的功能。

代理模式：为其他对象提供一个代理以便控制这个对象的访问。

亨元（蝇量）模式：通过共享技术来有效的支持大量细粒度的对象。

外观模式：对外提供一个统一的方法，来访问子系统中的一群接口。

桥接模式：将抽象部分和它的实现部分分离，使它们都可以独立的变化。

 

模板模式：定义一个算法结构，而将一些步骤延迟到子类实现。

解释器模式：给定一个语言，定义它的文法的一种表示，并定义一个解释器。

策略模式：定义一系列算法，把他们封装起来，并且使它们可以相互替换。

状态模式：允许一个对象在其对象内部状态改变时改变它的行为。

观察者模式：对象间的一对多的依赖关系。

备忘录模式：在不破坏封装的前提下，保持对象的内部状态。

中介者模式：用一个中介对象来封装一系列的对象交互。

命令模式：将命令请求封装为一个对象，使得可以用不同的请求来进行参数化。

访问者模式：在不改变数据结构的前提下，增加作用于一组对象元素的新功能。

责任链模式：将请求的发送者和接收者解耦，使的多个对象都有处理这个请求的机会。

迭代器模式：一种遍历访问聚合对象中各个元素的方法，不暴露该对象的内部结构。

 

**四、概说23种设计模式**                                                                   

1.单例模式                                                                       

单例模式，它的定义就是确保某一个类只有一个实例，并且提供一个全局访问点。

单例模式具备典型的3个特点：1、只有一个实例。 2、自我实例化。 3、提供全局访问点。

 因此当系统中只需要一个实例对象或者系统中只允许一个公共访问点，除了这个公共访问点外，不能通过其他访问点访问该实例时，可以使用单例模式。

单例模式的主要优点就是节约系统资源、提高了系统效率，同时也能够严格控制客户对它的访问。也许就是因为系统中只有一个实例，这样就导致了单例类的职责过重，违背了“单一职责原则”，同时也没有抽象类，所以扩展起来有一定的困难。其UML结构图非常简单，就只有一个类，如下图：

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170929202530606-709085396.png)

 

2.工厂方法模式                                                                   

作为抽象工厂模式的孪生兄弟，工厂方法模式定义了一个创建对象的接口，但由子类决定要实例化的类是哪一个，也就是说工厂方法模式让实例化推迟到子类。

工厂方法模式非常符合“开闭原则”，当需要增加一个新的产品时，我们只需要增加一个具体的产品类和与之对应的具体工厂即可，无须修改原有系统。同时在工厂方法模式中用户只需要知道生产产品的具体工厂即可，无须关系产品的创建过程，甚至连具体的产品类名称都不需要知道。虽然他很好的符合了“开闭原则”，但是由于每新增一个新产品时就需要增加两个类，这样势必会导致系统的复杂度增加。其UML结构图：

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170929204041684-1520979160.png)

 

3.抽象工厂模式                                                                   

所谓抽象工厂模式就是提供一个接口，用于创建相关或者依赖对象的家族，而不需要明确指定具体类。他允许客户端使用抽象的接口来创建一组相关的产品，而不需要关系实际产出的具体产品是什么。这样一来，客户就可以从具体的产品中被解耦。它的优点是隔离了具体类的生成，使得客户端不需要知道什么被创建了，而缺点就在于新增新的行为会比较麻烦，因为当添加一个新的产品对象时，需要更加需要更改接口及其下所有子类。其UML结构图如下：

 ![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170929204151184-2094793629.png)

 

4.建造者模式                                                                      

对于建造者模式而已，它主要是将一个复杂对象的构建与表示分离，使得同样的构建过程可以创建不同的表示。适用于那些产品对象的内部结构比较复杂。

建造者模式将复杂产品的构建过程封装分解在不同的方法中，使得创建过程非常清晰，能够让我们更加精确的控制复杂产品对象的创建过程，同时它隔离了复杂产品对象的创建和使用，使得相同的创建过程能够创建不同的产品。但是如果某个产品的内部结构过于复杂，将会导致整个系统变得非常庞大，不利于控制，同时若几个产品之间存在较大的差异，则不适用建造者模式，毕竟这个世界上存在相同点大的两个产品并不是很多，所以它的使用范围有限。其UML结构图：

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170929204518044-666328371.png)

 

 

5.原型模式                                                                       

在我们应用程序可能有某些对象的结构比较复杂，但是我们又需要频繁的使用它们，如果这个时候我们来不断的新建这个对象势必会大大损耗系统内存的，这个时候我们需要使用原型模式来对这个结构复杂又要频繁使用的对象进行克隆。所以原型模式就是用原型实例指定创建对象的种类，并且通过复制这些原型创建新的对象。

它主要应用与那些创建新对象的成本过大时。它的主要优点就是简化了新对象的创建过程，提高了效率，同时原型模式提供了简化的创建结构。UML结构图：

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170929205441153-1950745368.png)

 

**模式结构**
原型模式包含如下角色：
Prototype：抽象原型类
ConcretePrototype：具体原型类
Client：客户类

6.适配器模式                                                                       

在我们的应用程序中我们可能需要将两个不同接口的类来进行通信，在不修改这两个的前提下我们可能会需要某个中间件来完成这个衔接的过程。这个中间件就是适配器。所谓适配器模式就是将一个类的接口，转换成客户期望的另一个接口。它可以让原本两个不兼容的接口能够无缝完成对接。

作为中间件的适配器将目标类和适配者解耦，增加了类的透明性和可复用性。

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170929205627606-1781915371.png)

 

适配器模式包含如下角色：
Target：目标抽象类
Adapter：适配器类
Adaptee：适配者类
Client：客户类

7.桥接模式                                                                        

如果说某个系统能够从多个角度来进行分类，且每一种分类都可能会变化，那么我们需要做的就是讲这多个角度分离出来，使得他们能独立变化，减少他们之间的耦合，这个分离过程就使用了桥接模式。所谓桥接模式就是讲抽象部分和实现部分隔离开来，使得他们能够独立变化。

桥接模式将继承关系转化成关联关系，封装了变化，完成了解耦，减少了系统中类的数量，也减少了代码量。

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170929205836028-1108970959.png)

桥接模式包含如下角色：
Abstraction：抽象类
RefinedAbstraction：扩充抽象类
Implementor：实现类接口
ConcreteImplementor：具体实现类

 

8.组合模式                                                                        

组合模式组合多个对象形成树形结构以表示“整体-部分”的结构层次。它定义了如何将容器对象和叶子对象进行递归组合，使得客户在使用的过程中无须进行区分，可以对他们进行一致的处理。在使用组合模式中需要注意一点也是组合模式最关键的地方：叶子对象和组合对象实现相同的接口。这就是组合模式能够将叶子节点和对象节点进行一致处理的原因。

虽然组合模式能够清晰地定义分层次的复杂对象，也使得增加新构件也更容易，但是这样就导致了系统的设计变得更加抽象，如果系统的业务规则比较复杂的话，使用组合模式就有一定的挑战了。

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170929210011122-1282025445.png)

 

 **模式结构**
组合模式包含如下角色：
Component: 抽象构件
Leaf: 叶子构件
Composite: 容器构件
Client: 客户类

9.装饰模式                                                                          

我们可以通过继承和组合的方式来给一个对象添加行为，虽然使用继承能够很好拥有父类的行为，但是它存在几个缺陷：一、对象之间的关系复杂的话，系统变得复杂不利于维护。二、容易产生“类爆炸”现象。三、是静态的。在这里我们可以通过使用装饰者模式来解决这个问题。

装饰者模式，动态地将责任附加到对象上。若要扩展功能，装饰者提供了比继承更加有弹性的替代方案。虽然装饰者模式能够动态将责任附加到对象上，但是他会产生许多的细小对象，增加了系统的复杂度。

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170929210140794-1843692006.png)

**模式结构**
装饰模式包含如下角色：
Component: 抽象构件
ConcreteComponent: 具体构件
Decorator: 抽象装饰类
ConcreteDecorator: 具体装饰类

 

10.外观模式                                                                         

我们都知道类与类之间的耦合越低，那么可复用性就越好，如果两个类不必彼此通信，那么就不要让这两个类发生直接的相互关系，如果需要调用里面的方法，可以通过第三者来转发调用。外观模式非常好的诠释了这段话。外观模式提供了一个统一的接口，用来访问子系统中的一群接口。它让一个应用程序中子系统间的相互依赖关系减少到了最少，它给子系统提供了一个简单、单一的屏障，客户通过这个屏障来与子系统进行通信。通过使用外观模式，使得客户对子系统的引用变得简单了，实现了客户与子系统之间的松耦合。但是它违背了“开闭原则”，因为增加新的子系统可能需要修改外观类或客户端的源代码。

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170929210303044-1520176505.png)

外观模式包含如下角色：
Facade: 外观角色
SubSystem:子系统角色

11.亨元模式                                                                         

在一个系统中对象会使得内存占用过多，特别是那些大量重复的对象，这就是对系统资源的极大浪费。享元模式对对象的重用提供了一种解决方案，它使用共享技术对相同或者相似对象实现重用。享元模式就是运行共享技术有效地支持大量细粒度对象的复用。系统使用少量对象,而且这些都比较相似，状态变化小，可以实现对象的多次复用。这里有一点要注意：享元模式要求能够共享的对象必须是细粒度对象。享元模式通过共享技术使得系统中的对象个数大大减少了，同时享元模式使用了内部状态和外部状态，同时外部状态相对独立，不会影响到内部状态，所以享元模式能够使得享元对象在不同的环境下被共享。同时正是分为了内部状态和外部状态，享元模式会使得系统变得更加复杂，同时也会导致读取外部状态所消耗的时间过长。

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170929210423544-273577223.png)

享元模式包含如下角色：
Flyweight: 抽象享元类
ConcreteFlyweight: 具体享元类
UnsharedConcreteFlyweight: 非共享具体享元类
FlyweightFactory: 享元工厂类

12.代理模式                                                                        

 代理模式就是给一个对象提供一个代理，并由代理对象控制对原对象的引用。它使得客户不能直接与真正的目标对象通信。代理对象是目标对象的代表，其他需要与这个目标对象打交道的操作都是和这个代理对象在交涉。

代理对象可以在客户端和目标对象之间起到中介的作用，这样起到了的作用和保护了目标对象的，同时也在一定程度上面减少了系统的耦合度。

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170929210617169-1348842872.png)

代理模式包含如下角色：
 Subject: 抽象主题角色
 Proxy: 代理主题角色
 RealSubject: 真实主题角色

13.访问者模式                                                                      

访问者模式俗称23大设计模式中最难的一个。除了结构复杂外，理解也比较难。在我们软件开发中我们可能会对同一个对象有不同的处理，如果我们都做分别的处理，将会产生灾难性的错误。对于这种问题，访问者模式提供了比较好的解决方案。访问者模式即表示一个作用于某对象结构中的各元素的操作，它使我们可以在不改变各元素的类的前提下定义作用于这些元素的新操作。

访问者模式的目的是封装一些施加于某种数据结构元素之上的操作，一旦这些操作需要修改的话，接受这个操作的数据结构可以保持不变。为不同类型的元素提供多种访问操作方式，且可以在不修改原有系统的情况下增加新的操作方式。同时我们还需要明确一点那就是访问者模式是适用于那些数据结构比较稳定的，因为他是将数据的操作与数据结构进行分离了，如果某个系统的数据结构相对稳定，但是操作算法易于变化的话，就比较适用适用访问者模式，因为访问者模式使得算法操作的增加变得比较简单了。

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170929210854090-2120695915.png)

访问者模式包含如下角色：
Vistor: 抽象访问者
ConcreteVisitor: 具体访问者
Element: 抽象元素
ConcreteElement: 具体元素 
ObjectStructure: 对象结构

14.模板模式                                                                        

有些时候我们做某几件事情的步骤都差不多，仅有那么一小点的不同，在软件开发的世界里同样如此，如果我们都将这些步骤都一一做的话，费时费力不讨好。所以我们可以将这些步骤分解、封装起来，然后利用继承的方式来继承即可，当然不同的可以自己重写实现嘛！这就是模板方法模式提供的解决方案。

所谓模板方法模式就是在一个方法中定义一个算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以在不改变算法结构的情况下，重新定义算法中的某些步骤。

模板方法模式就是基于继承的代码复用技术的。在模板方法模式中，我们可以将相同部分的代码放在父类中，而将不同的代码放入不同的子类中。也就是说我们需要声明一个抽象的父类，将部分逻辑以具体方法以及具体构造函数的形式实现，然后声明一些抽象方法让子类来实现剩余的逻辑，不同的子类可以以不同的方式来实现这些逻辑。所以模板方法的模板其实就是一个普通的方法，只不过这个方法是将算法实现的步骤封装起来的。

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170929211025028-883324233.png)

模板方法模式包含如下角色：
AbstractClass: 抽象类 
ConcreteClass: 具体子类

15.策略模式                                                                       

 我们知道一件事可能会有很多种方式来实现它，但是其中总有一种最高效的方式，在软件开发的世界里面同样如此，我们也有很多中方法来实现一个功能，但是我们需要一种简单、高效的方式来实现它，使得系统能够非常灵活，这就是策略模式。

所以策略模式就是定义了算法族，分别封装起来，让他们之前可以互相转换，此模式然该算法的变化独立于使用算法的客户。

在策略模式中它将这些解决问题的方法定义成一个算法群，每一个方法都对应着一个具体的算法，这里的一个算法我就称之为一个策略。虽然策略模式定义了算法，但是它并不提供算法的选择，即什么算法对于什么问题最合适这是策略模式所不关心的，所以对于策略的选择还是要客户端来做。客户必须要清楚的知道每个算法之间的区别和在什么时候什么地方使用什么策略是最合适的，这样就增加客户端的负担。

同时策略模式也非常完美的符合了“开闭原则”，用户可以在不修改原有系统的基础上选择算法或行为，也可以灵活地增加新的算法或行为。但是一个策略对应一个类将会是系统产生很多的策略类。

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170929211201231-1986841784.png)

策略模式包含如下角色：
Context: 环境类
Strategy: 抽象策略类
ConcreteStrategy: 具体策略类

16.状态模式                                                                        

 在很多情况下我们对象的行为依赖于它的一个或者多个变化的属性，这些可变的属性我们称之为状态，也就是说行为依赖状态，即当该对象因为在外部的互动而导致他的状态发生变化，从而它的行为也会做出相应的变化。对于这种情况，我们是不能用行为来控制状态的变化，而应该站在状态的角度来思考行为，即是什么状态就要做出什么样的行为。这个就是状态模式。

所以状态模式就是允许对象在内部状态发生改变时改变它的行为，对象看起来好像修改了它的类。

在状态模式中我们可以减少大块的if…else语句，它是允许态转换逻辑与状态对象合成一体，但是减少if…else语句的代价就是会换来大量的类，所以状态模式势必会增加系统中类或者对象的个数。

同时状态模式是将所有与某个状态有关的行为放到一个类中，并且可以方便地增加新的状态，只需要改变对象状态即可改变对象的行为。但是这样就会导致系统的结构和实现都会比较复杂，如果使用不当就会导致程序的结构和代码混乱，不利于维护。

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170929211342028-1513098324.png)

 

 状态模式包含如下角色：
Context: 环境类
State: 抽象状态类
ConcreteState: 具体状态类

17.观察者模式                                                                      

何谓观察者模式？观察者模式定义了对象之间的一对多依赖关系，这样一来，当一个对象改变状态时，它的所有依赖者都会收到通知并且自动更新。

在这里，发生改变的对象称之为观察目标，而被通知的对象称之为观察者。一个观察目标可以对应多个观察者，而且这些观察者之间没有相互联系，所以么可以根据需要增加和删除观察者，使得系统更易于扩展。所以观察者提供了一种对象设计，让主题和观察者之间以松耦合的方式结合。

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170929211501637-186121583.png)

 

 观察者模式包含如下角色：
Subject: 目标
ConcreteSubject: 具体目标
Observer: 观察者
ConcreteObserver: 具体观察者

18.备忘录模式                                                                     

 后悔药人人都想要，但是事实却是残酷的，根本就没有后悔药可买，但是也不仅如此，在软件的世界里就有后悔药！备忘录模式就是一种后悔药，它给我们的软件提供后悔药的机制，通过它可以使系统恢复到某一特定的历史状态。

所谓备忘录模式就是在不破坏封装的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态，这样可以在以后将对象恢复到原先保存的状态。它实现了对信息的封装，使得客户不需要关心状态保存的细节。保存就要消耗资源，所以备忘录模式的缺点就在于消耗资源。如果类的成员变量过多，势必会占用比较大的资源，而且每一次保存都会消耗一定的内存。

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170929211612762-15763568.png)

 

备忘录模式包含如下角色：
Originator: 原发器
Memento: 备忘录
Caretaker: 负责人

19.中介者模式                                                                     

 租房各位都有过的经历吧！在这个过程中中介结构扮演着很重要的角色，它在这里起到一个中间者的作用，给我们和房主互相传递信息。在外面软件的世界里同样需要这样一个中间者。在我们的系统中有时候会存在着对象与对象之间存在着很强、复杂的关联关系，如果让他们之间有直接的联系的话，必定会导致整个系统变得非常复杂，而且可扩展性很差！在前面我们就知道如果两个类之间没有不必彼此通信，我们就不应该让他们有直接的关联关系，如果实在是需要通信的话，我们可以通过第三者来转发他们的请求。同样，这里我们利用中介者来解决这个问题。

所谓中介者模式就是用一个中介对象来封装一系列的对象交互，中介者使各对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互。在中介者模式中，中介对象用来封装对象之间的关系，各个对象可以不需要知道具体的信息通过中介者对象就可以实现相互通信。它减少了对象之间的互相关系，提供了系统可复用性，简化了系统的结构。

 在中介者模式中，各个对象不需要互相知道了解，他们只需要知道中介者对象即可，但是中介者对象就必须要知道所有的对象和他们之间的关联关系，正是因为这样就导致了中介者对象的结构过于复杂，承担了过多的职责，同时它也是整个系统的核心所在，它有问题将会导致整个系统的问题。所以如果在系统的设计过程中如果出现“多对多”的复杂关系群时，千万别急着使用中介者模式，而是要仔细思考是不是您设计的系统存在问题。

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170929211741247-840833944.png)

Mediator: 抽象中介者
ConcreteMediator: 具体中介者
Colleague: 抽象同事类
ConcreteColleague: 具体同事类

20.迭代器模式                                                                     

对于迭代在编程过程中我们经常用到，能够游走于聚合内的每一个元素，同时还可以提供多种不同的遍历方式，这就是迭代器模式的设计动机。在我们实际的开发过程中，我们可能会需要根据不同的需求以不同的方式来遍历整个对象，但是我们又不希望在聚合对象的抽象接口中充斥着各种不同的遍历操作，于是我们就希望有某个东西能够以多种不同的方式来遍历一个聚合对象，这时迭代器模式出现了。

何为迭代器模式？所谓迭代器模式就是提供一种方法顺序访问一个聚合对象中的各个元素，而不是暴露其内部的表示。迭代器模式是将迭代元素的责任交给迭代器，而不是聚合对象，我们甚至在不需要知道该聚合对象的内部结构就可以实现该聚合对象的迭代。

通过迭代器模式，使得聚合对象的结构更加简单，它不需要关注它元素的遍历，只需要专注它应该专注的事情，这样就更加符合单一职责原则了。

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170929211907231-1120239995.png)

迭代器模式包含如下角色：
Iterator: 抽象迭代器
ConcreteIterator: 具体迭代器
Aggregate: 抽象聚合类
ConcreteAggregate: 具体聚合类

21.解释器模式                                                                     

所谓解释器模式就是定义语言的文法，并且建立一个解释器来解释该语言中的句子。解释器模式描述了如何构成一个简单的语言解释器，主要应用在使用面向对象语言开发的编译器中。它描述了如何为简单的语言定义一个文法，如何在该语言中表示一个句子，以及如何解释这些句子。

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170929212030919-2061065218.png)

解释器模式包含如下角色：
AbstractExpression: 抽象表达式
TerminalExpression: 终结符表达式
NonterminalExpression: 非终结符表达式
Context: 环境类
Client: 客户类

22.命令模式                                                                      

 有些时候我们想某个对象发送一个请求，但是我们并不知道该请求的具体接收者是谁，具体的处理过程是如何的，们只知道在程序运行中指定具体的请求接收者即可，对于这样将请求封装成对象的我们称之为命令模式。所以命令模式将请求封装成对象，以便使用不同的请求、队列或者日志来参数化其他对象。同时命令模式支持可撤销的操作。

命令模式可以将请求的发送者和接收者之间实现完全的解耦，发送者和接收者之间没有直接的联系，发送者只需要知道如何发送请求命令即可，其余的可以一概不管，甚至命令是否成功都无需关心。同时我们可以非常方便的增加新的命令，但是可能就是因为方便和对请求的封装就会导致系统中会存在过多的具体命令类。

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170929212209419-2076536302.png)

命令模式包含如下角色：
Command: 抽象命令类
ConcreteCommand: 具体命令类
Invoker: 调用者
Receiver: 接收者
Client:客户类

23.责任链模式                                                                    

职责链模式描述的请求如何沿着对象所组成的链来传递的。它将对象组成一条链，发送者将请求发给链的第一个接收者，并且沿着这条链传递，直到有一个对象来处理它或者直到最后也没有对象处理而留在链末尾端。

避免请求发送者与接收者耦合在一起，让多个对象都有可能接收请求，将这些对象连接成一条链，并且沿着这条链传递请求，直到有对象处理它为止，这就是职责链模式。在职责链模式中，使得每一个对象都有可能来处理请求，从而实现了请求的发送者和接收者之间的解耦。同时职责链模式简化了对象的结构，它使得每个对象都只需要引用它的后继者即可，而不必了解整条链，这样既提高了系统的灵活性也使得增加新的请求处理类也比较方便。但是在职责链中我们不能保证所有的请求都能够被处理，而且不利于观察运行时特征。

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170929212323622-1583713986.png)

 

职责链模式包含如下角色：
Handler: 抽象处理者
ConcreteHandler: 具体处理者
Client: 客户类

**五、如何学习设计模式**                                                              

**说明，《如何学习设计模式》转摘自：http://blog.csdn.net/yqj2065/article/details/39103857**

### ①  学习技巧

学习设计模式时，有一些技巧能够帮助你快速理解设计模式。

a)  使用较简单的**面向对象的语言**如Java、C#。GoF的[设计模式]实质上是**面向对象的设计模式**。[GoF·1.1]中提到“程序设计语言的选择非常重要，它将影响人们理解问题的出发点”。从学习设计模式的角度看，Java和C#较C++更容易一些。比如Java接口等，能够更有效展现设计模式的意图。

b)  使用**工具BlueJ**。BlueJ最大的好处，就是提供了简单的类图。正如我在[简明设计模式Java](http://blog.csdn.net/column/details/dp-in-java-yqj2065.html)中所做的，较少去专门画类图，而是在BlueJ中截图。在学生上机编写演示程序时，常常先看他的类图，以判断他的程序是否正确，必要时再看源代码。

c)  日常生活的**隐喻**。用一些实际生活中的例子来说明某某模式，能够让你快速掌握某模式的目的和实现代码的结构。同时，你要认识到，这种隐喻如同告诉你（2+3）2=22+2*2*3+32，你需要自己举一反三，得出（a+b）2=a2+2ab+b2。在实际工作中的模式的具体应用，则相当于应用代数公式。

d)   **动手实践和怀疑精神**。看**显浅的参考书或上网查阅资料**时，要自己敲(复制也可以)代码并运行，要多修改别人的源代码提出自己的观点：为什么书中不这样设计、为什么要那样设计；如果增添一些方法、方法参数、或成员变量会如何？必须要自己亲自动手，最起码要运行。另外，要敢于向博主提问、拍砖。你甚至可以质疑GoF的某些章节的解说和意图，更何况一些博主呢。

### ②  基础知识

这些知识让你知道，设计模式好在何处。

a)  **面向对象范式**。也就是人们传说的思想。封装、继承和多态这些东西，在我看来比if、for等稍微高一点，也属于语法问题。面向对象编程要掌握的[三大原则](http://blog.csdn.net/yqj2065/article/details/8502681)是**柏拉图(Plato)原则、里氏(Liskov)替换原则和Parnas原则**。这三个原则其实非常简单。任何原则，你觉得很难一见钟情，很难快速认同，那它就不会是好原则。

b)  **设计原则。**许多人列举了7大原则，如单一职责原则、开闭原则、里氏代换原则、依赖倒转原则、接口隔离原则、合成复用原则、迪米特法则。LSP，我将它提升为面向对象范式的3大基石之一；单一职责和接口隔离，主要作为面向对象分析/OOA时职责划分所遵循的原则，此时你可以不太在意。依赖倒转原则，我把它作为垃圾扔掉，因为**开闭原则或者直接地说“依赖于抽象类型原则”**已经包含了依赖倒转原则的精华，而依赖倒转原则的糟粕由IoC继承。当然，[回调](http://blog.csdn.net/yqj2065/article/details/8758101)，我很强调。所以，你要掌握的有**抽象依赖原则(OCP)、单向依赖原则(含对回调的学习)和最低依赖原则(合成复用原则、迪米特法则)**。

c)   UML的初步了解。这是学习设计模式的工具。在早期，你甚至可以仅了解BlueJ的相关图示，也就10分钟的事情。

### ③  境界

《五灯会元》卷十七中，有一则唐朝禅师青原惟信禅师的语录:“老僧三十年前未参禅时，见山是山，见水是水。及至后来亲见知识，有个入处，见山不是山，见水不是水。而今得个休歇处，依前见山只是山，见水只是水。”

a)   仔细研究GoF的[设计模式]，逐个学习其**意图和结构**，是一个抱着字典学习英语的方式。见山是山，见水是水，导致你可能在实际工作中生搬硬套、东施效颦。

b)   建议从简单的场景出发，**自己发现或设计出某种模式**。你从中体会该模式是如何解决问题的，这样，该模式成为你自己的东西，你才不会出现**知易行难**的问题。所有的设计模式不过是基本原则和理念在特定场合的应用。你可能不知道某个设计模式的名字，但是你知道它一切的优缺点和变体以及应用场合。见山不是山，见水不是水。

c)   你对**基本原则和理念**融会贯通，你可以惋惜：“我找到一种模式，原来在[设计模式](其实是某个特殊的书、文章提到的模式)中早就有了这种模式”。这时，模式不模式又如何呢？反模式又怎样。看见一个模式，你会说：“嗯，这是一种有用的模式”。见山只是山，见水只是水。

以上一点浅见。

------

注：【】中的内容是我加的。

1 转录【IT168知识库】

​     发现很多初学设计模式的人都有一些特点就是学习了某个设计模式之后，**貌似理解了，但是却不知道怎么去使用**这些所谓的精华经验，苦于不知如何下手。我最初学习设计模式的时候也有类似的经验，我将我的经验分享出来，希望能对初学者有帮助。
​    我对设计模式产生兴趣是在大概一年以前，最初看书的时候好像是看懂了，大概知道他在说什么。看了几个模式之后就开始寻找时机来套用套用这些模式。结果是除了Singleton模式以外的其他模式都没有找到应用的场所。然后我就没开始看下去了，我知道再看也没用，但是我并没有放弃对设计模式的关注。
​    不久我就在MSDN的Webcast上看到李建忠的 C#面向对象设计模式纵横谈讲座，很不错的系列讲座，让我对设计模式有了新的理解。我**意识到学习设计模式，确切的讲是学习面向对象的设计模式，应该从学习面向对象开始**。【**面向对象的原理如同了解下象棋的规则，而设计模式相当于残局，不知道规则看什么残局**】由于之前一年都在做asp.net开发，虽然都是在写类、学着duwamish搞分层架构、搞类型化DataSet、也弄过自定义实体类，但好像一年下来还没怎么用过接口，什么多态也是极少用。事实上对面向对象的编程思想的认识还是很模糊的。
​    重新认识OO：面向对象编程是一种思想，以面向对象的思维来思考软件设计结构，从而强化面向对象的编程范式。面向对象的特点是封装，继承，多态【**这些也算？**】。所以从那是开始，当我设计一个类的时候，不断的提示自己以下三点：
第一：别把自己的数据公开，除非你要向别人提供数据，使用尽量低的访问权限。
第二：以一个外部的视角来看类，紧记不要要求别人要在知道你是怎么实现一个方法之后才能使用我的类。
第三：分清类的职责，该这个类做的事情就要在这个类中实现，不该我的类做的事情就让别的类去实现。
在这三点的指导下来写类，写程序开始像在做“设计”了^_^。
一段时间后对设计模式就慢慢有感觉了，并能够找到一些设计模式的应用场景了。并常套用套用那些模式，逐渐的加深对模式的理解，并把它变成自己的东西，能够在其他的地方灵活的用起来。

\2. 转录 《易学设计模式·1.4  如何学习设计模式》郭志学 人民邮电出版社

如何学习设计模式
在了解了设计模式的历史和分类后，应该如何学习设计模式呢？在学习设计模式之前，读者一定要树立一种意识，那就是：**设计模式并不只是一种方法和技术，它更是一种思想、一个方法论。**它和具体的语言没有关系，学习设计模式**最主要的目的就是要建立**面向对象的思想，尽可能地面向接口编程、低耦合、高内聚，使你设计的程序尽可能地复用。【**似是而非。学习设计模式能够更好理解面向对象的思想，设计模式是一些设计的技巧和窍门，不要上升到思想、方法论好不好**】
有些软件开发人员，在程序设计时，总想着往某个设计模式上套，其实这样是不对的，并没有真正掌握设计模式的思想。其实很多时候读者用了某种设计模式，只是自己不知道这个模式叫什么名字而已。因此，在程序设计时，要根据自己的理解，使用合适的设计模式。
而有另外一些软件开发人员，在程序设计时，动不动就给类起个类似模式的名字，比如叫某某Facade、某某Factory等，其实类里面的内容和设计模式根本没有一点关系，只是用来标榜自己懂设计模式而已。
因此，学习设计模式，首先要了解有哪些方面的设计模式可以供开发人员使用，然后再分别研究每个设计模式的原理，使用时机和方法，也就是说要在什么情况下才使用某个设计模式，在了解某个设计模式的使用时机时，还要了解此时如果不使用这个设计模式，会造成什么样的后果。当对每个模式的原理和使用方法都了解了**以后**，更重要的是，学习面向对象的思想方式，在掌握面向对象的思想方式后，再回过头来看设计模式，就会有更深刻的理解，**最后，学习设计模式，一定要勤学多练**。【就最后一句很赞同】

 

***\*六、个人感悟\****                                                         

学习设计模式确实有几种境界：

第一种是学习了一两个设计模式，就一直想用到自己的代码中去；

第二种是学完全部设计模式，觉得很多模式都很相似，分不清楚它们之间有什么区别；

第三种是灵活运用设计模式，就算不用具体哪种模式也可以设计也高质量的代码，无剑胜有剑。

最后附上总结图：

![img](https://images2017.cnblogs.com/blog/401339/201709/401339-20170929215000231-1094575081.png)



# [1、简单工厂模式](https://www.cnblogs.com/xiaobai1226/p/8477059.html)

　　简单工厂模式是属于创建型模式，又叫做静态工厂方法（Static Factory Method）模式，但不属于23种GOF设计模式之一。简单工厂模式是由一个工厂对象决定创建出哪一种产品类的实例，简单来说就是，通过专门定义一个类来负责创建其他类的实例，被创建的实例通常都具有共同的父类。简单工厂模式是工厂模式家族中最简单实用的模式，可以理解为是不同工厂模式的一个特殊实现。

　　首先举个例子：创建两个类，一个Apple，一个Banana，都有个一方法

```
1 public class Apple{
2     public void get(){
3         System.out.println("采集苹果");
4     }
5 }
1 public class Banana{
2     public void get(){
3         System.out.println("采集香蕉");
4     }
5 }
```

 　在创建一个主方法来调用这两个类中的方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         //实例化Apple
 4         Apple apple = new Apple();
 5         //实例化Banana
 6         Banana banana = new Banana();
 7         
 8         apple.get();
 9         banana.get();
10     }
11 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　执行后，输出结果为：

　  　　采集苹果
　 　 采集香蕉

　　但是代码还可以改进，可以看出Apple与Banana的功能类似，所以可以抽取出一个接口，并让Apple与Banana实现此接口

```
1 public interface Fruit {
2     public void get();
3 }
1 public class Apple implements Fruit{
2     public void get(){
3         System.out.println("采集苹果");
4     }
5 }
1 public class Banana implements Fruit{
2     public void get(){
3         System.out.println("采集香蕉");
4     }
5 }
```

 　这时，主方法调用形式就变成了这样

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         //实例化一个Apple，用到了多态
 4         Fruit apple = new Apple();
 5         //实例化一个Banana，用到了多态
 6         Fruit banana= new Banana();
 7         
 8         apple.get();
 9         banana.get();
10     }
11 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这样还是刚才的结果，不同的代码达到了相同的目的。

　　而简单工厂模式则是通过专门定义一个类来负责创建其他类的实例，被创建的实例通常都具有共同的父类，上面的代码可以看出Apple与Banana实现了同一个接口，所以我们还需要创建一个工厂类来专门创建Apple与Banana的实例，继续改进

　　创建一个工具类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class FruitFactory {
 2     //获取Apple的实例，用static修饰，方便使用
 3     public static Fruit getApple(){
 4         return new Apple();
 5     }
 6     
 7     //获取Banana的实例，用static修饰，方便使用
 8     public static Fruit getBanana(){
 9         return new Banana();
10     }
11 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　Apple类，Banana类与Fruit接口没有变化

　　主方法修改为

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         //实例化一个Apple，用到了工厂类
 4         Fruit apple = FruitFactory.getApple();
 5         //实例化一个Banana，用到了工厂类
 6         Fruit banana= FruitFactory.getBanana();
 7         
 8         apple.get();
 9         banana.get();
10     }
11 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　同样和之前是一样的结果

　　这就是一个简单工厂模式的基本使用了，但这样的工厂类还不够好，例子中只有两个实例对象，但如果例子多了以后，工厂类就会产生很多很多的get方法。所以进行如下优化

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class FruitFactory {
 2     public static Fruit getFruit(String type) throws InstantiationException, IllegalAccessException{
 3         //不区分大小写
 4         if(type.equalsIgnoreCase("Apple")){
 5             return Apple.class.newInstance();
 6         }else if(type.equalsIgnoreCase("Banana")){
 7             return Banana.class.newInstance();
 8         }else{
 9             System.out.println("找不到相应的实体类");
10             return null;
11         }
12     }
13 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　这时主方法修改为：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) throws InstantiationException, IllegalAccessException {
 3         //实例化一个Apple，用到了工厂类
 4         Fruit apple = FruitFactory.getFruit("apple");
 5         //实例化一个Banana，用到了工厂类
 6         Fruit banana= FruitFactory.getFruit("banana");
 7         apple.get();
 8         banana.get();
 9     }
10 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这样就可以根据传入的参数动态的创建实例对象，而且传入的参数还可以自定义，非常的灵活，但缺点也很明显，工厂类中有大量的判断

　　还有另外一种方式

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class FruitFactory {
    public static Fruit getFruit(String type) throws ClassNotFoundException, InstantiationException, IllegalAccessException{
        Class fruit = Class.forName(type);
        return (Fruit) fruit.newInstance();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　主方法修改为

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class MainClass {
    public static void main(String[] args) throws InstantiationException, IllegalAccessException, ClassNotFoundException {
        //实例化一个Apple，用到了工厂类
        Fruit apple = FruitFactory.getFruit("Apple");
        //实例化一个Banana，用到了工厂类
        Fruit banana= FruitFactory.getFruit("Banana");
        apple.get();
        banana.get();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这种方法可以看到，工厂类非常的简洁，但主方法在调用时，输入的参数就固定了，必须为实例类名，不像上一种方法那么灵活。

　　这两种方法各有各的优点，可根据实际情况自己选择。

 

　　下面介绍一下简单工厂模式中的角色与职责

### 简单工厂模式中包含的角色及其职责

 　1、工厂（Creater）角色

　　　　简单工厂模式的核心，它负责实现创建所有实例的内部逻辑。工厂类可以被外界直接调用，创建所需的产品对象。（FruitFactory类）

　　 2、抽象（Product）角色

　　　　简单工厂模式所创建的所有对象的父类，它负责描述所有实例所共有的公共接口。（Fruit接口）

　　 3、具体产品（Concrete Product）角色

　　　　简单工厂模式所创建的具体实例对象。（Apple类与Banana类）

 

### 简单设计模式的优缺点：

- 优点：简单工厂模式中，工厂类是整个模式的关键所在。它包含必要的判断逻辑，能够根据外界给定的信息，决定究竟应该创建哪个具体类的对象。用户在创建时可以直接使用工厂类去创建所需的实例，而无需去了解这些对象是如何创建以及如何组织的，明确区分了各自的职责和权力，有利于整个软件体系结构的优化。
- 缺点：很明显简单工厂模式的缺点也体现在其工厂类上，由于工厂类集中了所有实例的创建逻辑，容易违反GRASPR的高内聚的责任分配原则，另外，当系统中的具体产品类不断增多时，可能会出现要求更改相应工厂类的情况，拓展性并不是很好。

　　

### 使用场景

　　　1、工厂类负责创建的对象比较少；

　　　2、客户只知道传入工厂类的参数，对于如何创建对象（逻辑）不关心；



# [2、工厂方法模式](https://www.cnblogs.com/xiaobai1226/p/8482256.html)

　　再看工厂方法模式之前先看看[简单工厂模式](http://www.cnblogs.com/xiaobai1226/p/8477059.html)

　　工厂方法模式（FACTORY METHOD）同样属于一种常用的对象创建型设计模式，又称为多态工厂模式,此模式的核心精神是封装类中不变的部分，提取其中个性化善变的部分为独立类，通过依赖注入以达到解耦、复用和方便后期维护拓展的目的。它的核心结构有四个角色，分别是抽象工厂，具体工厂，抽象产品，具体产品。

　　工厂方法(Factory Method)模式的意义是定义一个创建产品对象的工厂接口，将实际创建工作推迟到子类当中。核心工厂类不再负责产品的创建，这样核心类成为一个抽象工厂角色，仅负责具体工厂子类必须实现的接口，这样进一步抽象化的好处是使得工厂方法模式可以使系统在不修改具体工厂角色的情况下引进新的产品。

　　工厂方法模式是简单工厂模式的衍生，解决了许多简单工厂模式的问题。首先完全实现‘开－闭 原则’，实现了可扩展。其次更复杂的层次结构，可以应用于产品结果复杂的场合。

　　工厂方法模式对简单工厂模式进行了抽象。有一个抽象的Factory类（可以是抽象类和接口），这个类将不再负责具体的产品生产，而是只制定一些规范，具体的生产工作由其子类去完成。在这个模式中，工厂类和产品类往往可以依次对应。即一个抽象工厂对应一个抽象产品，一个具体工厂对应一个具体产品，这个具体的工厂就负责生产对应的产品。

　　工厂方法模式(Factory Method pattern)是最典型的模板方法模式(Template Method pattern)应用。

 

　　在上一篇[简单工厂模式](http://www.cnblogs.com/xiaobai1226/p/8477059.html)中，我们有两个具体产品，Apple与Banana，如果我们要增加新的具体工厂时。我们就需要修改已经写好的工厂。像这样

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class Pear implements Fruit{  //具体产品
2     @Override
3     public void get() {
4         System.out.println("采集梨子");
5     }
6 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class FruitFactory { //工厂
 2     public static Fruit getFruit(String type) throws InstantiationException, IllegalAccessException{
 3         //不区分大小写
 4         if(type.equalsIgnoreCase("Apple")){
 5             return Apple.class.newInstance();
 6         }else if(type.equalsIgnoreCase("Banana")){
 7             return Banana.class.newInstance();
 8         }else if(type.equalsIgnoreCase("Pear")){
 9             return Pear.class.newInstance();
10         }else{
11             System.out.println("找不到相应的实体类");
12             return null;
13         }
14     }
15 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) throws InstantiationException, IllegalAccessException, ClassNotFoundException {
 3         //实例化一个Apple，用到了工厂类
 4         Fruit apple = FruitFactory.getFruit("apple");
 5         //实例化一个Banana，用到了工厂类
 6         Fruit banana= FruitFactory.getFruit("banana");
 7         //实例化一个Pear，用到了工厂类
 8         Fruit pear= FruitFactory.getFruit("pear");
 9         apple.get();
10         banana.get();
11         pear.get();
12     }
13 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　可以看到，这样子，只要增加具体产品时，我们就要修改具体工厂，这样子并不符合开放-封闭原则。

 　所以，根据工厂方法模式我们创建一个抽象工厂

```
1 public interface FruitFactory {
2     public Fruit getFruit();
3 }
```

 

 　然后再创建相应的具体工厂实现抽象工厂

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class AppleFactory implements FruitFactory {
2     @Override
3     public Fruit getFruit() {
4         return new Apple();
5     }
6 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class BananaFactory implements FruitFactory {
2     @Override
3     public Fruit getFruit() {
4         return new Banana();
5     }
6 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

 　主方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) throws InstantiationException, IllegalAccessException, ClassNotFoundException {
 3         //获得AppleFactory
 4         FruitFactory af = new AppleFactory();
 5         //通过AppleFactory来获得Apple实例对象
 6         Fruit apple = af.getFruit();
 7         apple.get();
 8         
 9         //获得BananaFactory
10         FruitFactory bf = new BananaFactory();
11         //通过BananaFactory来获得Apple实例对象
12         Fruit banana = bf.getFruit();
13         banana.get();
14     }
15 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　可以看到，工厂方法模式，如果要新增具体产品，根本不必动原有工厂代码，只要新建一个新增产品的专属工厂，并实现抽象工厂即可。

 

### 工厂方法模式中包含的角色及其职责

　　　**1、抽象工厂(Creator)角色：（FruitFactory）**

　　　　　　是工厂方法模式的核心，与应用程序无关。任何在模式中创建的对象的工厂类必须实现这个接口。

　　　**2、具体工厂(Concrete Creator)角色：（AppleFactory、BananaFactory）**

　　　　　　这是实现抽象工厂接口的具体工厂类，包含与应用程序密切相关的逻辑，并且受到应用程序调用以创建产品对象。在上图中有两个这样的角色：BulbCreator与TubeCreator。

　　　**3、抽象产品(Product)角色：（Fruit）**

　　　　　　工厂方法模式所创建的对象的超类型，也就是产品对象的共同父类或共同拥有的接口。在上图中，这个角色是Light。

　　　**4、具体产品(Concrete Product)角色：（Apple、Banana）**

　　　　　　这个角色实现了抽象产品角色所定义的接口。某具体产品有专门的具体工厂创建，它们之间往往一一对应。

 

### 工厂方法模式与简单工厂模式比较

　　1、工厂方法模式与简单工厂模式在结构上的不同不是很明显。工厂方法类的核心是一个抽象工厂类，而简单工厂模式把核心放在一个具体类上。

　　2、工厂方法模式之所以有一个别名叫多态性工厂模式是因为具体工厂类都有共同的接口，或者有共同的抽象父类。

　　3、当系统扩展需要添加新的产品对象时，仅仅需要添加一个具体对象以及一个具体工厂对象，原有工厂对象不需要进行任何修改，也不需要修改客户端，很好的符合了开放-封闭原则。而简单工厂模式在添加新产品对象后，不得不修改工厂方法，扩展性不好。

　　4、工厂方法模式退化后可以演变成简单工厂模式。

###   应用场景

　　工厂方法经常用在以下两种情况中:

　　　　第一种情况是对于某个产品，调用者清楚地知道应该使用哪个具体工厂服务，实例化该具体工厂，生产出具体的产品来。Java Collection中的iterator() 方法即属于这种情况。

　　　　第二种情况，只是需要一种产品，而不想知道也不需要知道究竟是哪个工厂为生产的，即最终选用哪个具体工厂的决定权在生产者一方，它们根据当前系统的情况来实例化一个具体的工厂返回给使用者，而这个决策过程这对于使用者来说是透明的。



# [3、抽象工厂模式](https://www.cnblogs.com/xiaobai1226/p/8483480.html)

　　抽象工厂模式是所有形态的工厂模式中最为抽象和最具一般性的一种形态。抽象工厂模式是指当有多个抽象角色时，使用的一种工厂模式。抽象工厂模式可以向客户端提供一个接口，使客户端在不必指定产品的具体的情况下，创建多个产品族中的产品对象。

### 产品族

　　是指位于不同产品等级结构中，功能相关联的产品组成的家族。一般是位于不同的等级结构中的相同位置上。显然，每一个产品族中含有产品的数目，与产品等级结构的数目是相等的，形成一个二维的坐标系，水平坐标是产品等级结构，纵坐标是产品族。叫做相图。

　　当有多个不同的等级结构的产品时，如果使用工厂方法模式就势必要使用多个独立的工厂等级结构来对付这些产品的等级结构。如果这些产品等级结构是平行的，会导致多个平行的工厂等级结构。

　　抽象工厂模式使用同一个 工厂等级结构负责这些不同产品等级结构产品对象的创建。

　　对于每一个产品族，都有一个具体工厂。而每一个具体工厂创建属于同一个产品族，但是分属于不同等级结构的产品。

　　通过引进抽象工厂模式，可以处理具有相同（或者相似）等级结构的多个产品族中的产品对象的创建问题。

　　由于每个具体工厂角色都需要负责两个不同等级结构的产品对象的创建，因此每个工厂角色都需要提供两个工厂方法，分别用于创建两个等级结构的产品。既然每个具体工厂角色都需要实现这两个工厂方法，所以具有一般性，不妨抽象出来，移动到抽象工厂角色中加以声明。

　　就好比，水果分为

　　**北方水果**：北方苹果，北方香蕉；

　　**南方水果**：南方苹果，南方香蕉；

　　**热带水果**：热带苹果，热带香蕉；

　　这样看，北方水果，南方水果，热带水果这就是三个不同的产品族。

 

　　下面写一个简单的抽象工厂模式的小例子：

　　首先确定我们的产品族，产品族为南方水果与北方水果，而水果（产品等级）有苹果和香蕉产品等级，所以具体产品为南方苹果，北方苹果，南方香蕉，北方香蕉。

　　具体代码如下：首先每一个族中都有苹果和香蕉，所以定义两个抽象类，其中包含一个抽象方法

苹果

```
1 public abstract class Apple implements Fruit{
2     public abstract void get();
3 }
```

香蕉

```
1 public abstract class Banana implements Fruit{
2     public abstract void get();
3 }
```

　　在写苹果香蕉的具体产品，并各自继承对应的抽象类

北方苹果

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class NorthApple extends Apple {
2     @Override
3     public void get() {
4         System.out.println("采集北方苹果");
5     }
6 
7 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

南方苹果

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class SouthApple extends Apple{
    @Override
    public void get() {
        System.out.println("采集南方苹果");
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

北方香蕉

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class NorthBanana extends Banana {
    @Override
    public void get() {
        System.out.println("采集北方香蕉");
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

南方香蕉

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class SouthBanana extends Banana {
2     @Override
3     public void get() {
4         System.out.println("采集南方香蕉");
5     }
6 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　接下来创建工厂，而每一个产品族都对应一个具体的工厂，每个产品族都包含苹果和香蕉，所以每个工厂中都包含苹果和香蕉

抽象工厂

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public interface FruitFactory {
2     //实例化一个苹果
3     public Fruit getApple();
4     //实例化一个香蕉
5     public Fruit getBanana();
6 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

北方工厂

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class NorthFactory implements FruitFactory{
 2     @Override
 3     public Fruit getApple() {
 4         return new NorthApple();
 5     }
 6 
 7     @Override
 8     public Fruit getBanana() {
 9         return new NorthBanana();
10     }
11 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

南方工厂 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class SouthFactory implements FruitFactory{
 2     @Override
 3     public Fruit getApple() {
 4         return new SouthApple();
 5     }
 6 
 7     @Override
 8     public Fruit getBanana() {
 9         return new SouthBanana();
10     }
11 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　最后，写一个运行的主方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         FruitFactory nf = new NorthFactory();
 4         
 5         Fruit nApple = nf.getApple();
 6         nApple.get();
 7         
 8         Fruit nBanana = nf.getBanana();
 9         nBanana.get();
10         
11         FruitFactory sf = new SouthFactory();
12         
13         Fruit sApple = sf.getApple();
14         sApple.get();
15         
16         Fruit sBanana = sf.getBanana();
17         sBanana.get();
18     }
19 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　最终运行结果

　　　  采集北方苹果
　　　采集北方香蕉
　　　采集南方苹果
　　　采集南方香蕉

　这时如果想新增一个产品族热带水果，只需新建一个热带产品族的工厂即可，已经建好的南方与北方工厂无需改动，也符合开放-封闭原则。

　但缺点也很明显，从产品等级来看，如果想新增一个产品等级，例如上面的例子只有苹果与香蕉，如果现在新增一个葡萄，就需要在抽象工厂中添加一个葡萄抽象方法，再在每一个具体工厂中实现此方法。这样就完全不符合开放-封闭原则了。

　　**优点：**

　　　　1.它分离了具体的类

　　　　2.它使得易于交换产品系列

　　　　3.它有利于产品的一致性

　　**缺点：**

　　　　难以支持新种类的产品

 

### 抽象工厂模式中包含的角色及其职责

　　　**1、抽象工厂(Creator)角色：（FruitFactory）**

　　　　　　是抽象工厂模式的核心，包含对多个产品结构的声明，任何工厂类都必须实现这个接口。

　　　**2、具体工厂(Concrete Creator)角色：（AppleFactory、BananaFactory）**

　　　　　　这是实现抽象工厂接口的具体工厂类，负责实例化某个产品族中的产品对象。

　　　**3、抽象产品(Product)角色：（Fruit）**

　　　　　　抽象工厂模式所创建的对象的父类，它负责描述所有实例所共有的公共接口。

　　　**4、具体产品(Concrete Product)角色：（Apple、Banana）**

　　　　　　抽象模式所创建的具体实例对象。

 

 　**抽象工厂中方法对应产品结构，具体工厂对应产品族。**



# [4、单例模式](https://www.cnblogs.com/xiaobai1226/p/8487696.html)

　　单例模式是一种对象创建型模式，使用单例模式，可以保证为一个类只生成唯一的一个实例对象。也就是说，在整个程序空间中，该类只存在一个实例对象。

　　其实，GoF对单例模式的定义是：保证一个类，只有一个实例存在，同时提供能对该实例加以访问的全局访问方法。

　　

## 为什么要用单例模式呢？

　　这是因为在应用系统开发时，我们常常有以下需求：

　　1、在多个线程之间，比如servlet环境，共享同一个资源或者操作同一个对象。

　　2、在整个程序空间使用全局变量，共享资源。

　　3、在大规模系统中，为了性能的考虑，需要节省对象的创建时间等等。

　　因为单例模式可以保证为一个类只生成唯一的实例对象，所以这些情况，单例模式就派上用场了。

 

　　单例模式分为几种情况

　　**1、饿汉式（在类加载时就完成了初始化，所以类加载比较慢，但获取对象的速度快，同时无法做到延时加载）**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class Person {
    public static final Person person = new Person();

    //构造函数私有化
    private Person(){}

    //提供一个全局的静态方法
    public static Person getPerson(){
        return person;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　

　　2、接下来是**懒汉式（在类加载时不初始化，可以延时加载）**

　　　 懒汉式可以分为两种，一种线程安全，一种线程不安全

　　**（1）懒汉式（线程不安全，但效率高）**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class Person {
    public static Person person = null;

    //构造函数私有化
    private Person(){}

    //提供一个全局的静态方法
    public static Person getPerson(){
        if(person == null){
            person = new Person();
        }
        return person;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　**（2）懒汉式（线程安全，但效率低）**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class Person {
 2     public static Person person = null;
 3 
 4     //构造函数私有化
 5     private Person(){}
 6 
 7     //提供一个全局的静态方法
 8     public static synchronized Person getPerson(){
 9         if(person == null){
10             person = new Person();
11         }
12         return person;
13     }
14 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　**3、静态内部类（可以延时加载）**

　　这种方法是饿汉式的一种升级，这种方式同样利用了classloder的机制来保证初始化instance时只有一个线程，同时实现了延时加载

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class Person {
 2     private static class PersonHolder{
 3         private static final Person person = new Person();
 4     }
 5 
 6     //构造函数私有化
 7     private Person(){}
 8 
 9     //提供一个全局的静态方法  
10     public static final Person getPerson(){
11         return PersonHolder.person;
12     }
13 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　**4、双重检查（对懒汉式的升级，效率更高）**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class Person {
    public static Person person = null;

    //构造函数私有化
    private Person(){}

    //提供一个全局的静态方法
    public static Person getPerson(){
        if(person == null){
            synchronized(Person.class){
                if(person == null){
                    person = new Person();
                }
            }
        }
        return person;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　这样写，只把新建实例的代码放到同步锁中，为了保证线程安全再在同步锁中加一个判断，虽然看起来更繁琐，但是同步中的内容只会执行一次，执行过后，以后经过外层的if判断后，都不会在执行了，所以不会再有阻塞。程序运行的效率也会更加的高。



# [5、原型模式](https://www.cnblogs.com/xiaobai1226/p/8488332.html)

　　原型（Prototype）模式是一种对象创建型模式，他采取复制原型对象的方法来创建对象的实例。使用原型模式创建的实例，具有与原型一样的数据。

　　**原型模式的特点：**

　　1、由原型对象自身创建目标对象。也就是说，对象创建这一动作发自原型对象本身。

　　2、目标对象是原型对象的一个克隆。也就是说，通过原型模式创建的对象，不仅仅与原型对象具有相同的结构，还与原型对象具有相同的值。

　　3、根据对象克隆深度层次的不同，有浅度克隆与深度克隆。

 

　　先写一个支持克隆的类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 //如果要克隆就必须实现Cloneable接口
 2 public class Person implements Cloneable{
 3     //可能会抛出不支持克隆异常，原因是没有实现Cloneable接口
 4     @Override
 5     protected Person clone(){
 6         try{
 7             return (Person) super.clone();
 8         }catch(CloneNotSupportedException e){
 9             e.printStackTrace();
10             return null;
11         }
12     }
13 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这个样子，就说明这个类可以克隆了。

　　这样克隆

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class MainClass {
2     public static void main(String[] args) {
3         Person person1 = new Person();
4         
5         Person person2 = person1.clone();
6     }
7 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这样子克隆并不等同于Person p2 = p1；像Person p2 = p1；指的是在栈中创建一个变量p2，将p1的内存地址赋给p2，其实指的是同一个对象。而克隆是复制出一份一模一样的对象，两个对象内存地址不同，但对象中的结构与属性值一模一样。

　　这种不通过 new 关键字来产生一个对象，而是通过对象拷贝来实现的模式就叫做原型模式，这个模式的核心是一个clone( )方法，通过这个方法进行对象的拷贝，Java 提供了一个 Cloneable 接口来标示这个对象是可拷贝的，为什么说是“标示”呢？翻开 JDK 的帮助看看 Cloneable 是一个方法都没有的，

这个接口只是一个标记作用，在 JVM 中具有这个标记的对象才有可能被拷贝，所以覆盖了覆盖clone()方法就可以了。

　　在 clone()方法上增加了一个注解@Override， 没有继承一个类为什么可以重写呢？在 Java 中所有类的父类是Object 类，每个类默认都是继承了这个类，所以这个用上@Override是非常正确的。原型模式虽然很简单，但是在 Java 中使用原型模式也就是 clone 方法还是有一些注意事项的：　　

　　**对象拷贝时，类的构造函数是不会被执行的**。 一个实现了 Cloneable 并重写了 clone 方法的类 A,有一个无参构造或有参构造 B，通过 new 关键字产生了一个对象 S，再然后通过 S.clone()方式产生了一个新的对象 T，那么在对象拷贝时构造函数 B 是不会被执行的， 对象拷贝时确实构造函数没有被执行，这个从原理来讲也是可以讲得通的，Object 类的 clone 方法的 原理是从内存中（具体的说就是堆内存）以二进制流的方式进行拷贝，重新分配一个内存块，那构造函数 没有被执行也是非常正常的了。

　　还有就是深度克隆与浅度克隆

　　**首先，是浅度克隆**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 //如果要克隆就必须实现Cloneable接口
 2 public class Person implements Cloneable{
 3     private String name;
 4     private String sex;
 5     private List<String> list;
 6     public String getName() {
 7         return name;
 8     }
 9     public void setName(String name) {
10         this.name = name;
11     }
12     public String getSex() {
13         return sex;
14     }
15     public void setSex(String sex) {
16         this.sex = sex;
17     }
18     public List<String> getList() {
19         return list;
20     }
21     public void setList(List<String> list) {
22         this.list = list;
23     }
24     //可能会抛出不支持克隆异常，原因是没有实现Cloneable接口
25     @Override
26     protected Person clone(){
27         try{
28             return (Person) super.clone();
29         }catch(CloneNotSupportedException e){
30             e.printStackTrace();
31             return null;
32         }
33     }
34 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　　这就是浅度克隆，当被克隆的类中有引用对象**（String或Integer等包装类型除外）**时，克隆出来的类中的引用变量存储的还是之前的内存地址，也就是说克隆与被克隆的对象是同一个。这样的话两个对象共享了一个私有变量，所有人都可以改，是一个种非常不安全的方式，在实际项目中使用还是比较少的。

　　**所以就要说到深度拷贝**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 //如果要克隆就必须实现Cloneable接口
 2 public class Person implements Cloneable{
 3     private String name;
 4     private String sex;
 5     private List<String> list;
 6     public String getName() {
 7         return name;
 8     }
 9     public void setName(String name) {
10         this.name = name;
11     }
12     public String getSex() {
13         return sex;
14     }
15     public void setSex(String sex) {
16         this.sex = sex;
17     }
18     public List<String> getList() {
19         return list;
20     }
21     public void setList(List<String> list) {
22         this.list = list;
23     }
24     //可能会抛出不支持克隆异常，原因是没有实现Cloneable接口
25     @Override
26     protected Person clone(){
27         try{
28             Person person = (Person) super.clone();
29             List<String> newList = new ArrayList();
30             
31             for(String str : this.list){
32                 newList.add(str);
33             }
34             person.setList(newList);
35             return person;
36         }catch(CloneNotSupportedException e){
37             e.printStackTrace();
38             return null;
39         }
40     }
41 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这样就完成了深度拷贝，两种对象互为独立，属于单独对象。

　　**注意：final 类型修饰的成员变量不能进行深度拷贝**　　

### 原型模式的使用场景

　　1、在创建对象的时候，我们不只是希望被创建的对象继承其基类的基本结构，还希望继承原型对象的数据。

　　2、希望对目标对象的修改不影响既有的原型对象（深度克隆的时候可以完全互不影响）。

　　3、隐藏克隆操作的细节，很多时候，对对象本身的克隆需要涉及到类本身的数据细节。

　　4、类初始化需要消化非常多的资源，这个资源包括数据、硬件资源等；

　　5、通过 new 产生一个对象需要非常繁琐的数据准备或访问权限，则可以使用原型模式；

　　6、一个对象需要提供给其他对象访问，而且各个调用者可能都需要修改其值时，可以考虑使用原型模式拷贝多个对象供调用者使用。在实际项目中，原型模式很少单独出现，一般是和工厂方法模式一起出现，通过 clone的方法创建一个对象，然后由工厂方法提供给调用者。原型模式先产生出一个包含

　　大量共有信息的类，然后可以拷贝出副本，修正细节信息，建立了一个完整的个性对象。

# [6、建造者模式](https://www.cnblogs.com/xiaobai1226/p/8507239.html)

　　Builder模式也叫建造者模式或者生成器模式，是由GoF提出的23种设计模式中的一种。Builder模式是一种对象创建型模式之一，用来隐藏复合对象的创建过程，它把复合对象的创建过程加以抽象，通过子类继承和重载的方式，动态地创建具有复合属性的对象。

　　建造者模式的结构

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180305133545561-976325209.png)

 

　　**角色**

　　在这样的设计模式中，有以下几个角色：

　　　　1 builder：为创建一个产品对象的各个部件指定抽象接口。

　　　　2 ConcreteBuilder：实现Builder的接口以构造和装配该产品的各个部件，定义并明确它所创建的表示，并提供一个检索产品的接口。

　　　　3 Director：构造一个使用Builder接口的对象。

　　　　4 Product：表示被构造的复杂对象。ConcreteBuilder创建该产品的内部表示并定义它的装配过程，包含定义组成部件的类，包括将这些部件装配成最终产品的接口。

 

　　首先，举个例子，建造者模式我们比方我们要造个房子。

　　房子的图纸

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class House {
 2     //地板
 3     private String floor;
 4     //墙
 5     private String wall;
 6     //房顶
 7     private String roof;
 8     
 9     public String getFloor() {
10         return floor;
11     }
12     public void setFloor(String floor) {
13         this.floor = floor;
14     }
15     public String getWall() {
16         return wall;
17     }
18     public void setWall(String wall) {
19         this.wall = wall;
20     }
21     public String getRoof() {
22         return roof;
23     }
24     public void setRoof(String roof) {
25         this.roof = roof;
26     }
27 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　有了图纸后，最笨的方法就是自己造房子

　　客户端

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         //客户直接造房子
 4         House house = new House();
 5         
 6         house.setFloor("地板");
 7         house.setWall("墙");
 8         house.setRoof("屋顶");
 9         
10         System.out.println(house.getFloor());
11         System.out.println(house.getWall());
12         System.out.println(house.getRoof());
13     }
14 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　可是这样的方法不是很好，真正我们造房子都是找施工队，所以我们要把造房子分离出来，交给施工队

　　新建一个施工队，为了扩展性，声明一个施工队的接口

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public interface HouseBuilder {
 2     //修地板
 3     public void makeFloor();
 4     //修墙
 5     public void makeWall();
 6     //修屋顶
 7     public void makeRoof();
 8     //获得修好的房子
 9     public House getHouse();
10 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　新建一个施工队，实现此接口

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class LoufangBuilder implements HouseBuilder{
 2     House house = new House();
 3     
 4     @Override
 5     public void makeFloor() {
 6         house.setFloor("楼房->地板");
 7     }
 8 
 9     @Override
10     public void makeWall() {
11         house.setWall("楼房->墙");
12     }
13 
14     @Override
15     public void makeRoof() {
16         house.setRoof("楼房->屋顶");
17     }
18 
19     @Override
20     public House getHouse() {
21         return house;
22     }
23 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　客户端

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         //施工队造房子
 4         HouseBuilder loufangBuilder = new LoufangBuilder();
 5         loufangBuilder.makeFloor();
 6         loufangBuilder.makeWall();
 7         loufangBuilder.makeRoof();
 8         
 9         House house = loufangBuilder.getHouse();
10         System.out.println(house.getFloor());
11         System.out.println(house.getWall());
12         System.out.println(house.getRoof());
13     }
14 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　可以看到，这样子造房子就交给施工队了，但可以看到造房子的具体细节还在客户端里，如图。

　　![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180305115406310-31750419.png)

　　这就相当于我们在指导施工队干活，这肯定不是最好的方案，最好的解决方案，是由一个设计师也可以说是指挥者来指导工程队，所以在新建一个指挥者

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class HouseDirector {
 2     private HouseBuilder houseBuilder;
 3     
 4     public HouseDirector(HouseBuilder houseBuilder){
 5         this.houseBuilder = houseBuilder;
 6     }
 7     
 8     public void make(){
 9         houseBuilder.makeFloor();
10         houseBuilder.makeWall();
11         houseBuilder.makeRoof();
12     }
13 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

 　客户端

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         //施工队造房子
 4         HouseBuilder loufangBuilder = new LoufangBuilder();
 5 //        loufangBuilder.makeFloor();
 6 //        loufangBuilder.makeWall();
 7 //        loufangBuilder.makeRoof();
 8         HouseDirector houseDirector = new HouseDirector(loufangBuilder);
 9         houseDirector.make();
10         
11         House house = loufangBuilder.getHouse();
12         System.out.println(house.getFloor());
13         System.out.println(house.getWall());
14         System.out.println(house.getRoof());
15     }
16 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这样子，把施工队交给这个设计者，施工细节的工作就由这个设计者执行了。

　　当然，还有一种写法，有一些细微的改动，也是更常用的，就是设计者（Director）不在构造时传入builder，而是在调用方法时，才传入，像这样

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class HouseDirector {
2     public void make(HouseBuilder houseBuilder){
3         houseBuilder.makeFloor();
4         houseBuilder.makeWall();
5         houseBuilder.makeRoof();
6     }
7 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　客户端

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         //施工队造房子
 4         HouseBuilder loufangBuilder = new LoufangBuilder();
 5 
 6         HouseDirector houseDirector = new HouseDirector();
 7         houseDirector.make(loufangBuilder);
 8         
 9         House house = loufangBuilder.getHouse();
10         System.out.println(house.getFloor());
11         System.out.println(house.getWall());
12         System.out.println(house.getRoof());
13     }
14 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这样子，出来的效果是一样的。

　　这就是一个简单的建造者模式

　　这样也提高了系统的扩展性与可维护性，如果不想造楼房了，想造一个别墅，只需新增一个别墅施工队就好了，像这样

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class BieshuBuilder implements HouseBuilder{
 2     House house = new House();
 3     
 4     @Override
 5     public void makeFloor() {
 6         house.setFloor("别墅->地板");
 7     }
 8 
 9     @Override
10     public void makeWall() {
11         house.setWall("别墅->墙");
12     }
13 
14     @Override
15     public void makeRoof() {
16         house.setRoof("别墅->屋顶");
17     }
18 
19     @Override
20     public House getHouse() {
21         return house;
22     }
23 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　客户端只需把施工队换成别墅施工队

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class MainClass {
    public static void main(String[] args) {
        //施工队造房子
        HouseBuilder bieshuBuilder = new BieshuBuilder();//只需要修改这里

        HouseDirector houseDirector = new HouseDirector();
        houseDirector.make(bieshuBuilder);
        
        House house = bieshuBuilder.getHouse();
        System.out.println(house.getFloor());
        System.out.println(house.getWall());
        System.out.println(house.getRoof());
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### 适用范围

　　1、对象的创建：Builder模式是为对象的创建而设计的模式

　　2、创建的是一个复合对象：被创建的对象为一个具有复合属性的复合对象

　　3、关注对象创建的各部分的创建过程：不同的工厂（这里指builder生成器）对产品属性有不同的创建方法

# [7、装饰模式](https://www.cnblogs.com/xiaobai1226/p/8522992.html)

### 什么是装饰者模式？

　　装饰（ Decorator ）模式又叫做包装模式。通过一种对客户端透明的方式来扩展对象的功能，是继承关系的一个替换方案。他是23种设计模式之一，英文叫Decorator Pattern，又叫装饰者模式。装饰模式是在不必改变原类文件和使用继承的情况下，动态地扩展一个对象的功能。它是通过创建一个包装对象，也就是装饰来包裹真实的对象。

### 为什么要使用装饰模式

　　1. 多用组合，少用继承。

　　　　利用继承设计子类的行为，是在编译时静态决定的，而且所有的子类都会继承到相同的行为。然而，如果能够利用组合的做法扩展对象的行为，就可以在运行时动态地进行扩展。

　　2. 类应设计的对扩展开放，对修改关闭。

 

　　之前我们假如实现这样一个功能，建造出各种各样不同功能的车，咱们是这样实现的：

　　首先，新建一个Car接口，定义了所有车的基本功能，就是跑run()，和展示自己功能的方法show(),

```
1 public interface Car {
2     public void show();
3     public void run();
4 }
```

　　然后，在创建各个具体的车实现Car接口

　　会跑的车

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class RunCar  implements Car{
 2 
 3     @Override
 4     public void show() {
 5         this.run();
 6         
 7     }
 8 
 9     @Override
10     public void run() {
11         System.out.println("可以跑");
12         
13     }
14 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　会游泳的车

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class SwimCar implements Car {
 2 
 3     @Override
 4     public void show() {
 5         this.run();
 6         this.swim();
 7     }
 8 
 9     @Override
10     public void run() {
11         System.out.println("可以跑");
12 
13     }
14 
15     public void swim(){
16         System.out.println("可以游泳");
17     }
18 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　会飞的车

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class FlyCar implements Car {
 2 
 3     @Override
 4     public void show() {
 5         this.run();
 6         this.fly();
 7     }
 8 
 9     @Override
10     public void run() {
11         System.out.println("可以跑");
12 
13     }
14 
15     public void fly(){
16         System.out.println("可以飞");
17     }
18 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这时，写一个客户端展示每个车的功能

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class MainClass {
2     public static void main(String[] args) {
3 //        Car car = new RunCar();
4 //        Car car = new FlyCar();
5         Car car = new SwimCar();
6         car.show();
7     }
8 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　这样，每新增一种车，就要新写一个子类实现或继承其他类或接口，就相当于，每新增一种功能，就要新建一辆车。

　　这时，我们还有一种替代方案，就是使用装饰模式

　　首先，新建一个Car接口，和一个基础的Car的实现类RunCar，因为只要是车一定有跑的功能，这两个和上面一样，不在重复写了。

　　然后在新建装饰类，不同的功能建不同的装饰类

　　1、新建一个装饰类父类，实现Car接口，提供一个有参的构造方法，共有的方法show()，私有的Car成员变量，并为之提供get()，set()方法。

　　一定要继承Car，因为装饰过后，还是一辆车

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public abstract class CarDecorator implements Car{
 2     private Car car;
 3     
 4     public CarDecorator(Car car){
 5         this.car = car;
 6     }
 7     
 8     public Car getCar() {
 9         return car;
10     }
11 
12     public void setCar(Car car) {
13         this.car = car;
14     }
15 
16     public abstract void show();
17 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　2、为不同的装饰新建装饰类，并继承CarDecorator抽象类

　　（1）游泳装饰类，覆盖抽象方法，在新增特有的方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class SwimCarDecorator extends CarDecorator {
 2 
 3     public SwimCarDecorator(Car car){
 4         super(car);
 5     }
 6     
 7     @Override
 8     public void show() {
 9         this.getCar().show();
10         this.swim();
11     }
12     
13     public void swim(){
14         System.out.println("可以游泳");
15     }
16 
17     @Override
18     public void run() {
19     }
20 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　（2）飞行装饰类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class FlyCarDecorator extends CarDecorator {
 2 
 3     public FlyCarDecorator(Car car){
 4         super(car);
 5     }
 6     @Override
 7     public void show() {
 8         this.getCar().show();
 9         this.fly();
10     }
11     
12     public void fly(){
13         System.out.println("可以飞");
14     }
15     
16     @Override
17     public void run() {
18     }
19 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　添加客户端，执行

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class MainClass {
    public static void main(String[] args) {
        Car car = new RunCar();
        Car swimCar = new SwimCarDecorator(car);
        swimCar.show();
    }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这样，就不等于是，每新增一个功能就新建一辆车了，而是基础有一个RunCar，这是最基本的车，装饰类就相当于在基本的车的基础上，添加功能，装饰这台最基本的车。

　　所以一定要继承Car，因为装饰过后，还是一辆车，我们可以直接Car swimCar = new SwimCarDecorator(car);用Car来创建变量。

 

　　同样，继承关系如果每一个功能都要添加一个新的子类，如果，一辆车已经拥有了游泳和飞行的功能，这时有新增同时拥有游泳和飞行的Car，继承关系就需要在新建一个子类同时拥有这两个功能，而装饰模式什么都不需要新增，对基础的RunCar修饰两遍即可。像这样：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class MainClass {
    public static void main(String[] args) {
        Car car = new RunCar();
        Car swimCar = new SwimCarDecorator(car);
        Car flySwimCar = new FlyCarDecorator(swimCar);
        flySwimCar.show();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这样子，最后的flySwimCar就同时拥有了飞行和游泳的功能，这也是装饰类继承Car的原因，这样子装饰类才能当做参数放进构造方法中。

## 装饰模式的结构图

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180308105949662-223183950.png)

#### 装饰模式的角色与职责：

　　1、抽象组件角色**（Component）**： 一个抽象接口，是被装饰类和装饰类的父接口。**（Car）**

　　2、具体组件角色**（ConcreteComponent）**：为抽象组件的实现类。**（RunCar）**

　　3、抽象装饰角色**（Decorator）**：包含一个组件的引用，并定义了与抽象组件一致的接口。**（CarDecorator）**

　　4、具体装饰角色**（ConcreteDecorator）**：为抽象装饰角色的实现类。负责具体的装饰。**（FlyCarDecorator、SwimCarDecorator）**

### 使用范围

　　以下情况使用Decorator模式

　　1. 需要扩展一个类的功能，或给一个类添加附加职责。

　　2. 需要动态的给一个对象添加功能，这些功能可以再动态的撤销。

　　3. 需要增加由一些基本功能的排列组合而产生的非常大量的功能，从而使继承关系变的不现实。

　　4. 当不能采用生成子类的方法进行扩充时。一种情况是，可能有大量独立的扩展，为支持每一种组合将产生大量的子类，使得子类数目呈爆炸性增长。另一种情况可能是因为类定义被隐藏，或类定义不能用于生成子类。

### 装饰模式的优缺点

#### 优点

　　　　1. Decorator模式与继承关系的目的都是要扩展对象的功能，但是Decorator可以提供比继承更多的灵活性。

　　　　2. 通过使用不同的具体装饰类以及这些装饰类的排列组合，设计师可以创造出很多不同行为的组合。

#### 缺点

　　　　1. 这种比继承更加灵活机动的特性，也同时意味着更加多的复杂性。

　　　　2. 装饰模式会导致设计中出现许多小类，如果过度使用，会使程序变得很复杂。

　　　　3. 装饰模式是针对抽象组件（Component）类型编程。但是，如果你要针对具体组件编程时，就应该重新思考你的应用架构，以及装饰者是否合适。当然也可以改变Component接口，增加新的公开的行为，实现“半透明”的装饰者模式。在实际项目中要做出最佳选择。

# [8、策略模式](https://www.cnblogs.com/xiaobai1226/p/8527420.html)

　　Strategy模式也叫策略模式是行为模式之一，它对一系列的算法加以封装，为所有算法定义一个抽象的算法接口，并通过继承该抽象算法接口对所有的算法加以封装和实现，具体的算法选择交由客户端决定（策略）。Strategy模式主要用来平滑地处理算法的切换 。

　　举个例子：假如有两个加密算法，我们分别调用他们，之前我们可以这么写

先写一个算法接口

```
1 public interface Strategy {
2     //加密
3     public void encrypt();
4 }
```

再写两个对应的实现类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class MD5Strategy implements Strategy {
2 
3     @Override
4     public void encrypt() {
5         System.out.println("执行MD5加密");
6     }
7 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class RSAStrategy implements Strategy{
2 
3     @Override
4     public void encrypt() {
5         System.out.println("执行RSA加密");
6     }
7 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

最后，写一个主函数调用

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class MainClass {
2     public static void main(String[] args) {
3         Strategy md5 = new MD5Strategy();
4         md5.encrypt();
5         
6         Strategy rsa = new RSAStrategy();
7         rsa.encrypt();
8     }
9 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　这样使我们使用传统继承关系方式来实现此功能，接下来使用策略模式

　　首先、新建一个Context类，这个类是策略模式的核心类，类似于一个工厂，它包含了所有算法类的所有方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class Context {
 2     private Strategy strategy;
 3     
 4     public Context(Strategy stratrgy){
 5         this.strategy = stratrgy;
 6     }
 7     
 8     public void encrypt(){
 9         this.strategy.encrypt();
10     }
11 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　两个算法类都不用改变，主函数调用改为

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class MainClass {
2     public static void main(String[] args) {
3         Context md5 = new Context(new MD5Strategy());
4         md5.encrypt();
5         
6         Context rsa = new Context(new RSAStrategy());
7         rsa.encrypt();
8     }
9 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这样子，用户就只需要关心context即可，也不必关心算法的具体实现，这就是一个简单了策略模式例子

### 策略模式的结构

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180308135318564-961228216.png)

 

 

### 策略模式的角色与职责

　　　　**1、Strategy:** 策略（算法）抽象。（Strategy接口）

　　　　**2、ConcreteStrategy：**各种策略（算法）的具体实现。（MD5Strategy、RSAStrategy）

　　　　3、**Context：**策略的外部封装类，或者说策略的容器类。根据不同策略执行不同的行为。策略由外部环境决定。（Context）

 

### 策略模式的优缺点：　

#### 优点：

　　1. 策略模式提供了管理相关的算法族的办法。策略类的等级结构定义了一个算法或行为族。恰当使用继承可以把公共的代码移到父类里面，从而避免重复的代码。

　　2. 策略模式提供了可以替换继承关系的办法。继承可以处理多种算法或行为。如果不是用策略模式，那么使用算法或行为的环境类就可能会有一些子类，每一个子类提供一个不同的算法或行为。但是，这样一来算法或行为的使用者就和算法或行为本身混在一起。决定使用哪一种算法或采取哪一种行为

的逻辑就和算法或行为的逻辑混合在一起，从而不可能再独立演化。继承使得动态改变算法或行为变得不可能。

　　3. 使用策略模式可以避免使用多重条件转移语句。多重转移语句不易维护，它把采取哪一种算法或采取哪一种行为的逻辑与算法或行为的逻辑混合在一起，统统列在一个多重转移语句里面，比使用继承的办法还要原始和落后。

#### 缺点：

　　1.客户端必须知道所有的策略类，并自行决定使用哪一个策略类。这就意味着客户端必须理解这些算法的区别，以便适时选择恰当的算法类。换言之，策略模式只适用于客户端知道所有的算法或行为的情况。

　　2. 策略模式造成很多的策略类。有时候可以通过把依赖于环境的状态保存到客户端里面，而将策略类设计成可共享的，这样策略类实例可以被不同客户端使用。换言之，可以使用享元模式来减少对象的数量。

### 应用场景：　

　　1、 多个类只区别在表现行为不同，可以使用Strategy模式，在运行时动态选择具体要执行的行为。

　　2、 需要在不同情况下使用不同的策略(算法)，或者策略还可能在未来用其它方式来实现。

　　3、 对客户隐藏具体策略(算法)的实现细节，彼此完全独立。

# [9、观察者模式](https://www.cnblogs.com/xiaobai1226/p/8528322.html)

　　Observer模式是行为模式之一，它的作用是当一个对象的状态发生变化时，能够自动通知其他关联对象，自动刷新对象状态。

　　Observer模式提供给关联对象一种同步通信的手段，使某个对象与依赖它的其他对象之间保持状态同步。

### 观察者模式的结构

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180308153754305-1174557275.png)

 

### 观察者模式的角色和职责

　　**1、Subject（被观察者）**
  　　被观察的对象。当需要被观察的状态发生变化时，需要通知队列中所有观察者对象。Subject需要维持（添加，删除，通知）一个观察者对象的队列列表。

　　**2、ConcreteSubject**
  　　被观察者的具体实现。包含一些基本的属性状态及其他操作。

　　**3、Observer（观察者）**
  　　接口或抽象类。当Subject的状态发生变化时，Observer对象将通过一个callback函数得到通知。

　　**4、ConcreteObserver**
  　　观察者的具体实现。得到通知后将完成一些具体的业务逻辑处理。

 

　　而被观察者想要起作用，就必须继承java.util包下的Observable类，这是它的方法，后面会有介绍

| 构造方法摘要                                                 |
| ------------------------------------------------------------ |
| `**Observable**()`      构造一个带有零个观察者的 Observable。 |

 

| 方法摘要          |                                                              |
| ----------------- | ------------------------------------------------------------ |
| ` void`           | `**addObserver**(Observer o)`      如果观察者与集合中已有的观察者不同，则向对象的观察者集中添加此观察者。 |
| `protected  void` | `**clearChanged**()`      指示对象不再改变，或者它已对其所有的观察者通知了最近的改变，所以 `hasChanged` 方法将返回 `false`。 |
| ` int`            | `**countObservers**()`      返回 `Observable` 对象的观察者数目。 |
| ` void`           | `**deleteObserver**(Observer o)`      从对象的观察者集合中删除某个观察者。 |
| ` void`           | `**deleteObservers**()`      清除观察者列表，使此对象不再有任何观察者。 |
| ` boolean`        | `**hasChanged**()`      测试对象是否改变。                   |
| ` void`           | `**notifyObservers**()`      如果 `hasChanged` 方法指示对象已改变，则通知其所有观察者，并调用 `clearChanged` 方法来指示此对象不再改变。 |
| ` void`           | `**notifyObservers**(Object arg)`      如果 `hasChanged` 方法指示对象已改变，则通知其所有观察者，并调用 `clearChanged` 方法来指示此对象不再改变。 |
| `protected  void` | `**setChanged**()`      标记此 `Observable` 对象为已改变的对象；现在 `hasChanged` 方法将返回 `true`。 |

 

　　下面写一个例子：新建一个Person类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class Person {
 2     private String name;
 3     private String sex;
 4     private int age;
 5     
 6     public String getName() {
 7         return name;
 8     }
 9     public void setName(String name) {
10         this.name = name;
11     }
12     public String getSex() {
13         return sex;
14     }
15     public void setSex(String sex) {
16         this.sex = sex;
17     }
18     public int getAge() {
19         return age;
20     }
21     public void setAge(int age) {
22         this.age = age;
23     }
24 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

 　我们要做的就是监听成员变量name，sex，age的变化，在数值变化是，执行我们的操作，所以Person就是被观察者，所以server必须继承Observable，而Observable中有这三个方法：

　　1、`**notifyObservers**()` ： 如果 `hasChanged` 方法指示对象已改变，则通知其所有观察者，并调用 `clearChanged` 方法来指示此对象不再改变。

　　　　这个方法是通知观察者被观察者是否改变的，只要hasChanged()方法指示的对象改变，就会调用观察者中的方法。

　　2、`**hasChanged**()` ： 测试对象是否改变。

　　3、`**setChanged**()` ：标记此 `Observable` 对象为已改变的对象；现在 `hasChanged` 方法将返回 `true`。

　　所以，如果想观察成员变量是否改变，就要在set方法中，执行`**setChanged**()与**notifyObservers**()`

　　所以，被观察者应该改为：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 import java.util.Observable;
 2 
 3 public class Person extends Observable{
 4     private String name;
 5     private String sex;
 6     private int age;
 7     
 8     public String getName() {
 9         return name;
10     }
11     public void setName(String name) {
12         this.name = name;
13         this.setChanged();
14         this.notifyObservers();
15     }
16     public String getSex() {
17         return sex;
18     }
19     public void setSex(String sex) {
20         this.sex = sex;
21         this.setChanged();
22         this.notifyObservers();
23     }
24     public int getAge() {
25         return age;
26     }
27     public void setAge(int age) {
28         this.age = age;
29         this.setChanged();
30         this.notifyObservers();
31     }
32 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　有了被观察者，就要有观察者，观察者必须实现java.util包下的Observer接口，并重写update(Observable o, Object arg)方法，当被观察者改变时，就会执行update()方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 import java.util.Observable;
 2 import java.util.Observer;
 3 
 4 public class MyObserver implements Observer {
 5 
 6     @Override
 7     public void update(Observable o, Object arg) {
 8         System.out.println("对象已改变");
 9     }
10 
11 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　现在，就可以执行看一看了。不过在执行set()方法之前一定要使用`**addObserver**(Observer o)` 这个方法注册观察者，不然不会生效。

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         Person person = new Person();
 4         //注册观察者
 5         person.addObserver(new MyObserver());
 6         person.setName("小明");
 7         person.setSex("男");
 8         person.setAge(18);
 9     }
10 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

输出结果是这样的：

　　　　　　对象已改变
　　　　　　对象已改变
　　　　　　对象已改变

　　同时，notifyObservers()为什么是s结尾呢，因为我们可以同时注册多个观察者，这样写

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         Person person = new Person();
 4         //注册观察者
 5         person.addObserver(new MyObserver());
 6         person.addObserver(new MyObserver());
 7         
 8         person.setName("小明");
 9         person.setSex("男");
10         person.setAge(18);
11     }
12 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　我们注册两个观察者，两个都会生效，结果就变为了：　　　　

　　　　　　对象已改变
　　　　　对象已改变
　　　　　对象已改变
　　　　　对象已改变
　　　　　对象已改变
　　　　　对象已改变

　　还有三个方法`**deleteObserver**(Observer o)` ，`**deleteObservers**()` ，`**countObservers**()` 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         Person person = new Person();
 4         //注册观察者
 5         MyObserver myObserver = new MyObserver();
 6         person.addObserver(myObserver);
 7         person.addObserver(new MyObserver());
 8         //获得当前对象已注册的观察者数目
 9         person.countObservers();
10         //删除指定的一个观察者
11         person.deleteObserver(myObserver);
12         //删除该对象全部观察者
13         person.deleteObservers();
14         
15         person.setName("小明");
16         person.setSex("男");
17         person.setAge(18);
18     }
19 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

### **观察者模式的典型应用**

　  1、侦听事件驱动程序设计中的外部事件

　  2、侦听/监视某个对象的状态变化

　  3、发布者/订阅者(publisher/subscriber)模型中，当一个外部事件（新的产品，消息的出现等等）被触发时，通知邮件列表中的订阅者

# [10、享元模式](https://www.cnblogs.com/xiaobai1226/p/8532674.html)

　　Flyweight模式也叫享元模式，是构造型模式之一，它通过与其他类似对象共享数据来减小内存占用。它使用共享物件，用来尽可能减少内存使用量以及分享资讯给尽可能多的相似物件；它适合用于只是因重复而导致使用无法令人接受的大量内存的大量物件。通常物件中的部分状态是可以分享。常见做法是把它们放在外部数据结构，当需要使用时再将它们传递给享元。　

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180313155224589-2060882133.png)

 

 

### 享元模式的角色和职责

　　　　**1、抽象享元角色：**为具体享元角色规定了必须实现的方法，而外蕴状态就是以参数的形式通过此方法传入。在Java中可以由抽象类、接口来担当**。**。

　　　　**2、具体享元角色：**实现抽象角色规定的方法。如果存在内蕴状态，就负责为内蕴状态提供存储空间。

　　　　**3、享元工厂角色：**负责创建和管理享元角色。要想达到共享的目的，这个角色的实现是关键。

　　　　**4、客户端角色**  ：维护对所有享元对象的引用，而且还需要存储对应的外蕴状态。

### 两个状态

　　　　1、内蕴状态存储在享元内部，不会随环境的改变而有所不同，是可以共享的

　　　　2、外蕴状态是不可以共享的，它随环境的改变而改变的，因此外蕴状态是由客户端来保持（因为环境的变化是由客户端引起的）。

　　下面举个小例子

　　首先，创建一个抽象享元角色

```
1 public interface Flyweight {
2     public void display();
3 }
```

　　接着，创建具体享元角色

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MyFlyweight implements Flyweight{
 2     private String str;
 3     
 4     public MyFlyweight(String str){
 5         this.str = str;
 6     }
 7     
 8     public void display(){
 9         System.out.println(str);
10     }
11 }　　
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　如果，不使用享元模式的话，不创建享元工厂，直接，创建客户端，代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         Flyweight myFlyweight1 = new MyFlyweight("a");
 4         Flyweight myFlyweight2 = new MyFlyweight("b");
 5         Flyweight myFlyweight3 = new MyFlyweight("a");
 6         Flyweight myFlyweight4 = new MyFlyweight("d");
 7         
 8         myFlyweight1.display();
 9         myFlyweight2.display();
10         myFlyweight3.display();
11         myFlyweight4.display();
12         
13         System.out.println(myFlyweight1 == myFlyweight3);
14     }
15 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这样子，运行结果为：

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180313160937446-899677766.png)

　　这样子，可以看到，第一个与第三个明明都是a，但却不是同一个对象，说明虽然对象内部一模一样，但却创建了两个对象，这样就浪费了资源。

　　如果用到享元模式，继续创建享元工厂

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MyFlyweightFactory {
 2     private Map<String,MyFlyweight> pool;
 3     
 4     public MyFlyweightFactory(){
 5         pool = new HashMap<String,MyFlyweight>();
 6     }
 7     
 8     public Flyweight getMyFlyweight(String str){
 9         MyFlyweight myFlyweight = pool.get(str);
10         
11         //若池中没有则创建一个新的并放入池中，若池中已存在，则返回池中的
12         if(myFlyweight == null){
13             myFlyweight = new MyFlyweight(str);
14             pool.put(str, myFlyweight);
15         }
16         
17         return myFlyweight;
18     }
19 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这样，修改客户端

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         MyFlyweightFactory myFlyweightFactory = new MyFlyweightFactory();
 4         Flyweight myFlyweight1 = myFlyweightFactory.getMyFlyweight("a");
 5         Flyweight myFlyweight2 = myFlyweightFactory.getMyFlyweight("b");
 6         Flyweight myFlyweight3 = myFlyweightFactory.getMyFlyweight("a");
 7         Flyweight myFlyweight4 = myFlyweightFactory.getMyFlyweight("d");
 8         
 9         myFlyweight1.display();
10         myFlyweight2.display();
11         myFlyweight3.display();
12         myFlyweight4.display();
13         
14         System.out.println(myFlyweight1 == myFlyweight3);
15     }
16 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　此时，结果就变为了

 

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180313162421648-1551411963.png)

　　可以看到，第一个与第三个变为了同一个对象，一模一样的对象只创建一次，节约了资源，这样，享元模式的作用就达到了。

### 使用场景

　　如果一个应用程序使用了大量的对象，而这些对象造成了很大的存储开销的时候就可以考虑是否可以使用享元模式。

例如,如果发现某个对象的生成了大量细粒度的实例，并且这些实例除了几个参数外基本是相同的，如果把那些共享参数移到类外面，在方法调用时将他们传递进来，就可以通过共享大幅度单个实例的数目。

# [11、代理模式](https://www.cnblogs.com/xiaobai1226/p/8558614.html)

　　Proxy模式又叫做代理模式，是构造型的设计模式之一，它可以为其他对象提供一种代理（Proxy）以控制对这个对象的访问。

　　所谓代理，是指具有与代理元（被代理的对象）具有相同的接口的类，客户端必须通过代理与被代理的目标类交互，而代理一般在交互的过程中（交互前后），进行某些特别的处理。

### 代理模式的结构

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180314082334136-1522527791.png)

### 代理模式的角色与职责

　　　**1、subject（抽象主题角色）**：真实主题与代理主题的共同接口或抽象类。

　　　**2、RealSubject（真实主题角色）：**定义了代理角色所代表的真实对象。

　　　**3、Proxy（代理主题角色）：**含有对真实主题角色的引用，代理角色通常在将客户端调用传递给真是主题对象之前或者之后执行某些操作，而不是单纯返回真实的对象。

#### 什么是代理模式

　　比如说买书，网上有很多专门卖书的网站，我们从这些商城买书，但是书不是这些商城印的，他们只负责卖，书是出版社印的，所以说到底，我们其实还是从出版社买书，网上书城只是出版社的代理，所以出版社是被代理对象，书城是代理对象。

　　所以，根据角色与职责划分，**subject（抽象主题角色）**就是卖书，卖书是书城与出版社的共同功能，**RealSubject（真实主题角色）**就是出版社，它的功能就是卖书，但不直接卖给用户，而是被书城代理，通过代理来卖，**Proxy（代理主题角色）**就是书城，根据概念发现，代理对象在代理的过程中不仅仅只有被代理对象的功能，他还会执行某些他自己的操作，这个意思是，比如，书城会推出优惠券与打折活动。在卖书的基础上，增加许多功能来吸引消费者。

　　接下来，我们用代码实现刚才的例子：

　　**代理模式分为两种类型：（1）静态代理（2）动态代理**

### （1）静态代理

　　**（静态代理在使用时,需要定义接口或者父类,被代理对象与代理对象一起实现相同的接口或者是继承相同父类）**

　　首先，创建subject，一个接口，是代理对象与被代理对象的共同接口

　　书城与出版社都有卖书的功能

```
1 public interface Subject {
2     public void sailBook();
3 }
```

 　然后，创建RealSubject，被代理对象，也就是出版社

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class RealSubject implements Subject {
2 
3     @Override
4     public void sailBook() {
5         System.out.println("卖书");
6     }
7 
8 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　然后，创建代理对象，也就是书城

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class ProxySubject implements Subject{
 2     //代理对象含有对真实主题角色的引用
 3     private Subject subject;
 4     
 5     public ProxySubject(Subject subject){
 6         this.subject = subject;
 7     }
 8 
 9     @Override
10     public void sailBook() {
11         dazhe();
12         this.subject.sailBook();
13         give();
14     }
15     
16     //代理角色通常在将客户端调用传递给真是主题对象之前或者之后执行某些操作，而不是单纯返回真实的对象。
17     public void dazhe(){
18         System.out.println("打折");
19     }
20     
21     public void give(){
22         System.out.println("赠送代金券");
23     }
24 
25 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　最后，创建客户端，也就是用户

　　首先如果，不使用代理直接从出版社买

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class MainClass {
2     public static void main(String[] args) {
3         Subject realSubject = new RealSubject();
4         realSubject.sailBook();
5     }
6 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　结果如下：仅仅是卖书

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180314091707897-1665400730.png)

　　而如果我们通过代理也就是书城来买

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class MainClass {
2     public static void main(String[] args) {
3         Subject realSubject = new RealSubject();
4         Subject proxySubject = new ProxySubject(realSubject);
5         proxySubject.sailBook();
6     }
7 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　结果变成了这样，作为用户，我们享受了更多的优惠，所以我们当然更愿意通过代理来买

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180314091833280-399853363.png)

　　

　　**静态代理总结:**
　　1、可以做到在不修改目标对象的功能前提下,对目标功能扩展.
　　2、**缺点:**因为代理对象需要与目标对象实现一样的接口,所以会有很多代理类,类太多.同时,一旦接口增加方法,目标对象与代理对象都要维护.

　　**如何解决静态代理中的缺点呢?答案是可以使用动态代理方式**

### （2）动态代理

　　**动态代理有以下特点:**

　　1.代理对象,不需要实现接口

　　2.代理对象的生成,是利用JDK的API,动态的在内存中构建代理对象(需要我们指定创建代理对象/目标对象实现的接口的类型)

　　3.动态代理也叫做:JDK代理,接口代理

　　代理代理，Subject与RealSubject不变

   而使用动态代理，首先要创建代理实例的调用处理程序，通过这个程序来执行代理对象特有的方法

　　创建MyHandler实现**InvocationHandler（是代理实例的\*调用处理程序\* 实现的接口）**这个接口，并覆盖**invoke(Object proxy, Method method, Object[] args)**这个方法。

| ` Object` | `**invoke**(Object proxy, Method method, Object[] args)`      在代理实例上处理方法调用并返回结果。 |
| --------- | ------------------------------------------------------------ |
|           |                                                              |

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 import java.lang.reflect.InvocationHandler;
 2 import java.lang.reflect.Method;
 3 
 4 public class MyHandler implements InvocationHandler{
 5     //这里也需要传入被代理对象
 6     private Subject subject;
 7     
 8     public MyHandler(Subject subject){
 9         this.subject = subject;
10     }
11     
12     //这个方法中就是代理对象要执行的方法
13     @Override
14     public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
15         dazhe();
16         //执行被代理对象中的方法
17         Object result = method.invoke(subject, args);
18         give();
19         return result;
20     }
21     
22     public void dazhe() {
23         System.out.println("打折");
24     }
25     
26     public void give() {
27         System.out.println("赠送代金券");
28     }
29 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

 　最后，写客户端，客户端中需要动态创建代理对象，要使用Proxy类中的`**newProxyInstance**(ClassLoader loader, Class[] interfaces, InvocationHandler h)方法`

| `static Object` | `**newProxyInstance**(ClassLoader loader, Class[] interfaces, InvocationHandler h)`      返回一个指定接口的代理类实例，该接口可以将方法调用指派到指定的调用处理程序。 |
| --------------- | ------------------------------------------------------------ |
|                 |                                                              |

　　**参数：**

- 　　**返回：**

    　　一个带有代理类的指定调用处理程序的代理实例，它由指定的类加载器定义，并实现指定的接口

- 　　**抛出：**

  `　　IllegalArgumentException` - 如果违反传递到 `getProxyClass` 的参数上的任何限制

  `　　NullPointerException` - 如果 `interfaces` 数组参数或其任何元素为 `null`，或如果调用处理程序 `h` 为 `null`

 　所以三个参数，第一个是RealSubject的类加载器，第二个是被代理类要实现的接口列表，第三个就是处理程序也就是MyHandler

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class MainClass {
2     public static void main(String[] args) {
3         Subject realSubject = new RealSubject();
4         MyHandler myHandler = new MyHandler(realSubject);
5         //创建代理对象实例
6         Subject proxySubject = (Subject) Proxy.newProxyInstance(Subject.class.getClassLoader(), realSubject.getClass().getInterfaces(), myHandler);
7         proxySubject.sailBook();
8     }
9 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　结果是相同的

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180314103155748-1601116339.png)

　　 **总结:代理对象不需要实现接口,但是目标对象一定要实现接口,否则不能用动态代理**

# [12、外观模式](https://www.cnblogs.com/xiaobai1226/p/8566231.html)

　　Facade模式也叫外观模式，是由GoF提出的23种设计模式中的一种。Facade模式为一组具有类似功能的类群，比如类库，子系统等等，提供一个一致的简单的界面。这个一致的简单的界面被称作facade。

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180314112325866-702970469.png)

### 外观模式的角色和职责　　

　　**1、Facade：**为调用方定义简单的调用接口。　　

　　**2、Clients：**调用者。通过Facade接口调用提供某功能的内部类群。　　

　　**3、Packages：**功能提供者。指提供功能的类群（模块或子系统）。

### 使用场景

　　在以下情况下可以考虑使用外观模式：

　　(1)设计初期阶段，应该有意识的将不同层分离，层与层之间建立外观模式。

　　(2) 开发阶段，子系统越来越复杂，增加外观模式提供一个简单的调用接口。

　　(3) 维护一个大型遗留系统的时候，可能这个系统已经非常难以维护和扩展，但又包含非常重要的功能，为其开发一个外观类，以便新系统与其交互。

 

　　简单来说，使用了外观模式，用户就不必直接面对众多功能模块，降低了使用难度，简单举个例子

　　电脑开机进入系统，我们把他分为4步，首先打开电源，bois自检，系统引导，进入系统，4个功能是四个功能模块

　　**打开电源**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class StartPower {
2     /*
3      * 打开电源
4      */
5     public void startPower(){
6         System.out.println("电脑通电");
7     }
8 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　**bois自检**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class BoisSelfTest {
2     /*
3      * bios自检
4      */
5     public void boisSelfTest(){
6         System.out.println("bios自检");
7     }
8 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　**系统引导**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class SystemGuide {
2     /*
3      * 系统引导
4      */
5     public void systemGuide(){
6         System.out.println("系统引导");
7     } 
8 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　**进入系统**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class EnterSystem {
2     /*
3      * 进入系统
4      */
5     public void enterSystem(){
6         System.out.println("进入系统");
7     }
8 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　如果，我们没有使用外观模式，就需要用户自己挨个使用这些功能

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         StartPower startPower = new StartPower(); 
 4         startPower.startPower();
 5         
 6         BoisSelfTest boisSelfTest = new BoisSelfTest();
 7         boisSelfTest.boisSelfTest();
 8         
 9         SystemGuide systemGuide = new SystemGuide();
10         systemGuide.systemGuide();
11         
12         EnterSystem enterSystem = new EnterSystem();
13         enterSystem.enterSystem();
14     }
15 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　运行结果为：

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180314135203544-1600315309.png)

　　这样，电脑顺利启动，可是同样可以看出来，用户使用非常繁琐，不仅需要用户自己挨个使用所有用到的功能，同时还需要用户知道电脑启动的顺序，按顺序使用功能，不然就会导致问题，这样显然是不可取的

　　所以，用到外观模式，创建一个Facade，专门用于使用功能模块

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class Facade {
 2     private StartPower startPower = null; 
 3     private BoisSelfTest boisSelfTest = null;
 4     private SystemGuide systemGuide = null;
 5     private EnterSystem enterSystem = null;
 6     
 7     public void startComputer(){
 8         startPower = new StartPower(); 
 9         boisSelfTest = new BoisSelfTest();
10         systemGuide = new SystemGuide();
11         enterSystem = new EnterSystem();
12         
13         startPower.startPower();
14         boisSelfTest.boisSelfTest();
15         systemGuide.systemGuide();
16         enterSystem.enterSystem();
17     }
18 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这时客户端就是这样

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class MainClass {
2     public static void main(String[] args) {
3         Facade computer = new Facade();
4         computer.startComputer();
5     }
6 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　可以看到，用户这时只需要调用Facade中的方法即可，无需知道电脑有什么具体的功能模块，无需知道功能模块执行的顺序是什么，只是调用一下就好了。

 　简化了使用，同时也增加了代码的复用与可维护性。

### 优点

　　**1、****松散耦合：**使得客户端和子系统之间解耦，让子系统内部的模块功能更容易扩展和维护；

　　**2、****简单易用：**客户端根本不需要知道子系统内部的实现，或者根本不需要知道子系统内部的构成，它只需要跟Facade类交互即可。

　　**3、****更好的划分访问层次：**有些方法是对系统外的，有些方法是系统内部相互交互的使用的。子系统把那些暴露给外部的功能集中到门面中，这样就可以实现客户端的使用，很好的隐藏了子系统内部的细节。

# [13、组合模式](https://www.cnblogs.com/xiaobai1226/p/8567280.html)

　　Composite模式也叫组合模式，是构造型的设计模式之一。通过递归手段来构造树形的对象结构，并可以通过一个对象来访问整个对象树。　

　　组合(Composite)模式的其它翻译名称也很多，比如合成模式、树模式等等。在《设计模式》一书中给出的定义是：将对象以树形结构组织起来，以达成“部分－整体”的层次结构，使得客户端对单个对象和组合对象的使用具有一致性。

 

　　从定义中可以得到使用组合模式的环境为：在设计中想表示对象的“部分－整体”层次结构；希望用户忽略组合对象与单个对象的不同，统一地使用组合结构中的所有对象。

 

　　![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180314143554503-722119602.png)

 

### 组合模式的角色和职责

　　**1、Component** **（树形结构的节点抽象）**

　　　　1.1、为所有的对象定义统一的接口（公共属性，行为等的定义）

　　　　1.2、提供管理子节点对象的接口方法

　　　　1.3、[可选]提供管理父节点对象的接口方法

　　**2、Leaf** **（树形结构的叶节点）**

　　　　Component的实现子类

　　**3、Composite****（树形结构的枝节点）**

　　　　Component的实现子类

### 安全性与透明性

　　组合模式中必须提供对子对象的管理方法，不然无法完成对子对象的添加删除等等操作，也就失去了灵活性和扩展性。但是管理方法是在Component中就声明还是在Composite中声明呢？

　　一种方式是在Component里面声明所有的用来管理子类对象的方法，以达到Component接口的最大化（如下图所示）。目的就是为了使客户看来在接口层次上树叶和分支没有区别——**透明性**。但树叶是不存在子类的，因此Component声明的一些方法对于树叶来说是不适用的。这样也就带来了一些安全性问题。

 ![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180315082650051-357062003.png)

 

 

　　另一种方式就是只在Composite里面声明所有的用来管理子类对象的方法（如下图所示）。这样就避免了上一种方式的**安全性**问题，但是由于叶子和分支有不同的接口，所以又失去了透明性。

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180315082740848-1010330558.png)

　　《设计模式》一书认为：在这一模式中，相对于安全性，我们比较强调透明性。对于第一种方式中叶子节点内不需要的方法可以使用空处理或者异常报告的方式来解决。

 　我们举个例子，比如说文件夹与文件，层次结构就符合树形结构

　　首先我们新建一个Component接口**（树形结构的节点抽象）**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 文件节点抽象(是文件和目录的父类)
 3  */
 4 public interface IFile {
 5     
 6     //显示文件或者文件夹的名称
 7     public void display();
 8     
 9     //添加
10     public boolean add(IFile file);
11     
12     //移除
13     public boolean remove(IFile file);
14     
15     //获得子节点
16     public List<IFile> getChild();
17 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　该例子符合透明性，如果是安全性，将IFile中的方法去除，不在IFile声明，直接在子类中声明即可。

　　接下来，创建一个**Leaf**（叶子结点），因为文件是不可再分的，所以File是叶子结点

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 文件（leaf 叶子结点）
 3  */
 4 public class File implements IFile {
 5     private String name;
 6     
 7     public File(String name) {
 8         this.name = name;
 9     }
10     
11 
12     public void display() {
13         System.out.println(name);
14     }
15 
16     public List<IFile> getChild() {
17         return null;
18     }
19 
20 
21     public boolean add(IFile file) {
22         return false;
23     }
24 
25     public boolean remove(IFile file) {
26         return false;
27     }
28 
29 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　然后继续创建**Composite**(文件夹)，因为文件夹下面还可能有文件与文件夹，所以是composite

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class Folder implements IFile{
 2     private String name;
 3     private List<IFile> children;
 4     
 5     public Folder(String name) {
 6         this.name = name;
 7         children = new ArrayList<IFile>();
 8     }
 9     
10     public void display() {
11         System.out.println(name);
12     }
13 
14     public List<IFile> getChild() {
15         return children;
16     }
17 
18     public boolean add(IFile file) {
19         return children.add(file);
20     }
21 
22     public boolean remove(IFile file) {
23         return children.remove(file);
24     }
25 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　然后是客户端，用递归的形式把这个具有树形结构的对象遍历出来

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         //C盘
 4         Folder rootFolder = new Folder("C:");
 5         //C盘下的目录一
 6         Folder folder1 = new Folder("目录一");
 7         //C盘下的文件一
 8         File file1 = new File("文件一.txt");
 9         
10         rootFolder.add(folder1);
11         rootFolder.add(file1);
12         
13         //目录一下的目录二
14         Folder folder2 = new Folder("目录二");
15         //目录一下的文件二
16         File file2 = new File("文件二.txt");
17         folder1.add(folder2);
18         folder1.add(file2);
19         
20         //目录二下的目录三
21         Folder folder3 = new Folder("目录三");
22         //目录二下的文件三
23         File file3 = new File("文件三.txt");
24         folder2.add(folder3);
25         folder2.add(file3);
26         
27         displayTree(rootFolder,0);
28         
29     }
30     
31     public static void displayTree(IFile rootFolder, int deep) {
32         for(int i = 0; i < deep; i++) {
33             System.out.print("--");
34         }
35         //显示自身的名称
36         rootFolder.display();
37         //获得子树
38         List<IFile> children = rootFolder.getChild();
39         //遍历子树
40         for(IFile file : children) {
41             if(file instanceof File) {
42                 for(int i = 0; i <= deep; i++) {
43                     System.out.print("--");
44                 }
45                 file.display();
46             }else {
47                 displayTree(file,deep + 1);
48             }
49         }
50     }
51 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这样子，就把，这棵树遍历了出来。

### 优缺点

**优点：**

　　　　1) 使客户端调用简单，客户端可以一致的使用组合结构或其中单个对象，用户就不必关心自己处理的是单个对象还是整个组合结构，这就简化了客户端代码。

　　　　2)更容易在组合体内加入对象部件. 客户端不必因为加入了新的对象部件而更改代码。这一点符合开闭原则的要求，对系统的二次开发和功能扩展很有利！

**缺点：**

　　　　组合模式不容易限制组合中的构件。

### 总结

　　组合模式是一个应用非常广泛的设计模式，它本身比较简单但是很有内涵，掌握了它对你的开发设计有很大的帮助。

# [14、桥接模式](https://www.cnblogs.com/xiaobai1226/p/8577907.html)

　　Bridge 模式又叫做桥接模式，是构造型的设计模式之一。Bridge模式基于类的最小设计原则，通过使用封装，聚合以及继承等行为来让不同的类承担不同的责任。它的主要特点是把抽象（abstraction）与行为实现（implementation）分离开来，从而可以保持各部分的独立性以及应对它们的功能扩展。

 

　　举个例子，我们都知道，汽车有不同的发动机，有油的，有电的，还有用天然气的

　　我们先举两个不采用桥接模式的例子，通过对比，来看出桥接模式的优势

　　**第一个：**

　　先新建一个接口Car，所有车都实现这个接口，这个接口中有一个公共的方法installEngine()安装发动机

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 /*
2  * 汽车
3  */
4 public interface Car {
5     //安装引擎
6     public void installEngine();
7 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　然后，不同的车，比如公交，suv，轿车，都有不同的发动机，所以先新建不同的车，实现Car接口，因为不同种类的车下面还有不同的发动机，所以将具体的车建为抽象类，将其中实现Car的方法也抽象化。

　　公交

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 /*
2  * 公交车
3  */
4 public abstract class Bus implements Car{
5 
6     @Override
7     public abstract void installEngine();
8 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　SUV

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 /*
2  * SUV
3  */
4 public abstract class Suv implements Car{
5 
6     @Override
7     public abstract void installEngine();
8 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　不同的车有不同的发动机，每种车的不同发动机都是一个子类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class OilBus extends Bus{
2     @Override
3     public void installEngine() {
4         System.out.println("给公交安装燃油发动机");
5     }
6 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class ElectricityBus extends Bus{
2     @Override
3     public void installEngine() {
4         System.out.println("给公交安装电动发动机");
5     }
6 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class OilSuv extends Suv{
2     @Override
3     public void installEngine() {
4         System.out.println("给SUV安装燃油发动机");
5     }
6 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class ElectricitySuv extends Suv{
2     @Override
3     public void installEngine() {
4         System.out.println("给SUV安装电动发动机");
5     }
6 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　运行客户端

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class MainClass {
2     public static void main(String[] args) {
3         Car car = new OilBus();
4         car.installEngine();
5     }
6 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　可以看到这样的结果

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180316085824381-1143907530.png)

　　但是这样有个很大的弊端

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180316092409159-364251352.png)

　　可以看到，一种车每增加一种发动机就要增加一个子类，到了后期，子类会越来越多，越来越庞大。这种方法不推荐使用。

　　**第二个例子：**

　　把所有的发动机都定义到Car接口中，每种车实现一次就好了

　　Car

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 /*
2  * 汽车
3  */
4 public interface Car {
5     //安装燃油引擎
6     public void installOilEngine();
7     //安装电动引擎
8     public void installElectricityEngine();
9 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　Bus

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 公交车
 3  */
 4 public class Bus implements Car{
 5     @Override
 6     public void installOilEngine() {
 7         System.out.println("给公交安装燃油发动机");
 8     }
 9 
10     @Override
11     public void installElectricityEngine() {
12         System.out.println("给公交安装电动发动机");
13     }
14 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　Suv

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * SUV
 3  */
 4 public class Suv implements Car{
 5     @Override
 6     public void installOilEngine() {
 7         System.out.println("给SUV安装燃油发动机");
 8     }
 9 
10     @Override
11     public void installElectricityEngine() {
12         System.out.println("给SUV安装电动发动机");
13     }
14 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　客户端

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class MainClass {
    public static void main(String[] args) {
        Car bus = new Bus();
        Car suv = new Suv();
        
        bus.installOilEngine();
        bus.installElectricityEngine();
        
        suv.installOilEngine();
        suv.installElectricityEngine();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　运行结果

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180316093000650-1989257258.png)

　　这种方式就没有那么多子类了，每种车只有一个实现类，但是这种方法不符合我们的开放封闭原则，每增加一种发动机，就要修改Car接口，并修改所有实现类。这种方法肯定也是不推荐的。

　　接下来，就要说到桥接模式了。

 　桥接模式的概念：**Bridge模式基于类的最小设计原则，通过使用封装，聚合以及继承等行为来让不同的类承担不同的责任。它的主要特点是把抽象（abstraction）与行为实现（implementation）分离开来，从而可以保持各部分的独立性以及应对它们的功能扩展。**

 　把抽象与行为实现分开，保证各部分的独立性，所以我们可以把车和发动机分离开来，保证他们各自的独立性。

 　![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180316094346489-192847381.png)

　　增加一种车，对发动机没有影响，增加发动机吧对车也没有影响，Car接口中有对Engine的引用，也就是Car体系与Engine体系之间的桥梁。

　　![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180316081636940-1247165404.png)

 

　　

### 桥接模式的角色和职责

　　**1、Client：**Bridge模式的使用者

　　**2、Abstraction（Car）**

  　　（1）抽象类接口（接口或抽象类）

  　　（2）维护对行为实现（Implementor）的引用

　　**3、Refined Abstraction（Bus、Suv）：**Abstraction子类

　　**4、Implementor（Engine）：** 行为实现类接口 (Abstraction接口定义了基于Implementor接口的更高层次的操作)

　　**5、ConcreteImplementor（OilEngine、ElectricityEngine）：**Implementor子类

 　接下来，用代码实现：

　　首先，新建发动机体系

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 /*
2  * 发动机
3  */
4 public interface Engine {
5     public void installEngine();
6 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 /*
2  * 燃油发动机
3  */
4 public class OilEngine implements Engine{
5     @Override
6     public void installEngine() {
7         System.out.println("安装燃油发动机");
8     }
9 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 /*
2  * 电动发动机
3  */
4 public class ElectricityEngine implements Engine{
5     @Override
6     public void installEngine() {
7         System.out.println("安装电动发动机");
8     }
9 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

 　新建Car体系

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 汽车
 3  */
 4 public abstract class Car {
 5     private Engine engine;
 6     
 7     public Car(Engine engine){
 8         this.engine = engine;
 9     }
10     
11     //安装发动机
12     public abstract void installEngine();
13 
14     public Engine getEngine() {
15         return engine;
16     }
17 
18     public void setEngine(Engine engine) {
19         this.engine = engine;
20     }
21 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 公交车
 3  */
 4 public class Bus extends Car{
 5 
 6     public Bus(Engine engine) {
 7         super(engine);
 8     }
 9 
10     @Override
11     public void installEngine() {
12         System.out.print("公交：");
13         this.getEngine().installEngine();
14     }
15 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * SUV
 3  */
 4 public class Suv extends Car{
 5     public Suv(Engine engine) {
 6         super(engine);
 7     }
 8 
 9     @Override
10     public void installEngine() {
11         System.out.print("SUV：");
12         this.getEngine().installEngine();
13     }
14 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

 　最后，新建客户端调用

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         Engine oilEngine = new OilEngine();
 4         Engine electricityEngine = new ElectricityEngine();
 5         
 6         Car oilBus = new Bus(oilEngine);
 7         Car electricityBus = new Bus(electricityEngine);
 8         
 9         Car oilSuv = new Suv(oilEngine);
10         Car electricitySuv = new Suv(electricityEngine);
11         
12         oilBus.installEngine();
13         electricityBus.installEngine();
14         
15         oilSuv.installEngine();
16         electricitySuv.installEngine();
17     }
18 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　结果如下

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180316101155727-1028450790.png)

　　可以看到，这样子的话，新建这的种类，丝毫不影响发动机，新建一种发动机，也不影响车，这就是一个简单的桥接模式的例子

### **总结：**

　　**1.桥接模式的优点**

　　（1）实现了抽象和实现部分的分离

　　桥接模式分离了抽象部分和实现部分，从而极大的提供了系统的灵活性，让抽象部分和实现部分独立开来，分别定义接口，这有助于系统进行分层设计，从而产生更好的结构化系统。对于系统的高层部分，只需要知道抽象部分和实现部分的接口就可以了。

　　（2）更好的可扩展性

　　由于桥接模式把抽象部分和实现部分分离了，从而分别定义接口，这就使得抽象部分和实现部分可以分别独立扩展，而不会相互影响，大大的提供了系统的可扩展性。

　　（3）可动态的切换实现

　　由于桥接模式实现了抽象和实现的分离，所以在实现桥接模式时，就可以实现动态的选择和使用具体的实现。

　　（4）实现细节对客户端透明，可以对用户隐藏实现细节。

　　**2.桥接模式的缺点**

　　（1）桥接模式的引入增加了系统的理解和设计难度，由于聚合关联关系建立在抽象层，要求开发者针对抽象进行设计和编程。

　　（2）桥接模式要求正确识别出系统中两个独立变化的维度，因此其使用范围有一定的局限性。

　　**3.桥接模式的使用场景**

　　（1）如果一个系统需要在构件的抽象化角色和具体化角色之间增加更多的灵活性，避免在两个层次之间建立静态的继承联系，通过桥接模式可以使它们在抽象层建立一个关联关系。

　　（2）抽象化角色和实现化角色可以以继承的方式独立扩展而互不影响，在程序运行时可以动态将一个抽象化子类的对象和一个实现化子类的对象进行组合，即系统需要对抽象化角色和实现化角色进行动态耦合。

　　（3）一个类存在两个独立变化的维度，且这两个维度都需要进行扩展。

　　（4）虽然在系统中使用继承是没有问题的，但是由于抽象化角色和具体化角色需要独立变化，设计要求需要独立管理这两者。

　　（5）对于那些不希望使用继承或因为多层次继承导致系统类的个数急剧增加的系统，桥接模式尤为适用

# [15、适配器模式](https://www.cnblogs.com/xiaobai1226/p/8598890.html)

### 概念：

　　Adapter模式也叫适配器模式，是构造型模式之一，通过Adapter模式可以改变已有类（或外部类）的接口形式。

　　举个例子：我们使用电脑，家里的电源是220V的，而我们的电脑是18V的，这时如果我们直接把电源连接电脑，一定会导致电脑被烧坏，因为电源电压太高了，这时我们就需要一个电源适配器，连接在电源与电脑之间，通过适配器进行一个降压，来保证电脑的正常工作。

　　![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180319084329545-2121481002.png)

　　增加适配器

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180319084506741-2142229869.png)

 　用代码实现：

　　首先如果不使用适配器的话

　　新建一个220V电源

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 //220V电源
2 public class PowerSupply {
3     public void powerSupply220V(){
4         System.out.println("使用220V电源");    
5     }
6 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　新建一个笔记本电脑，使用电源

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 //笔记本电脑使用220V电源
2 public class Computer {
3     public static void main(String[] args) {
4         PowerSupply powerSupply = new PowerSupply();
5         powerSupply.powerSupply220V();
6     }
7 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　结果如下：

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180319103015768-322288133.png)

　　这样笔记本电脑就直接使用了220V电压，但是这样的话，笔记本电脑会直接烧毁，无法使用，因为电压太高了，所以我们需要在中间接一个适配器，以达到降压的目的

　　适配器继承220V电压

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 //适配器
2 public class Adapter extends PowerSupply{
3     public void powerSupply18V(){
4         System.out.println("使用适配器");
5         this.powerSupply220V();
6         System.out.println("电压降为18V");
7     }
8 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　笔记本电脑通过适配器调用电源

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
//笔记本电脑通过适配器使用220V电源
public class Computer {
    public static void main(String[] args) {
        Adapter adapter = new Adapter();
        adapter.powerSupply18V();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　结果如下：

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180319103340710-1095207269.png)

　　可以看到，通过这种形式，我们使用的还是之前的那个电源，但是通过适配器，电压就降到了18V，电脑就可以正常使用了。

　　但是这只是适配器模式的其中一种形式。

　　下面更详细的说明一下适配器模式

### 适配器模式使用场景

　　在大规模的系统开发过程中，我们常常碰到诸如以下这些情况：

　　我们需要实现某些功能，这些功能已有还不太成熟的一个或多个外部组件，如果我们自己重新开发这些功能会花费大量时间；所以很多情况下会选择先暂时使用外部组件，以后再考虑随时替换。但这样一来，会带来一个问题，随着对外部组件库的替换，可能需要对引用该外部组件的源代码进行大面积

的修改，因此也极可能引入新的问题等等。如何最大限度的降低修改面呢？

　　Adapter模式就是针对这种类似需求而提出来的。Adapter模式通过定义一个新的接口（对要实现的功能加以抽象），和一个实现该接口的Adapter（适配器）类来透明地调用外部组件。这样替换外部组件时，最多只要修改几个Adapter类就可以了，其他源代码都不会受到影响。

　　简单来说：

　　1、 系统需要使用现有的类，而这些类的接口不符合系统的需要。 

　　2、 想要建立一个可以重复使用的类，用于与一些彼此之间没有太大关联的一些类，包括一些可能在将来引进的类一起工作。 

　　3、 需要一个统一的输出接口，而输入端的类型不可预知。

　　下面说一下适配器模式的结构　

### 1.通过继承实现Adapter

 ![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180319110517623-2145875060.png)

　　这种形式，就是我们刚才举的例子

 　**Client：**就是笔记本电脑**（Computer）**

　　**Target：**就是笔记本电脑中调用的方法**（adapter.powerSupply18V()）**

 　 **Adaptee：**就是220V电压**（PowerSupply）**

　　**Adapter：**就是适配器**（Adapter）**

### 2.通过委让实现Adapter

 ![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180319110956502-371912298.png)

　　

 

　　第二种模式，只需修改适配器与电脑即可

　　适配器不再继承电源，而是当成一个成员变量

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 //适配器
 2 public class Adapter{
 3     private PowerSupply powerSupply;
 4     
 5     public Adapter(PowerSupply powerSupply){
 6         this.powerSupply = powerSupply;
 7     }
 8     
 9     public void powerSupply18V(){
10         System.out.println("使用适配器");
11         this.powerSupply.powerSupply220V();
12         System.out.println("电压降为18V");
13     }
14 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　电脑

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 //笔记本电脑通过适配器使用220V电源
2 public class Computer {
3     public static void main(String[] args) {
4         Adapter2 adapter = new Adapter2(new PowerSupply());
5         adapter.powerSupply18V();
6     }
7 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　一般，使用第二种委让形式更多一些，因为这种方式不必继承，使用成员变量更加的灵活



# [16、解释器模式](https://www.cnblogs.com/xiaobai1226/p/8601760.html)

### 概念：

　　Interpreter模式也叫解释器模式，是行为模式之一，它是一种特殊的设计模式，它建立一个解释器，对于特定的计算机程序设计语言，用来解释预先定义的文法。简单地说，Interpreter模式是一种简单的语法解释器构架。　

### 解释器模式应用场景

　　当有一个语言需要解释执行, 并且你可将该语言中的句子表示为一个抽象语法树时，可使用解释器模式。而当存在以下情况时该模式效果最好： 

　　　　1、该文法简单，对于复杂的文法, 文法的类层次变得庞大而无法管理。此时语法分析程序生成器这样的工具是更好的选择。它们无需构建抽象语法树即可解释表达式, 这样可以节省空间而且还可能节省时间。 

　　　　2、效率不是一个关键问题，最高效的解释器通常不是通过直接解释语法分析树实现的, 而是首先将它们转换成另一种形式。例如，正则表达式通常被转换成状态机。但即使在这种情况下, 转换器仍可用解释器模式实现, 该模式仍是有用的。

### 结构图

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180320132742078-1514737858.png)

### 解释器模式的角色和职责

　　**1、Context：**解释器上下文环境类。用来存储解释器的上下文环境，比如需要解释的文法等，简单来说，它的任务一般是用来存放文法中各个终结符所对应的具体值，比如R=R1+R2，给R1赋值100，给R2赋值200，这些信息需要存放到环境中。

　　**2、AbstractExpression：**解释器抽象类。抽象表达式，声明一个所有的具体表达式都需要实现的抽象接口；这个接口主要是一个interpret()方法，称做解释操作。

　　**3、Terminal Expression：**终结符表达式，解释器具体实现类，实现了抽象表达式所要求的接口；文法中的每一个终结符都有一个具体终结表达式与之相对应。比如公式R=R1+R2，R1和R2就是终结符，对应的解析R1和R2的解释器就是终结符表达式。

　　**4、Nonterminal Expression：**非终结符表达式，解释器具体实现类，文法中的每一条规则都需要一个具体的非终结符表达式，非终结符表达式一般是文法中的运算符或者其他关键字，比如公式R=R1+R2中，“+"就是非终结符，解析“+”的解释器就是一个非终结符表达式。

　　**5、Client：**客户端，指的是使用解释器的客户端，通常在这里将按照语言的语法做的表达式转换成使用解释器对象描述的抽象语法树，然后调用解释操作。

　　举一个计算器的例子

　　首先，新建一个**Context**，通过一个map来存储上下文信息

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 解释器上下文环境类
 3  */
 4 public class Context {
 5     private Map<Character, Double> variable;
 6 
 7     public Map<Character, Double> getVariable() {
 8         if (this.variable == null)
 9         {
10             this.variable = new HashMap<Character, Double>();
11         }
12         return this.variable;
13     }
14     
15     //返回结果
16     public void setVariable(Map<Character, Double> variable) {
17         this.variable = variable;
18     }
19 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　再新建**AbstractExpression**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 /*
2  * 抽象表达式
3  */
4 public abstract class Expression {
5     public abstract double Interpret(Context context);
6 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　新建**Terminal Expression**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 变量，终结符表达式
 3  */
 4 public class VariableExpression extends Expression {
 5     private char key;
 6     
 7     public VariableExpression(char key)
 8     {
 9         this.key = key;
10     }
11     
12     @Override
13     public double Interpret(Context context) {
14         return context.getVariable().get(key);
15     }
16 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　再新建**Nonterminal Expression，**因为这其中包含许多不同的实现功能，比如加减乘除，所以我们先新建一个抽象类，再新建不同的实现类

　　抽象运算符号解析器

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 操作符，非终结符表达式
 3  */
 4 public abstract class OperatorExpression extends Expression {
 5     protected Expression left;
 6     protected Expression right;
 7 
 8     public OperatorExpression(Expression left, Expression right)
 9     {
10         this.left = left;
11         this.right = right;
12     }
13 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　具体的解析器

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 加法解析器
 3  */
 4 public class AddExpression extends OperatorExpression {
 5 
 6     public AddExpression(Expression left, Expression right) {
 7         super(left, right);
 8     }
 9 
10     @Override
11     public double Interpret(Context context) {
12         return this.left.Interpret(context) + this.right.Interpret(context);
13     }
14 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 减法解析器
 3  */
 4 public class SubExpression extends OperatorExpression {
 5 
 6     public SubExpression(Expression left, Expression right) {
 7         super(left, right);
 8     }
 9 
10     @Override
11     public double Interpret(Context context) {
12         return this.left.Interpret(context) - this.right.Interpret(context);
13     }
14 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 乘法解析器
 3  */
 4 public class MulExpression extends OperatorExpression {
 5 
 6     public MulExpression(Expression left, Expression right) {
 7         super(left, right);
 8     }
 9 
10     @Override
11     public double Interpret(Context context) {
12         return this.left.Interpret(context) * this.right.Interpret(context);
13     }
14 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 除法解析器
 3  */
 4 public class DivExpression extends OperatorExpression {
 5 
 6     public DivExpression(Expression left, Expression right) {
 7         super(left, right);
 8     }
 9 
10     @Override
11     public double Interpret(Context context) {
12         return this.left.Interpret(context) / this.right.Interpret(context);
13     }
14 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　接下来，新建一个解析器封装类

　　解析器封装类，这个类是根据迪米特法则进行封装，他并不是解释器模式的组成部分，目的是让Client只与功能模块打交道，不涉及到具体的实现，相当于Facade（外观模式）

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class Calculator {
 2     private String expression;
 3     private Context context;
 4 
 5     public Calculator(String expression)
 6     {
 7         this.expression = expression;
 8         this.context = new Context();
 9     }
10 
11     public double Calculate()
12     {
13         char[] vars = this.expression.toCharArray();
14         for(char c : vars){
15             if (c == '+' || c == '-' || c == '*' || c == '/')
16             {
17                 continue;
18             }
19             if (!this.context.getVariable().containsKey(c))
20             {    
21                 System.out.println("请输入一个数字：");
22                 System.out.print(c+"=");
23                 Scanner sn = new Scanner(System.in);
24                 String readLine = sn.next();
25                 this.context.getVariable().put(c, Double.parseDouble(readLine));
26             }
27         }
28         Expression left = new VariableExpression(vars[0]);
29         Expression right = null;
30         Stack<Expression> stack = new Stack<Expression>();
31         stack.push(left);
32         for (int i = 1; i < vars.length; i += 2)
33         {
34             left = stack.pop();
35             right = new VariableExpression(vars[i + 1]);
36             switch (vars[i])
37             {
38                 case '+':
39                     stack.push(new AddExpression(left, right));
40                     break;
41                 case '-':
42                     stack.push(new SubExpression(left, right));
43                     break;
44                 case '*':
45                     stack.push(new MulExpression(left, right));
46                     break;
47                 case '/':
48                     stack.push(new DivExpression(left, right));
49                     break;
50             }
51         }
52         double value = stack.pop().Interpret(this.context);
53         stack.clear();
54         return value;
55     }
56 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　最后，是客户端，解释器的使用者

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 //Client客户端
2 public class Client {
3     public static void main(String[] args) {
4         Calculator c = new Calculator("a*b/c");
5         System.out.println("结果等于" + c.Calculate());
6     }
7 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　运行结果：

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180320153852838-467837187.png)

　　

　　**优点：** 1、可扩展性比较好，灵活。 2、增加了新的解释表达式的方式。 3、易于实现简单文法。

　　**缺点：** 1、可利用场景比较少。 2、对于复杂的文法比较难维护。 3、解释器模式会引起类膨胀。 4、解释器模式采用递归调用方法。



# [17、中介者模式](https://www.cnblogs.com/xiaobai1226/p/8609719.html)

### 概念：

　　Mediator模式也叫中介者模式，是由GoF提出的23种软件设计模式的一种。Mediator模式是行为模式之一，在Mediator模式中，类之间的交互行为被统一放在Mediator的对象中，对象通过Mediator对象同其他对象交互，Mediator对象起着控制器的作用。

　　中介者模式其实就好比租房中介和相亲网站一样，房东将信息发布到租房中介，而租客可以中介挑选自己理想的房子，或者单身男女各自将自己的信息发布到相亲网站，同时也可以挑选符合自己条件的另一半。

　　拿相亲网站举个例子，首先不用中介者模式

　　相亲，肯定是人相亲，所以新建一个Person类，并提供一个getCompanion(Person person)方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public abstract class Person {
 2     private String name;
 3     private int condition;
 4     
 5     public Person(String name, int condition){
 6         this.name = name;
 7         this.condition = condition;
 8     }
 9 
10     public String getName() {
11         return name;
12     }
13 
14     public void setName(String name) {
15         this.name = name;
16     }
17 
18     public int getCondition() {
19         return condition;
20     }
21 
22     public void setCondition(int condition) {
23         this.condition = condition;
24     }
25     
26     //得到是否符合信息
27     public abstract void getCompanion(Person person);
28 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　人肯定是分为男人和女人，所以在新建Man与Woman类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class Man extends Person {
 2 
 3     public Man(String name, int condition) {
 4         super(name, condition);
 5     }
 6 
 7     @Override
 8     public void getCompanion(Person person) {
 9         if(person instanceof Man){
10             System.out.println("我不是gay");
11         }else{
12             if(person.getCondition() == this.getCondition()){
13                 System.out.println(this.getName()+"先生与"+person.getName()+"女士"+"非常般配");
14             }else{
15                 System.out.println(this.getName()+"先生与"+person.getName()+"女士"+"不合适");
16             }
17         }
18     }
19 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class Woman extends Person{
 2 
 3     public Woman(String name, int condition) {
 4         super(name, condition);
 5     }
 6 
 7     @Override
 8     public void getCompanion(Person person) {
 9         if(person instanceof Woman){
10             System.out.println("我不是gay");
11         }else{
12             if(person.getCondition() == this.getCondition()){
13                 System.out.println(this.getName()+"女士与"+person.getName()+"先生"+"非常般配");
14             }else{
15                 System.out.println(this.getName()+"女士与"+person.getName()+"先生"+"不合适");
16             }
17         }
18     }
19 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　在写一个客户端

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainCLass {
 2     public static void main(String[] args) {
 3         Person xiaoming = new Man("小明",1);
 4         Person xiaoqiang = new Man("小强",2);
 5         
 6         Person xiaohong = new Woman("小红",1);
 7         
 8         xiaoming.getCompanion(xiaohong);
 9         
10         xiaoqiang.getCompanion(xiaohong);
11         
12         xiaoming.getCompanion(xiaoqiang);
13     }
14 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　运行结果：

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180321083529298-1032658608.png)

　　可以看到，这种形式判断对方是否合适都交给了男女双方自己来解决，自己去寻找是否符合条件的，像大海捞针一样，不管是男，女人，甚至不是人都要挨个去比较。

　　同时这样Man和Woman之间存在了一个交互行为，大大提高了代码的耦合性，如果这两个类中都有自己特定的方法需要对方调用时，只要一方修改，另一方就需要跟着修改。两个类紧紧联系到了一起，这样的设计是很不好的。

　　下面，正式进入中介者模式

### 中介者模式结构图

　　![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180321090009850-1007592625.png)

　　

### 中介者模式的角色和职责

　　**1、Mediator：**中介者类的抽象父类 
　　　　抽象中介者角色定义统一的接口，用于各角色（男和女）之间的通信。

　　**2、ConcreteMediator：**具体中介者角色 
　　　　具体中介者角色，通过协调各角色（男和女）实现协作行为，因此它必须依赖于各个角色。

　　**3、Colleague：**关联类的抽象父类
　　　　每一个角色都知道中介者角色，而且与其它的角色通信的时候，一定要通过中介者角色来协作。 
　　　　每个类（Person）的行为分为二种（男和女）：一种是男女本身的行为，这种行为叫做自发行为(Self-Method);第二种是必须依赖中介者才能完成的行为，叫做依赖行为(Dep-Method)。

　　**4、concreteColleague：**具体的关联类（Man和Woman）。

　　下面，用代码实现中介者模式

　　首先新建一个**Mediator**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public abstract class Mediator {
 2     private Man man;
 3     private Woman woman;
 4     
 5     public Man getMan() {
 6         return man;
 7     }
 8     public void setMan(Man man) {
 9         this.man = man;
10     }
11     public Woman getWoman() {
12         return woman;
13     }
14     public void setWoman(Woman woman) {
15         this.woman = woman;
16     }
17     
18     //比较条件的方法
19     public abstract void getCompanion(Person person);
20 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　在新建一个**ConcreteMediator：**具体中介者角色

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class BlindDateMediator extends Mediator{
 2     @Override
 3     public void getCompanion(Person person) {
 4         if(person instanceof Man) {
 5             this.setMan((Man)person);
 6         } else {
 7             this.setWoman((Woman)person);
 8         }
 9         
10         //如果是同性
11         if(this.getMan() == null || this.getWoman() == null) {
12             System.out.println("我不是gay");
13         }else {
14             //条件合适
15             if(this.getMan().getCondition() == this.getWoman().getCondition()) {
16                 System.out.println(this.getMan().getName() + "先生与" + this.getWoman().getName() + "女士很般配");
17             //条件不合适
18             }else {
19                 System.out.println(this.getMan().getName() + "先生" + this.getWoman().getName() + "女士不合适");
20             }
21         }
22         
23         //比较之后，将条件置空，不然会影响下一次比较
24         this.setMan(null);
25         this.setWoman(null);
26     }
27 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　接下来，新建Person，Man与Woman

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public abstract class Person {
 2     private String name;
 3     private int condition;
 4     private Mediator mediator;
 5     
 6     public Person(String name, int condition, Mediator mediator){
 7         this.name = name;
 8         this.condition = condition;
 9         this.mediator = mediator;
10     }
11 
12     public String getName() {
13         return name;
14     }
15 
16     public void setName(String name) {
17         this.name = name;
18     }
19 
20     public int getCondition() {
21         return condition;
22     }
23 
24     public void setCondition(int condition) {
25         this.condition = condition;
26     }
27     
28     public Mediator getMediator() {
29         return mediator;
30     }
31 
32     public void setMediator(Mediator mediator) {
33         this.mediator = mediator;
34     }
35 
36     //得到是否符合信息
37     public abstract void getCompanion(Person person);
38 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class Man extends Person {
 2 
 3     public Man(String name, int condition, Mediator mediator) {
 4         super(name, condition, mediator);
 5     }
 6 
 7     @Override
 8     public void getCompanion(Person person) {
 9         this.getMediator().setMan(this);
10         this.getMediator().getCompanion(person);
11     }
12 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class Woman extends Person{
 2 
 3     public Woman(String name, int condition, Mediator mediator) {
 4         super(name, condition, mediator);
 5     }
 6 
 7     @Override
 8     public void getCompanion(Person person) {
 9         this.getMediator().setWoman(this);
10         this.getMediator().getCompanion(person);
11     }
12 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　最后是客户端

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainCLass {
 2     public static void main(String[] args) {
 3         Mediator mediator = new BlindDateMediator();
 4         Person xiaoming = new Man("小明",1,mediator);
 5         Person lisi = new Man("李四",2,mediator);
 6         Person xiaohong = new Woman("小红",1,mediator);
 7         
 8         xiaoming.getCompanion(xiaohong);
 9         
10         lisi.getCompanion(xiaohong);
11         
12         xiaoming.getCompanion(lisi);
13     }
14 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　运行结果如下：

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180321101150485-1148087274.png)

### 中介者模式的优缺点

　　**优点：**

　　1，将系统按功能分割成更小的对象，符合类的最小设计原则

　　2，对关联对象的集中控制

　　3，减小类的耦合程度，明确类之间的相互关系：当类之间的关系过于复杂时，其中任何一个类的修改都会影响到其他类，不符合类的设计的开闭原则 ，而Mediator模式将原来相互依存的多对多的类之间的关系简化为Mediator控制类与其他关联类的一对多的关系，当其中一个类修改时，可以对其他关联

类不产生影响（即使有修改，也集中在Mediator控制类）。

　　4，有利于提高类的重用性

　　**缺点：**

　　中介者会膨胀得很大，而且逻辑复杂，原本N个对象直接的相互依赖关系转换为中介者和同事类的依赖关系，同事类越多，中介者的逻辑就越复杂。

博客是我交流学习的平台，如果大家发现有错误，欢迎大家评论指正。 如果本文对您有帮助也请推荐本文，谢谢大家的点赞，因为您的支持是我学习得最大动力。 同时转载也请注明出处，谢谢！！！



# [18、职责链模式](https://www.cnblogs.com/xiaobai1226/p/8615596.html)

### 概念：

　　Chain of Responsibility（CoR）模式也叫职责链模式、责任链模式或者职责连锁模式，是行为模式之一，该模式构造一系列分别担当不同的职责的类的对象来共同完成一个任务，这些类的对象之间像链条一样紧密相连，所以被称作职责链模式。

###  　职责链模式的应用场景

　　**例1：**比如客户Client要完成一个任务，这个任务包括a,b,c,d四个部分。首先客户Client把任务交给A，A完成a部分之后，把任务交给B，B完成b部分之后，把任务交给C，C完成c部分之后，把任务交给D，直到D完成d部分。

　　**例2：**比如政府部分的某项工作，县政府先完成自己能处理的部分，不能处理的部分交给省政府，省政府再完成自己职责范围内的部分，不能处理的部分交给中央政府，中央政府最后完成该项工作。

　　**例3：**SERVLET容器的过滤器（Filter）框架实现。

　　下面，举一个例子，假如我们要造汽车，制造汽车的流程是先制造车头，再制造车身，最后制造车尾

　　我们先不用职责链模式

　　首先，新建一个制造汽车流程的抽象类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 /*
2  * 制造汽车流程抽象类
3  */
4 public abstract class CarHandler {
5     public abstract void HandlerCar();
6 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　制造车头

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 制造车头
 3  */
 4 public class CarHeadHandler extends CarHandler {
 5 
 6     @Override
 7     public void HandlerCar() {
 8         System.out.println("制造车头");
 9     }
10 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　制造车身

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 制造车身
 3  */
 4 public class CarBodyHandler extends CarHandler {
 5 
 6     @Override
 7     public void HandlerCar() {
 8         System.out.println("制造车身");
 9     }
10 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

 　制造车尾

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 制造车尾
 3  */
 4 public class CarTailHandler extends CarHandler {
 5 
 6     @Override
 7     public void HandlerCar() {
 8         System.out.println("制造车尾");
 9     }
10 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　客户端

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         CarHandler carHead = new CarHeadHandler();
 4         CarHandler carBody = new CarBodyHandler();
 5         CarHandler carTail = new CarTailHandler();
 6         
 7         carHead.HandlerCar();
 8         carBody.HandlerCar();
 9         carTail.HandlerCar();
10     }
11 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　运行结果：

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180321114654660-1979361135.png)

　　可以看到这种形式，制造汽车的每一步都需要用户来完成，用户制造完车头了，再由用户去制造车身，非常麻烦。

　　我们想要的是什么，用户制造汽车，制造完车头了，自动的去制造车身，然后再自动的去制造车尾。这就需要用到职责链模式了。

### 职责链模式的基本条件

　　要实现Chain of Responsibility模式，需要满足该模式的基本条件：

　　1，对象链的组织。需要将某任务的所有职责执行对象以链的形式加以组织。

　　2，消息或请求的传递。将消息或请求沿着对象链传递，以让处于对象链中的对象得到处理机会。

　　3，处于对象链中的对象的职责分配。不同的对象完成不同的职责。

　　4，任务的完成。处于对象链的末尾的对象结束任务并停止消息或请求的继续传递。

### 职责链模式的结构

 ![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180321132928732-2142843546.png)

　　

### 职责链模式的角色和职责

　　**1、抽象处理者(Handler)角色：**定义出一个处理请求的接口。如果需要，接口可以定义 出一个方法以设定和返回对下家的引用。这个角色通常由一个Java抽象类或者Java接口实现。上图中Handler类的聚合关系给出了具体子类对下家的引用，抽象方法handleRequest()规范了子类处理请求的操作。

　　**2、具体处理者(ConcreteHandler)角色：**具体处理者接到请求后，可以选择将请求处理掉，或者将请求传给下家。由于具体处理者持有对下家的引用，因此，如果需要，具体处理者可以访问下家。

　　下面，用职责链模式实现一下刚才的功能

　　首先，新建抽象处理者(Handler)角色也就是制造汽车流程的抽象类，在刚才的基础基础上增加一个成员属性，并提供一个set方法、

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 制造汽车流程抽象类
 3  */
 4 public abstract class CarHandler {
 5     //提供一个CarHandler类型属性，记录下一个要执行的流程
 6     private CarHandler carHandler;
 7     //提供一个getNext方法，用来得到下一个要执行的流程
 8     public CarHandler getNextCarHandler() {
 9         return carHandler;
10     }
11     //提供一个setNext方法，用来给属性赋值
12     public void setNextCarHandler(CarHandler carHandler) {
13         this.carHandler = carHandler;
14     }
15     
16     public abstract void HandlerCar();
17 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　接着新建三个流程

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 制造车头
 3  */
 4 public class CarHeadHandler extends CarHandler {
 5 
 6     @Override
 7     public void HandlerCar() {
 8         System.out.println("制造车头");
 9         //如果当前流程不是最后一个流程，继续执行下一个流程
10         if(this.getNextCarHandler() != null){
11             this.getNextCarHandler().HandlerCar();
12         }
13     }
14 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 制造车身
 3  */
 4 public class CarBodyHandler extends CarHandler {
 5 
 6     @Override
 7     public void HandlerCar() {
 8         System.out.println("制造车身");
 9         //如果当前流程不是最后一个流程，继续执行下一个流程
10         if(this.getNextCarHandler() != null){
11             this.getNextCarHandler().HandlerCar();
12         }
13     }
14 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 制造车尾
 3  */
 4 public class CarTailHandler extends CarHandler {
 5 
 6     @Override
 7     public void HandlerCar() {
 8         System.out.println("制造车尾");
 9         //如果当前流程不是最后一个流程，继续执行下一个流程
10         if(this.getNextCarHandler() != null){
11             this.getNextCarHandler().HandlerCar();
12         }
13     }
14 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　最后是客户端

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         CarHandler carHead = new CarHeadHandler();
 4         CarHandler carBody = new CarBodyHandler();
 5         CarHandler carTail = new CarTailHandler();
 6         
 7         //组装顺序预先设定好，顺序是车头，车身，车尾（真正开发也可以封装成一个单独的功能模块）
 8         carHead.setNextCarHandler(carBody);
 9         carBody.setNextCarHandler(carTail);
10         //调用职责链模式的链头来完成操作
11         carHead.HandlerCar();
12     }
13 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

### 职责链模式的优缺点

　　**优点：**

　　**1、**责任的分担。每个类只需要处理自己该处理的工作（不该处理的传递给下一个对象完成），明确各类的责任范围，符合类的最小封装原则。

　　**2、**可以根据需要自由组合工作流程。如工作流程发生变化，可以通过重新分配对象链便可适应新的工作流程。

　　**3、**类与类之间可以以松耦合的形式加以组织。

　　**缺点：**

　　因为处理时以链的形式在对象间传递消息，根据实现方式不同，有可能会影响处理的速度。

# [19、迭代模式](https://www.cnblogs.com/xiaobai1226/p/8617511.html)

### 概念：

　　Iterator模式也叫迭代模式，是行为模式之一，它把对容器中包含的内部对象的访问委让给外部类，使用Iterator（遍历）按顺序进行遍历访问的设计模式。

　　迭代模式使用比较少，JDK集合也提供了Iterator的具体实现，可以直接拿来用，不必自己实现

　　在应用Iterator模式之前，首先应该明白Iterator模式用来解决什么问题。或者说，如果不使用Iterator模式，会存在什么问题。

　　　1.由容器自己实现顺序遍历。直接在容器类里直接添加顺序遍历方法

　　　2.让调用者自己实现遍历。直接暴露数据细节给外部

 　我们用代码实现一下

　　首先，新建一个Book类，这是容器中的内容

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class Book {
 2     private String id;
 3     private String name;
 4     private double price;
 5     
 6     public Book(String id, String name, double price) {
 7         this.id = id;
 8         this.name = name;
 9         this.price = price;
10     }
11 
12     public String getId() {
13         return id;
14     }
15 
16     public void setId(String id) {
17         this.id = id;
18     }
19 
20     public String getName() {
21         return name;
22     }
23 
24     public void setName(String name) {
25         this.name = name;
26     }
27 
28     public double getPrice() {
29         return price;
30     }
31 
32     public void setPrice(double price) {
33         this.price = price;
34     }
35     
36     public void display() {
37         System.out.println("ID=" + id + ",\tname=" + name + ",\tprice" + price);
38     }
39 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　再新建一个容器

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class BookList {
 2     //容器内部还是一个List，也可以用数组
 3     private List<Book> bookList = new ArrayList<Book>();
 4     private int index;
 5     
 6     //添加书籍
 7     public void addBook(Book book){
 8         bookList.add(book);
 9     }
10     
11     //删除书籍
12     public void removeBook(Book book){
13         int bookIndex = bookList.indexOf(book);
14         bookList.remove(bookIndex);
15     }
16     
17     //判断是否有下一本书
18     public boolean hasNext(){
19         if(index >= bookList.size()){
20             return false;
21         }
22         return true;
23     }
24     
25     //获得下一本书
26     public Book getNext(){
27         return bookList.get(index++);
28     }
29     
30     //获取集合长度
31     public int getSize(){
32         return bookList.size();
33     }
34     
35     //根据index获取Book
36     public Book getByIndex(int index){
37         return bookList.get(index);
38     }
39 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　接下来，就是迭代容器了，先采用第一种方式**（由容器自己实现顺序遍历。直接在容器类里直接添加顺序遍历方法）**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class MainClass {
    public static void main(String[] args) {
        BookList bookList = new BookList();
        
        Book book1 = new Book("001","设计模式",200);
        Book book2 = new Book("002","Java核心编程",200);
        Book book3 = new Book("003","计算机组成原理",200);
        
        bookList.addBook(book1);
        bookList.addBook(book2);
        bookList.addBook(book3);
        
        while(bookList.hasNext()){
            Book book = bookList.getNext();
            book.display();
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

 　结果如下：

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180321162544744-1526616940.png)

 

　　然后是第二种方式**（让调用者自己实现遍历。直接暴露数据细节给外部）**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         BookList bookList = new BookList();
 4         
 5         Book book1 = new Book("001","设计模式",200);
 6         Book book2 = new Book("002","Java核心编程",200);
 7         Book book3 = new Book("003","计算机组成原理",200);
 8         
 9         bookList.addBook(book1);
10         bookList.addBook(book2);
11         bookList.addBook(book3);
12         
13         for(int i = 0; i < bookList.getSize(); i++){
14             Book book = bookList.getByIndex(i);
15             book.display();
16         }
17     }
18 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　结果同上

　　但这样，一定是有缺点的，不然就没有必要使用迭代模式了

### 不使用迭代模式的缺点

　　以上方法1与方法2都可以实现对遍历，但有这些问题

　　1，容器类承担了太多功能：一方面需要提供添加删除等本身应有的功能；一方面还需要提供遍历访问功能。

　　2，往往容器在实现遍历的过程中，需要保存遍历状态，当跟元素的添加删除等功能夹杂在一起，很容易引起混乱和程序运行错误等。

### 应用迭代模式的条件

　　Iterator模式就是为了有效地处理按顺序进行遍历访问的一种设计模式，简单地说，Iterator模式提供一种有效的方法，可以屏蔽聚集对象集合的容器类的实现细节，而能对容器内包含的对象元素按顺序进行有效的遍历访问。

　　所以，Iterator模式的应用场景可以归纳为满足以下几个条件：

　　1、访问容器中包含的内部对象
　　2、按顺序访问

### 迭代模式的结构

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180323095900288-2019357495.png)

　　

### 迭代模式的角色和职责

　　**1、Iterator****（迭代器接口）**：

　　该接口必须定义实现迭代功能的最小定义方法集

　　比如提供hasNext()和next()方法。

　　**2、ConcreteIterator****（迭代器实现类）**：

　　迭代器接口Iterator的实现类。可以根据具体情况加以实现。

　　**3、Aggregate****（容器接口）**：

　　定义基本功能以及提供类似Iterator iterator()的方法。

　　**4、concreteAggregate****（容器实现类）**：

　　容器接口的实现类。必须实现Iterator iterator()方法。

 　接下来，用代码实现一下迭代模式，只需修改BookList即可

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class BookList {
 2     //容器内部还是一个List，也可以用数组
 3     private List<Book> bookList = new ArrayList<Book>();
 4     private int index;
 5     
 6     //添加书籍
 7     public void addBook(Book book){
 8         bookList.add(book);
 9     }
10     
11     //删除书籍
12     public void removeBook(Book book){
13         int bookIndex = bookList.indexOf(book);
14         bookList.remove(bookIndex);
15     }
16     
17     //判断是否有下一本书
18 //    public boolean hasNext(){
19 //        if(index >= bookList.size()){
20 //            return false;
21 //        }
22 //        return true;
23 //    }
24     
25     //获得下一本书
26 //    public Book getNext(){
27 //        return bookList.get(index++);
28 //    }
29     
30     //获取集合长度
31     public int getSize(){
32         return bookList.size();
33     }
34     
35     //根据index获取Book
36     public Book getByIndex(int index){
37         return bookList.get(index);
38     }
39     
40     //得到Iterator实例
41     public Iterator Iterator() {
42         return new Itr();
43     }
44     
45     //内部类，Iterator实例（因为要使用容器的内部信息，所以要写成内部类）
46     private class Itr implements Iterator{
47         //判断是否有下一本书,将刚才hasNext()中内容复制过来即可
48         public boolean hasNext() {
49             if(index >= bookList.size()){
50                 return false;
51             }
52             return true;
53         }
54         //获得下一本书,将刚才getNext()中内容复制过来即可
55         public Object next() {
56             return bookList.get(index++);
57         }
58 
59         public void remove() {
60             
61         }
62     }
63 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　再在客户端实现一下

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         BookList bookList = new BookList();
 4         
 5         Book book1 = new Book("001","设计模式",200);
 6         Book book2 = new Book("002","Java核心编程",200);
 7         Book book3 = new Book("003","计算机组成原理",200);
 8         
 9         bookList.addBook(book1);
10         bookList.addBook(book2);
11         bookList.addBook(book3);
12         
13         Iterator iterator = bookList.Iterator();
14         while(iterator.hasNext()){
15             Book book = (Book) iterator.next();
16             book.display();
17         }
18     }
19 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　可以看到，这和使用JDK提供集合的Iterator方法就一模一样了。

### 迭代模式的优缺点

　　**优点：**

　　1，实现功能分离，简化容器接口。让容器只实现本身的基本功能，把迭代功能委让给外部类实现，符合类的设计原则。

　　2，隐藏容器的实现细节。

　　3，为容器或其子容器提供了一个统一接口，一方面方便调用；另一方面使得调用者不必关注迭代器的实现细节。

　　4，可以为容器或其子容器实现不同的迭代方法或多个迭代方法。

　　**缺点:**

　　由于迭代器模式将存储数据和遍历数据的职责分离，增加新的聚合类需要对应增加新的迭代器类，类的个数成对增加，这在一定程度上增加了系统的复杂性。

# [20、模板方法模式](https://www.cnblogs.com/xiaobai1226/p/8629441.html)

### 　　概念：

　　Template Method模式也叫模板方法模式，是行为模式之一，它把具有特定步骤算法中的某些必要的处理委让给抽象方法，通过子类继承对抽象方法的不同实现改变整个算法的行为。

### 　　模板方法模式的应用场景

　　Template Method模式一般应用在具有以下条件的应用中：

　　1、具有统一的操作步骤或操作过程

　　2、具有不同的操作细节

　　3、存在多个具有同样操作步骤的应用场景，但某些具体的操作细节却各不相同

### 　　模板方法模式的应用实例

　　 1、在造房子的时候，地基、走线、水管都一样，只有在建筑的后期才有加壁橱加栅栏等差异。

　　 2、西游记里面菩萨定好的 81 难，这就是一个顶层的逻辑骨架。

　　3、spring 中对 Hibernate 的支持，将一些已经定好的方法封装起来，比如开启事务、获取 Session、关闭 Session 等，程序员不重复写那些已经规范好的代码，直接丢一个实体就可以保存。

 

### 　　模板方法模式的结构

 ![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180323133913932-2075816994.png)

### 　　模板方法模式的角色和职责

　　**1、AbstractClass：**抽象类的父类

　　**2、ConcreteClass：**具体的实现子类

　　**3、templateMethod()：**模板方法，具体步骤方法的执行顺序（步骤）

　　**4、method1()与method2()：**具体步骤方法（细节）

 　下面用代码来实现一下：我们举个例子，加入我们要组装汽车，步骤是，先组装车头，再组装车身，最后组装车尾

　　这样，我们先建造**AbstractClass（其中包含template模板，执行顺序）**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
/*
 * 组装车（AbstractClass）
 */
public abstract class MakeCar {
    //组装车头
    public abstract void makeCarHead();
    
    //组装车身
    public abstract void makeCarBody();
    
    //组装车尾
    public abstract void makeCarTail();
    
    //汽车组装流程（template()）
    public void makeCar(){
        this.makeCarHead();
        this.makeCarBody();
        this.makeCarTail();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　再新建**ConcreteClass（具体的实现细节）**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 //组装公交车
 2 public class MakeBus extends MakeCar{
 3 
 4     @Override
 5     public void makeCarHead() {
 6         System.out.println("组装公交车车头");
 7     }
 8 
 9     @Override
10     public void makeCarBody() {
11         System.out.println("组装公交车车身");
12     }
13 
14     @Override
15     public void makeCarTail() {
16         System.out.println("组装公交车车尾");
17     }
18 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 组装SUV
 3  */
 4 public class MakeSuv extends MakeCar{
 5 
 6     @Override
 7     public void makeCarHead() {
 8         System.out.println("组装SUV车头");
 9     }
10 
11     @Override
12     public void makeCarBody() {
13         System.out.println("组装SUV车身");
14     }
15 
16     @Override
17     public void makeCarTail() {
18         System.out.println("组装SUV车尾");
19     }
20 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　最后运行一下

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         MakeCar makeBus = new MakeBus();
 4         makeBus.makeCar();
 5         
 6         System.out.println("===========================");
 7         
 8         MakeCar makeSuv = new MakeSuv();
 9         makeSuv.makeCar();
10     }
11 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　运行结果：

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180323150908788-766821178.png)

 

 

 　这样，用户不必关心具体的执行流程（步骤）了。

### 　　模板方法模式的优缺点：

　　**优点：** 1、封装不变部分，扩展可变部分。 2、提取公共代码，便于维护。 3、行为由父类控制，子类实现。

　　**缺点：**每一个不同的实现都需要一个子类来实现，导致类的个数增加，使得系统更加庞大。

### 　　**注意事项：**

　　为防止恶意操作，一般模板方法都加上 final 关键词。

# [21、备忘录模式](https://www.cnblogs.com/xiaobai1226/p/8630879.html)

### 　　概念：

　　Memento模式也叫备忘录模式又叫做快照模式（Snapshot Pattern）或Token模式，是GoF的23种设计模式之一，属于行为模式，它的作用是保存对象的内部状态，并在需要的时候（undo/rollback）恢复对象以前的状态。

　　在不破坏封闭的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态。这样以后就可将该对象恢复到原先保存的状态。

###  　备忘录模式的结构

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180323165006219-1077488159.png)

 

### 　　备忘录模式的角色和职责

　　**1、Originator（原生者）：**需要被保存状态以便恢复的那个对象，负责创建一个备忘录Memento，用以记录当前时刻自身的内部状态，并可使用备忘录恢复内部状态。Originator可以根据需要决定Memento存储自己的哪些内部状态。

　　**2、Memento（备忘录）：**该对象由Originator创建，主要负责存储Originator对象的内部状态，并可以防止Originator以外的其他对象访问备忘录。备忘录有两个接口：Caretaker只能看到备忘录的窄接口，他只能将备忘录传递给其他对象。Originator却可看到备忘录的宽接口，允许它访问返回到先前状

态所需要的所有数据。

　　**3、Caretaker（管理者）：**负责在适当的时间保存/恢复Originator对象的状态，负责备忘录Memento，不能对Memento的内容进行访问或者操作。

### 　　备忘录模式的应用场景

　　如果一个对象需要保存状态并可通过undo或rollback等操作恢复到以前的状态时，可以使用Memento模式。

　　（1）一个类需要保存它的对象的状态（相当于Originator角色）
　　（2）设计一个类，该类只是用来保存上述对象的状态（相当于Memento角色）
　　（3）需要的时候，Caretaker角色要求Originator返回一个Memento并加以保存
　　（4）undo或rollback操作时，通过Caretaker保存的Memento恢复Originator对象的状态

 　下面用代码来实现一下

　　首先，新建一个Person类，他就是**Originator（原生者）**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * Originator（原生者）
 3  */
 4 public class Person {
 5     /*
 6      * 姓名，年龄，性别就是结构图中的state，状态属性
 7      */
 8     //姓名
 9     private String name;
10     //年龄
11     private int age;
12     //性别
13     private String sex;
14 
15     public Person() {
16         super();
17     }
18 
19     public Person(String name, int age, String sex) {
20         super();
21         this.name = name;
22         this.age = age;
23         this.sex = sex;
24     }
25 
26     public String getName() {
27         return name;
28     }
29 
30     public void setName(String name) {
31         this.name = name;
32     }
33 
34     public int getAge() {
35         return age;
36     }
37 
38     public void setAge(int age) {
39         this.age = age;
40     }
41 
42     public String getSex() {
43         return sex;
44     }
45 
46     public void setSex(String sex) {
47         this.sex = sex;
48     }
49     
50     public void display() {
51         System.out.println("name:" + name + ",sex:" + sex + ",age:" + age);
52     }
53     
54     //创建一个备份
55     public Memento createMemento(){
56         return new Memento(name,age,sex);
57     }
58     
59     //恢复一个备份
60     public void setMemento(Memento memento){
61         this.name = memento.getName();
62         this.age = memento.getAge();
63         this.sex = memento.getSex();
64     }
65 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　接着，创建一个**Memento（备忘录），**他和Originator的基本结构是一样的

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * Memento（备忘录）
 3  */
 4 public class Memento {
 5     //姓名
 6     private String name;
 7     //年龄
 8     private int age;
 9     //性别
10     private String sex;
11 
12     public Memento() {
13         super();
14     }
15 
16     public Memento(String name, int age, String sex) {
17         super();
18         this.name = name;
19         this.age = age;
20         this.sex = sex;
21     }
22 
23     public String getName() {
24         return name;
25     }
26 
27     public void setName(String name) {
28         this.name = name;
29     }
30 
31     public int getAge() {
32         return age;
33     }
34 
35     public void setAge(int age) {
36         this.age = age;
37     }
38 
39     public String getSex() {
40         return sex;
41     }
42 
43     public void setSex(String sex) {
44         this.sex = sex;
45     }
46 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　接下来，创建客户端运行一下

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         Person person = new Person("小明",18,"男");
 4         //打印
 5         person.display();
 6         
 7         //创建一个备份
 8         Memento memento = person.createMemento();
 9         
10         person.setName("小红");
11         person.setAge(17);
12         person.setSex("女");
13         //打印
14         person.display();
15         
16         //备份还原
17         person.setMemento(memento);
18         //打印
19         person.display();
20     }
21 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　结果：

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180326085446972-2145912389.png)

　　这样，就做到了一个备份还原的过程，但是可以看到，根据结构图看，我们还缺少了一个**Caretaker（管理者）**角色

　　这个角色的作用其实就是把备份还原的过程交到管理者手中，通过管理者来备份还原

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * Caretaker（管理者）
 3  */
 4 public class Caretaker {
 5     //持有一个对Memento的引用
 6     private Memento memento;
 7 
 8     public Memento getMemento() {
 9         return memento;
10     }
11 
12     public void setMemento(Memento memento) {
13         this.memento = memento;
14     }
15 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　客户端

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         Person person = new Person("小明",18,"男");
 4         //打印
 5         person.display();
 6         
 7         //创建一个管理者
 8         Caretaker caretaker = new Caretaker();
 9         
10         //创建一个备份
11 //        Memento memento = person.createMemento();
12         caretaker.setMemento(person.createMemento());
13         
14         person.setName("小红");
15         person.setAge(17);
16         person.setSex("女");
17         //打印
18         person.display();
19         
20         //备份还原
21         person.setMemento(caretaker.getMemento());
22         //打印
23         person.display();
24     }
25 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　运行结果是一样的

### 　　备忘录模式的优点和缺点

　　**一、备忘录模式的优点**

　　　　1、有时一些发起人对象的内部信息必须保存在发起人对象以外的地方，但是必须要由发起人对象自己读取，这时，使用备忘录模式可以把复杂的发起人内部信息对其他的对象屏蔽起来，从而可以恰当地保持封装的边界。

　　　　2、本模式简化了发起人类。发起人不再需要管理和保存其内部状态的一个个版本，客户端可以自行管理他们所需要的这些状态的版本。

　　　　3、当发起人角色的状态改变的时候，有可能这个状态无效，这时候就可以使用暂时存储起来的备忘录将状态复原。

　　**二、备忘录模式的缺点：**

　　　　1、如果发起人角色的状态需要完整地存储到备忘录对象中，那么在资源消耗上面备忘录对象会很昂贵。

　　　　2、当负责人角色将一个备忘录存储起来的时候，负责人可能并不知道这个状态会占用多大的存储空间，从而无法提醒用户一个操作是否很昂贵。

　　　　3、当发起人角色的状态改变的时候，有可能这个协议无效。如果状态改变的成功率不高的话，不如采取“假如”协议模式。



# [22、状态模式](https://www.cnblogs.com/xiaobai1226/p/8650619.html)

### 　　概念：

　　State模式也叫状态模式，是行为设计模式的一种。State模式允许通过改变对象的内部状态而改变对象的行为，这个对象表现得就好像修改了它的类一样。

　　根据这个概念，我们举个例子

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class Behavior {
 2     private int time;
 3 
 4     public int getTime() {
 5         return time;
 6     }
 7 
 8     public void setTime(int time) {
 9         this.time = time;
10     }
11     
12     public void eat(){
13         if(time == 7){
14             System.out.println("吃早饭");
15         }else if(time == 12){
16             System.out.println("吃午饭");
17         }else if(time == 18){
18             System.out.println("吃晚饭");
19         }else{
20             System.out.println("还不到吃饭时间");
21         }
22     }
23 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         Behavior behavior = new Behavior();
 4         behavior.setTime(7);
 5         behavior.eat();
 6         
 7         behavior.setTime(12);
 8         behavior.eat();
 9         
10         behavior.setTime(18);
11         behavior.eat();
12         
13         behavior.setTime(20);
14         behavior.eat();
15     }
16 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　结果：

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180326110152183-2041589471.png)

　　可以看到，根据time属性的不同，对象的行为也发生了改变，但是这样的方式很不好，所有的事情都放到了eat（）方法中，导致eat（）方法过于复杂

　　下面就看一看状态模式

### 　　状态模式的应用场景

　　状态模式主要解决的是当控制一个对象状态转换的条件表达式过于复杂时的情况。把状态的判断逻辑转译到表现不同状态的一系列类当中，可以把复杂的判断逻辑简化。

　　简单来说：

　　1.一个对象的行为取决于它的状态，并且它必须在运行时刻根据状态改变它的行为。

　　2.一个操作中含有庞大的多分支结构，并且这些分支决定于对象的状态。

### 　　状态模式的结构

 ![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180326142709885-679678231.png)

　　

### 　　状态模式的角色和职责

　　**1、Context：**用户对象：拥有一个State类型的成员，以标识对象的当前状态**（Behavior）**。

　　**2、State：**接口或基类封装与Context的特定状态相关的行为；

　　**3、ConcreteState：**接口实现类或子类实现了一个与Context某个状态相关的行为。

 　按照状态模式，我们来改造一下，刚才的例子，吃早中晚饭，不是吃饭时间，都是状态，所以我们把状态单独封装出来。

　　首先，新建一个**State**

```
1 public abstract class State {
2     public abstract void eat();
3 }
```

　　接着新建**ConcreteState**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 早餐
 3  */
 4 public class BreakfastState extends State {
 5 
 6     @Override
 7     public void eat() {
 8         System.out.println("吃早餐");
 9     }
10 
11 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 午餐
 3  */
 4 public class LunchState extends State {
 5 
 6     @Override
 7     public void eat() {
 8         System.out.println("吃午餐");
 9     }
10 
11 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 晚餐
 3  */
 4 public class DinnerState extends State {
 5 
 6     @Override
 7     public void eat() {
 8         System.out.println("吃晚餐");
 9     }
10 
11 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 不是吃饭时间
 3  */
 4 public class NoFoodState extends State {
 5 
 6     @Override
 7     public void eat() {
 8         System.out.println("不是吃饭时间");
 9     }
10 
11 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　再修改一下behavior

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class Behavior {
 2     private int time;
 3     State state  = null;
 4 
 5     public int getTime() {
 6         return time;
 7     }
 8 
 9     public void setTime(int time) {
10         this.time = time;
11     }
12     
13     public void eat(){
14         if(time == 7){
15             state = new BreakfastState();
16             state.eat();
17         }else if(time == 12){
18             state = new LunchState();
19             state.eat();
20         }else if(time == 18){
21             state = new DinnerState();
22             state.eat();
23         }else{
24             state = new NoFoodState();
25             state.eat();
26         }
27     }
28 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这样，和刚才的结果一样，但是这样子，判断逻辑还是在对象中，我们继续修改，将逻辑写到**ConcreteState**中

　　因为，我们要知道time，所以需要向state中传入参数，所以我们将Behavior传进去

```
1 public abstract class State {
2     public abstract void eat(Behavior behavior);
3 }
```

　　然后，修改Behavior

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class Behavior {
 2     private int time;
 3     State state  = null;
 4 
 5     public Behavior() {
 6         state = new BreakfastState();
 7     }
 8 
 9     public int getTime() {
10         return time;
11     }
12 
13     public void setTime(int time) {
14         this.time = time;
15     }
16     
17     public State getState() {
18         return state;
19     }
20 
21     public void setState(State state) {
22         this.state = state;
23     }
24 
25     public void eat(){
26         //逻辑取出，所以这里只剩调用方法
27         state.eat(this);
28         //当所有方法都完成后，回到最初始状态
29         state = new BreakfastState();
30     }
31 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　接着，再继续修改每一个**ConcreteState**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 早餐
 3  */
 4 public class BreakfastState extends State {
 5 
 6     @Override
 7     public void eat(Behavior behavior) {
 8         if(behavior.getTime() == 7){
 9             System.out.println("吃早餐");
10         }else{
11             //如果不符合条件，重置state为下一个状态
12             behavior.setState(new LunchState());
13             behavior.eat();
14         }
15     }
16 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 午餐
 3  */
 4 public class LunchState extends State {
 5 
 6     @Override
 7     public void eat(Behavior behavior) {
 8         if(behavior.getTime() == 12){
 9             System.out.println("吃午餐");
10         }else{
11             behavior.setState(new DinnerState());
12             behavior.eat();
13         }
14     }
15 
16 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 晚餐
 3  */
 4 public class DinnerState extends State {
 5 
 6     @Override
 7     public void eat(Behavior behavior) {
 8         if(behavior.getTime() == 18){
 9             System.out.println("吃晚餐");
10         }else{
11             behavior.setState(new NoFoodState());
12             behavior.eat();
13         }
14     }
15 
16 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 不是吃饭时间
 3  */
 4 public class NoFoodState extends State {
 5 
 6     @Override
 7     public void eat(Behavior behavior) {
 8         System.out.println("不是吃饭时间");
 9     }
10 
11 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　这样，结果和之前是一样的

### 　　状态模式的优点与缺点

　　**优点：** 1、封装了转换规则。

　　　　　 2、枚举可能的状态，在枚举状态之前需要确定状态种类。

　　　　　 3、将所有与某个状态有关的行为放到一个类中，并且可以方便地增加新的状态，只需要改变对象状态即可改变对象的行为。

　　　　　 4、允许状态转换逻辑与状态对象合成一体，而不是某一个巨大的条件语句块。

　　　　　 5、可以让多个环境对象共享一个状态对象，从而减少系统中对象的个数。

　　**缺点：** 1、状态模式的使用必然会增加系统类和对象的个数。

　　　　　 2、状态模式的结构与实现都较为复杂，如果使用不当将导致程序结构和代码的混乱。

　　　　  3、状态模式对"开闭原则"的支持并不太好，对于可以切换状态的状态模式，增加新的状态类需要修改那些负责状态转换的源代码，否则无法切换到新增状态，而且修改某个状态类的行为也需修改对应类的源代码。

 **注意事项：**在行为受状态约束的时候使用状态模式，而且状态不超过 5 个。

# [23、命令模式](https://www.cnblogs.com/xiaobai1226/p/8651632.html)

### 　　概念：

　　Command模式也叫命令模式 ，是行为设计模式的一种。Command模式通过被称为Command的类封装了对目标对象的调用行为以及调用参数。

　　命令模式（Command Pattern）是一种数据驱动的设计模式，它属于行为型模式。请求以命令的形式包裹在对象中，并传给调用对象。调用对象寻找可以处理该命令的合适的对象，并把该命令传给相应的对象，该对象执行命令。

　　**主要解决：**

　　在软件系统中，“行为请求者”与“行为实现者”通常呈现一种“紧耦合”。但在某些场合，比如要对行为进行“记录、撤销/重做、事务”等处理，这种无法抵御变化的紧耦合是不合适的。在这种情况下，如何将“行为请求者”与“行为实现者”解耦？将一组行为抽象为对象，实现二

者之间的松耦合。这就是命令模式（Command Pattern）。

　　下面，举一个小例子，但不是使用命令模式实现的

　　新建一个Peddler类，是一个卖水果的小商贩

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 小商贩
 3  */
 4 public class Peddler {
 5     /*
 6      * 卖苹果
 7      */
 8     public void sailApple(){
 9         System.out.println("卖苹果");
10     }
11     
12     /*
13      * 卖香蕉
14      */
15     public void sailBanana(){
16         System.out.println("卖香蕉");
17     }
18 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　再建一个客户端

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class MainClass {
2     public static void main(String[] args) {
3         Peddler peddler = new Peddler();
4         peddler.sailApple();
5         peddler.sailBanana();
6     }
7 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　结果：

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180327092721695-2101146240.png)

### 　　命令模式的应用场景

　　在面向对象的程序设计中，一个对象调用另一个对象，一般情况下的调用过程是：创建目标对象实例；设置调用参数；调用目标对象的方法（像刚才的例子MainClass中，创建Peddler实例，再调度其中的方法）。

　　但在有些情况下有必要使用一个专门的类对这种调用过程加以封装，我们把这种专门的类称作command类。

　　1、整个调用过程比较繁杂，或者存在多处这种调用。这时，使用Command类对该调用加以封装，便于功能的再利用。

　　2、调用前后需要对调用参数进行某些处理。

　　3、调用前后需要进行某些额外处理，比如日志，缓存，记录历史操作等。

### 　　命令模式的结构

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180326162822391-1404514628.png)

　　

### 　　命令模式的角色和职责

　　**1、Command：**Command抽象类，定义命令的接口，声明执行的方法。。

　　**2、ConcreteCommand：**Command的具体实现类，命令接口实现对象，是“虚”的实现；通常会持有接收者，并调用接收者的功能来完成命令要执行的操作。

　　**3、Receiver：**需要被调用的目标对象，接收者，真正执行命令的对象。任何类都可能成为一个接收者，只要它能够实现命令要求实现的相应功能。

　　**4、Invorker：**通过Invorker执行Command对象，要求命令对象执行请求，通常会持有命令对象，可以持有很多的命令对象。这个是客户端真正触发命令并要求命令执行相应操作的地方，也就是说相当于使用命令对象的入口。

　　**5、Client：**创建具体的命令对象，并且设置命令对象的接收者。注意这个不是我们常规意义上的客户端，而是在组装命令对象和接收者，或许，把这个Client称为装配者会更好理解，因为真正使用命令的客户端是从Invoker来触发执行。

　

　　当然，我们的例子非常简单，用不到命令模式，我们假设操作很复杂，用命令模式改造一下

　　首先，创建类对调用过程进行一个封装，先创建一个Command抽象父类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public abstract class Command {
 2     //包含一个Receiver的引用
 3     private Peddler peddler;
 4     
 5     public Command(Peddler peddler) {
 6         super();
 7         this.peddler = peddler;
 8     }
 9 
10     public Peddler getPeddler() {
11         return peddler;
12     }
13 
14     public void setPeddler(Peddler peddler) {
15         this.peddler = peddler;
16     }
17 
18     public abstract void sail();
19 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　接下来创建**ConcreteCommand**，每一个操作都要创建一个，所以我们要创建两个Apple与Banana

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * Apple
 3  */
 4 public class AppleCommand extends Command{
 5 
 6     public AppleCommand(Peddler peddler) {
 7         super(peddler);
 8     }
 9 
10     @Override
11     public void sail() {
12         //还可以在这句话前后做额外的处理
13         this.getPeddler().sailApple();
14     }
15 
16 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * Banana
 3  */
 4 public class BananaCommand extends Command{
 5 
 6     public BananaCommand(Peddler peddler) {
 7         super(peddler);
 8     }
 9 
10     @Override
11     public void sail() {
12         //还可以在这句话前后做额外的处理
13         this.getPeddler().sailBanana();
14     }
15 
16 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　最后，修改MainClass

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         Peddler peddler = new Peddler();
 4         Command appleCommand = new AppleCommand(peddler);
 5         Command bananaCommand = new BananaCommand(peddler);
 6         
 7         appleCommand.sail();
 8         bananaCommand.sail();
 9         
10     }
11 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　但是这样还不够好，可以看到这是直接由命令来调用sail方法的![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180327103807237-358464412.png)，这样不好，我们希望由专人来卖东西，就像人家请了一个服务员一样，所以我们再新建一个**Invorker**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class Waiter {
    private Command command;

    public Command getCommand() {
        return command;
    }

    public void setCommand(Command command) {
        this.command = command;
    }
    
    public void sail(){
        command.sail();
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　再修改客户端

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         Peddler peddler = new Peddler();
 4         Command appleCommand = new AppleCommand(peddler);
 5         Command bananaCommand = new BananaCommand(peddler);
 6         
 7         Waiter waiter = new Waiter();
 8         waiter.setCommand(appleCommand);
 9         waiter.sail();
10         
11         waiter.setCommand(bananaCommand);
12         waiter.sail();
13         
14     }
15 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　这样**Client**就直接与**Invorker**进行通话

　　我们现在只能添加命令，却不能移除命令，，而且一次只能有一个命令，我们继续改造

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class Waiter {
 2     private Map<String,Command> commands;
 3 
 4     public Map<String, Command> getCommands() {
 5         if(commands == null){
 6             commands = new HashMap<String,Command>();
 7         }
 8         
 9         return commands;
10     }
11 
12     public void setCommand(String key, Command command) {
13         if(commands == null){
14             commands = new HashMap<String,Command>();
15         }
16         commands.put(key, command);
17     }
18     
19     public void RemoveCommand(String key) {
20         commands.remove(key);
21     }
22 
23     public void sail(String key){
24         commands.get(key).sail();
25     }
26 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　Client

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         Peddler peddler = new Peddler();
 4         Command appleCommand = new AppleCommand(peddler);
 5         Command bananaCommand = new BananaCommand(peddler);
 6         
 7         Waiter waiter = new Waiter();
 8         waiter.setCommand("apple", appleCommand);
 9         waiter.setCommand("banana", bananaCommand);
10         
11         waiter.sail("apple");
12         waiter.sail("banana");
13         
14         waiter.RemoveCommand("apple");
15         
16     }
17 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这样，就完成了，一个命令模式的例子

### 　　命令模式的优缺点

　　**优点：** 1、降低了系统耦合度。

　　　　　 2、新的命令可以很容易添加到系统中去。

　　**缺点：**使用命令模式可能会导致某些系统有过多的具体命令类。

# [24、访问者模式](https://www.cnblogs.com/xiaobai1226/p/8656969.html)

### 　　概念：

　　Visitor模式也叫访问者模式，是行为模式之一，它分离对象的数据和行为，使用Visitor模式，可以不修改已有类的情况下，增加新的操作。

### 　　访问者模式的应用示例

　　比如有一个公园，有一到多个不同的组成部分；该公园存在多个访问者：清洁工A负责打扫公园的A部分，清洁工B负责打扫公园的B部分，公园的管理者负责检点各项事务是否完成，上级领导可以视察公园等等。也就是说，对于同一个公园，不同的访问者有不同的行为操作，而且访问者的种类也可能需要根据时间的推移而变化（行为的扩展性）。根据软件设计的开闭原则（对修改关闭，对扩展开放），我们怎么样实现这种需求呢？

### 　　访问者模式的结构

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180327135141099-1806273739.png)

### 　　访问者模式的角色和职责

　　**1、 访问者角色（Visitor）：**为该对象结构中具体元素角色声明一个访问操作接口。该操作接口的名字和参数标识了发送访问请求给具体访问者的具体元素角色。这样访问者就可以通过该元素角色的特定接口直接访问它。

　　**2、具体访问者角色（Concrete Visitor）：**实现每个由访问者角色（Visitor）声明的操作。

　　**3、元素角色（Element）：**定义一个Accept操作，它以一个访问者为参数。

　　**4、具体元素角色（Concrete Element）：**实现由元素角色提供的Accept操作。

　　**5、对象结构角色（Object Structure）：**这是使用访问者模式必备的角色。它要具备以下特征：能枚举它的元素；可以提供一个高层的接口以允许该访问者访问它的元素；可以是一个复合（组合模式）或是一个集合，如一个列表或一个无序集合。

 

　　下面，用代码来实现一下访问者模式

　　首先，先新建一个公园的各个组成部分（因为有了各个组成部分才能有公园，所以我们先建各个组成部分，再建公园）

　　新建一个公园各个部分的抽象类**（\**元素角色\**Element），**需要传入一个访问者，一会后面会创建访问者

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 /*
2  * 公园每一部分的抽象
3  */
4 public interface ParkElement {
5     //用来接纳访问者
6     public void accept(Visitor visitor);
7 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　再新建公园的各个部分，这是**具体元素角色（Concrete Element）**

　　***\*关键代码：\**在数据基础类里面有一个方法接受访问者，将自身引用传入访问者。**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class ParkA implements ParkElement{
2 
3     @Override
4     public void accept(Visitor visitor) {
5         visitor.visit(this);
6     }
7 
8 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 public class ParkB implements ParkElement{
2 
3     @Override
4     public void accept(Visitor visitor) {
5         visitor.visit(this);
6     }
7 
8 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　有了公园的各个部分，我们就可以开始新建公园了，这就是**对象结构角色（Object Structure）**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 整个公园，其中包含了公园的各个部分
 3  */
 4 public class Park implements ParkElement{
 5     private ParkA parkA;
 6     private ParkB parkB;
 7     
 8     public Park() {
 9         super();
10         this.parkA = new ParkA();
11         this.parkB = new ParkB();
12     }
13 
14     @Override
15     public void accept(Visitor visitor) {
16         visitor.visit(this);
17         parkA.accept(visitor);
18         parkB.accept(visitor);
19     }
20 
21 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　有了公园，我们就可以开始建造访问者了，先建造**访问者角色（Visitor）**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 /*
2  * 其中包含访问公园不同区域的重载方法,其中重载的参数Park，ParkA，PArkB，不仅可以获得公园的信息，还用来作为重载的条件
3  */
4 public interface Visitor {
5     public void visit(Park park);
6     public void visit(ParkA parkA);
7     public void visit(ParkB parkB);
8 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　然后在建造具体访问者角色，就是**具体访问者角色（Concrete Visitor）**

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 清洁工A负责公园A的清洁情况
 3  */
 4 public class VisitorA implements Visitor{
 5 
 6     @Override
 7     public void visit(Park park) {
 8     }
 9 
10     @Override
11     public void visit(ParkA parkA) {
12         System.out.println("清洁工A完成了公园A的卫生");
13     }
14 
15     @Override
16     public void visit(ParkB parkB) {
17     }
18 
19 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 清洁工B负责公园B的清洁情况
 3  */
 4 public class VisitorB implements Visitor{
 5 
 6     @Override
 7     public void visit(Park park) {
 8     }
 9 
10     @Override
11     public void visit(ParkA parkA) {
12     }
13 
14     @Override
15     public void visit(ParkB parkB) {
16         System.out.println("清洁工B完成了公园B的卫生");
17     }
18 
19 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 /*
 2  * 公园管理员负责检查整个公园的卫生情况
 3  */
 4 public class VisitorManager implements Visitor{
 5 
 6     @Override
 7     public void visit(Park park) {
 8         System.out.println("管理员：负责公园的卫生检查");
 9     }
10 
11     @Override
12     public void visit(ParkA parkA) {
13         System.out.println("管理员：负责公园A部分的卫生检查");
14     }
15 
16     @Override
17     public void visit(ParkB parkB) {
18         System.out.println("管理员：负责公园B部分的卫生检查");
19     }
20 
21 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　最后，新建一个客户端

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 public class MainClass {
 2     public static void main(String[] args) {
 3         Park park = new Park();
 4         
 5         Visitor visitorA = new VisitorA();
 6         park.accept(visitorA);
 7         
 8         Visitor visitorB = new VisitorB();
 9         park.accept(visitorB);
10         
11         VisitorManager visitorManager = new VisitorManager();
12         park.accept(visitorManager);
13     }
14 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　结果：

![img](https://images2018.cnblogs.com/blog/1227182/201803/1227182-20180327154825054-1349021790.png)

　　这样，一个访问者模式的例子就完成了，一定把上面的程序看懂

### 　 访问者模式的使用意图：

　　主要将数据结构与数据操作分离，解决了稳定的数据结构和易变的操作耦合问题。　　　

　 **访问者模式的使用场景：**

　　需要对一个对象结构中的对象进行很多不同的并且不相关的操作，而需要避免让这些操作"污染"这些对象的类，使用访问者模式将这些封装到类中。

 

 

　　1、对象结构中对象对应的类很少改变，但经常需要在此对象结构上定义新的操作。

　　2、需要对一个对象结构中的对象进行很多不同的并且不相关的操作，而需要避免让这些操作"污染"这些对象的类，也不希望在增加新操作时修改这些类。

### 　 **访问者模式的优缺点：**

　　**优点：**

　　1、符合单一职责原则。

　　2、优秀的扩展性。

　　3、灵活性。

　　**缺点：** 

　　1、具体元素对访问者公布细节，违反了迪米特原则。

　　2、具体元素变更比较困难。

　　3、违反了依赖倒置原则，依赖了具体类，没有依赖抽象。

### 　　**注意事项：** 

　　访问者可以对功能进行统一，可以做报表、UI、拦截器与过滤器。