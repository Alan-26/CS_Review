# 异常处理机制



## Java类的继承体系



![image-20210305084604756](https://gitee.com/lgaaip/img/raw/master/20210305084613.png)

`Throwable`是异常的顶层父类，代表所有的非正常情况。它有两个直接子类，分别是**Error、Exception**。

**Error **  是错误，一般是指与虚拟机相关的问题，如系统崩溃、虚拟机错误、动态链接失败等，这种错误无法恢复或不可能捕获，将导致应用程序中断。通常应用程序无法处理这些错误，因此应用程序不应该试图使用catch块来捕获Error对象。在定义方法时，也无须在其throws子句中声明该方法可能抛出Error及其任何子类。

**Exception **   是异常，它被分为两大类，分别是`Checked异常`（受检异常）和`Runtime异常`（运行时异常）。所有的RuntimeException类及其子类的实例被称为Runtime异常；不是RuntimeException类及其子类的异常实例则被称为Checked异常。Java认为Checked异常都是可以被处理（修复）的异常，所以Java程序必须显式处理Checked异常。如果程序没有处理Checked异常，该程序在编译时就会发生错误，无法通过编译。Runtime异常则更加灵活，Runtime异常无须显式声明抛出，如果程序需要捕获Runtime异常，也可以使用try...catch块来实现



## 如何处理异常？

在Java中，可以按照如下三个步骤处理异常：

1. 捕获异常

将业务代码包裹在try块内部，当业务代码中发生任何异常时，系统都会为此异常创建一个异常对象。创建异常对象之后，JVM会在try块之后寻找可以处理它的catch块，并将异常对象交给这个catch块处理。

2. 处理异常

在catch块中处理异常时，应该先记录日志，便于以后追溯这个异常。然后根据异常的类型、结合当前的业务情况，进行相应的处理。比如，给变量赋予一个默认值、直接返回空值、向外抛出一个新的业务异常交给调用者处理，等等。

3. 回收资源

如果业务代码打开了某个资源，比如数据库连接、网络连接、磁盘文件等，则需要在这段业务代码执行完毕后关闭这项资源。并且，无论是否发生异常，都要尝试关闭这项资源。将关闭资源的代码写在finally块内，可以满足这种需求，即无论是否发生异常，finally块内的代码总会被执行。



## 说一说Java的异常机制

### **关于异常处理：**

​	在Java中，处理异常的语句由try、catch、finally三部分组成。其中，try块用于包裹业务代码，catch块用于捕获并处理某个类型的异常，finally块则用于	回收资源。当业务代码发生异常时，系统会创建一个异常对象，然后由JVM寻找可以处理这个异常的catch块，并将异常对象交给这个catch块处理。若业	务代码打开了某项资源，则可以在finally块中关闭这项资源，因为无论是否发生异常，finally块一定会执行。

### **关于抛出异常：**

​	当程序出现错误时，系统会自动抛出异常。除此以外，Java也允许程序主动抛出异常。当业务代码中，判断某项错误的条件成立时，可以使用throw关键	字向外抛出异常。在这种情况下，如果当前方法不知道该如何处理这个异常，可以在方法签名上通过throws关键字声明抛出异常，则该异常将交给JVM	      	处理。

> Throw和Throws的区别

**位置不同**

1. throws 用在函数上，后面跟的是异常类，可以跟多个；而 throw 用在函数内，后面跟的是异常对象。

**功能不同：**

2. throws 用来声明异常，让调用者只知道该功能可能出现的问题，可以给出预先的处理方式；throw 抛出具体的问题对象，执行到 throw，功能就已经结束了，跳转到调用者，并将具体的问题对象抛给调用者。也就是说 throw 语句独立存在时，下面不要定义其他语句，因为执行不到。

3. throws 表示出现异常的一种可能性，并不一定会发生这些异常；throw 则是抛出了异常，执行 throw 则一定抛出了某种异常对象。
4. 两者都是消极处理异常的方式，只是抛出或者可能抛出异常，但是不会由函数去处理异常，真正的处理异常由函数的上层调用处理。



### **关于异常跟踪栈：**

​	程序运行时，经常会发生一系列方法调用，从而形成方法调用栈。异常机制会导致异常在这些方法之间传播，而异常传播的顺序与方法的调用相反。异	常从发生异常的方法向外传播，首先传给该方法的调用者，再传给上层调用者，以此类推。最终会传到main方法，若依然没有得到处理，则JVM会终止	程序，并打印异常跟踪栈的信息



## finally了解过吗？

不管try块中的代码是否出现异常，也不管哪一个catch块被执行，甚至在try块或catch块中执行了return语句，finally块总会被执行。

> 如果在try块或catch块中使用 System.exit(1); 来退出虚拟机，则finally块将失去执行的机会。但是我们在实际的开发中，重来都不会这样做，所以尽管存在这种导致finally块无法执行的可能，也只是一种可能而已。

那么我们来看下面一段代码：

```java
    int test(){
        int a = 1;
        try {
            int i = 1/0; //制造异常
        }catch (Exception e){
            return a;
        }finally {
            a++;
        }
        return 0;
    }
```

执行结果会是什么样的呢？

在上面我们说到，在try块或catch块中执行了return语句，finally块总会被执行。所以我们可能会以为finally中执行了a++后，此时a变成了2，然后catch再去返回a，应该结果是2才对。那么我们来看一下结果：

![image-20210305091820019](https://gitee.com/lgaaip/img/raw/master/20210305091821.png)

很幸运，回答错了。所以这是我们需要注意的点，下面我们说下原因。

其实是这样的，在执行finally语句之前，try 或者 catch 语句块会保留其返回值到本地变量表（Local Variable Table）中。待 finally 执行完毕之后，再恢复保留的返回值到操作数栈中，然后通过 return 或者 throw 语句将其返回给该方法的调用者（invoker）。

> 还有一个问题，finally中return会发生什么问题？

在通常情况下，不要在finally块中使用return、throw等导致方法终止的语句，一旦在finally块中使用了return、throw语句，将会导致try块、catch块中的return、throw语句失效。

**详细解析**：当Java程序执行try块、catch块时遇到了return或throw语句，这两个语句都会导致该方法立即结束，但是系统执行这两个语句并不会结束该方法，而是去寻找该异常处理流程中是否包含finally块，如果没有finally块，程序立即执行return或throw语句，方法终止；如果有finally块，系统立即开始执行finally块。只有当finally块执行完成后，系统才会再次跳回来执行try块、catch块里的return或throw语句；如果finally块里也使用了return或throw等导致方法终止的语句，finally块已经终止了方法，系统将不会跳回去执行try块、catch块里的任何代码。



> 经典面试题：final、finally和 finalize 之间的区别？

**1.简单区别：**
final用于声明属性，方法和类，分别表示属性不可交变，方法不可覆盖，类不可继承。
finally是异常处理语句结构的一部分，表示总是执行。
finalize是Object类的一个方法，在垃圾收集器执行的时候会调用被回收对象的此方法，供垃圾收集时的其他资源回收，例如关闭文件等。
**2.区别：**
**final：Java中的关键字，修饰符。**
A).如果一个类被声明为final，就意味着它不能再派生出新的子类，不能作为父类被继承。因此，一个类不能同时被声明为abstract抽象类的和final的类。
B).如果将变量或者方法声明为final，可以保证它们在使用中不被改变.
　　1)被声明为final的变量必须在声明时给定初值，而在以后的引用中只能读取，不可修改。
　　2)被声明final的方法只能使用，不能重载。
**finally：Java的一种异常处理机制。**
finally是对Java异常处理模型的最佳补充。finally结构使代码总会执行，而不管无异常发生。使用finally可以维护对象的内部状态，并可以清理非内存资源。特别是在关闭数据库连接这方面，如果程序员把数据库连接的close()方法放到finally中，就会大大降低程序出错的几率。
**finalize：Java中的一个方法名。**
Java技术使用finalize()方法在垃圾收集器将对象从内存中清除出去前，做必要的清理工作。这个方法是由垃圾收集器在确定这个对象没被引用时对这个对象调用的。它是在Object类中定义的，因此所的类都继承了它。子类覆盖finalize()方法以整理系统资源或者执行其他清理工作。finalize()方法是在垃圾收集器删除对象之前对这个对象调用的。

