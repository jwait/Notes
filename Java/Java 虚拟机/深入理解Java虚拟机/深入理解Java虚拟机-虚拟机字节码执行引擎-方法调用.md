## 深入理解 Java 虚拟机——虚拟机字节码执行引擎

> 方法调用并不等同于方法执行,方法调用阶段唯一的任务就是确定被调用方法的版本(即调用哪一个方法),暂时还不涉及方法内部的具体运行过程
>
> Class 文件的编译过程中不包含传统编译中的连接步骤,一切方法调用在 Class 文件中存储的都是符号引用,而不是方法实际运行时内存布局中的方法入口地址



### 方法调用字节码指令

Java  虚拟机中提供了 5 条 方法调用字节码指令,分别如下:

* **invokestatic**：调用静态方法
* **invokespecial**：调用实例构造器`<init>`方法、私有方法和父类方法
* **invokevirtual**:调用所有的虚方法
* **invokeinterface**：调用接口方法，会在运行时在确定一个实现此接口的对象
* **invokedynamic**：先在运行时动态解析出调用点限定符所引出的方法，然后在执行该方法，在此之前的4条调用指令，分派逻辑是固=固化在 Java 虚拟机内部的，而invokedynamic 指令的分派逻辑是由用户所设定的引导方法决定的



### 虚方法与非虚方法

* **非虚方法**：可以在解析阶段中就唯一确定调用的版本，它们在类加载阶段就会把符号引用解析为方法的直接引用。符合这些条件的有：静态方法、私有方法、实例构造器、父类方法（final 方法也是非虚方法，尽管使用 invokevirtual 指令调用）。只能被 invokestatic 和 invokespacial 指令调用
* 除去以上 4 种方法以及 final 方法的其他方法称为虚方法 



### 解析调用(Resolution)

* 解析调用指的是：在类加载的解析阶段就会把涉及的符号引用全部转换为可确定的直接引用
* 这种解析能成立的前提是：方法在程序真正运行之前就会有一个可确定的调用版本，并且方法的调用版本在运行期间是不可改变的。
* 在 Java 语言种符合“编译器可知，运行期不可变”这个要求的方法，主要包括静态方法和私有方法两大类



### 分派调用(dispatch)

* 分派调用可能是静态的，也可能是动态的，根据分派依据的宗量数可分为：单分派和多分派
* Java 语言是静态多分派、动态单分派

##### 一些知识点：

###### 宗量：

​	方法的接收者（执行该方法的对象）与方法的参数称为方法的宗量

###### 静态类型与实际类型：

```java
Human man = new man();
```

* 以上代码中的`Human`称为变量的静态类型(Static Type)，或者是称为：外观类型(Apparent Type)，后面的`man`则称为变量的实际类型(Actual Type)
* 静态类型和实际类型都可以发生变化，区别是静态类型的变化仅仅发生在使用时(强制转换)，变量本身的静态类型并未发生改变，因而变量的静态类型在编译期是可知的；而实际类型需要在运行期才能够被确定，编译器在编译期间并不知道变量的实际类型是什么

#### 静态分派

* 所有依赖静态类型来定位方法执行版本的分派动作称为静态分派
* 静态分派的典型应用是**方法重载**(`overload`)
* 静态分派发生在编译阶段

###### 示例：

```java
public class StaticDispatch {

	
	public void sayHello(Human guy) {
		System.out.println("hello, guy!");
	}
	
	public void sayHello(Man guy) {
		System.out.println("hello, man!");
	}
	
	public void sayHello(Women guy) {
		System.out.println("hello, women!");
	}
	
	public static void main(String[] args) {
		
		Human man = new Man();
		
		Human women = new Women();
		
		StaticDispatch sd = new StaticDispatch();
		
		sd.sayHello(man);
		
		sd.sayHello(women);

	}

}

class Human {
	
}

class Man extends Human {
	
}

class Women extends Human {
	
}
```

###### 输出：

```
hello, guy!
hello, guy!
```

以上的代码中，由于方法的接受者已经确定是StaticDispatch的实例sd了，所以最终调用的是哪个重载版本也就取决于传入参数的类型了。

而且实际上，虚拟机（应该说是编译器）在重载时是通过参数的静态类型来当判定依据的，而且静态类型在编译期就可知，所以编译器在编译阶段就可根据静态类型来判定究竟使用哪个重载版本。因此对于例子中的两个方法的调用都是以Human为参数的版本。

#### 动态分派

* 动态分派，它和多态的另外一个重要体现——**方法重写**(`override`)有很大的关联

###### 示例：

```java
public class DynamicDispatch {  
          
    static abstract class Human {  
        protected abstract void sayHello();  
    }  
      
    static class Man extends Human {  
          
        @Override  
        protected void sayHello() {  
            System.out.println("hello man!");  
        }          
    }  
       
    static class Women extends Human {  
       
        @Override  
        protected void sayHello() {  
            System.out.println("hello women!");  
        }          
    }  
       
      
    public static void main(String[] args){  
          
        Human man = new Man();  
        Human women = new Women();  
           
        man.sayHello();  
        women.sayHello();  
          
        man = new Women();  
        man.sayHello();  
   
    }  
   
}  
```

###### 输出：

```
hello man!
hello women!
hello women!
```

以上代码中，执行`man.sayHello();`和`women.sayHello(); `时，字节码中对应着的是 invokevirtual 指令，这两条调用指令从字节角度来看，无论指令（invokevirtual）还是参数完全一样，但是这两条指令最终执行的目标方法并不相同，原因需要从invokevirtual指令的多态查找过程开始说起：

**invokevirtual指令的运行时解析过程大致如下几个步骤：**

1. 找到操作数栈顶的第一个元素所指向的对象的实际类型，记作C
2. 如果在类型C中找到与常量池中描述符和简单名称都相符的方法，则进行访问权限的校验，如果校验不通过，则返回`java.lang.IllegaAccessError`异常，校验通过则直接返回方法的直接引用，查找过程结束。
3. 否则，按照继承关系从下往上一次对C的各个父类进行第二步骤的搜索和验证过程。
4. 如果始终还是没有找到合适的方法直接引用，则抛出`java.lang.AbstractMethodError`异常。

由于invokevirtual指令执行的第一步是在运行时确定接收者的实际类型，所以两次中的invokevirtual指令把常量池中的类方法符号引用解析到不同的直接引用上，这个就是java语言中方法重写的本质



#### 静态多分派与动态单分派

* 多分派是根据多于一个宗量对目标方法进行选择，即根据方法的：接收者实际类型（方法重写）和方法参数（方法重载）
* 单分派是只根据一个宗量对目标方法进行选择

###### 示例：

```java
package test;  
  
public class Dispatch {  
  
    static class Ipad{}  
    static class Iphone{}  
    public static class Father{  
        public void hardChoice(Ipad arg){  
            System.out.println("Father choice Ipad!!!");  
        }  
        public void hardChoice(Iphone arg){  
            System.out.println("Father choice Iphone!!!");  
        }  
    }  
    public static class Son extends Father{  
        public void hardChoice(Ipad arg){  
            System.out.println("Son choice Ipad!!!");  
        }  
        public void hardChoice(Iphone arg){  
            System.out.println("Son choice Iphone!!!");  
        }  
    }  
    public static void main(String[] args) {  
        // TODO Auto-generated method stub  
       Father father = new Father();  
       Son son = new Son();  
         
       father.hardChoice(new Ipad());  
       son.hardChoice(new Iphone());  
    }  
  
}  
```

###### 输出：

```
Father choice Ipad!!!
Son choice Iphone!!!
```

* 下面看看编译阶段编译器的选择过程，也就是静态分派的过程。

  这个时候，选择目标方法依据两点：

        一个是静态类型Father和Son，

        二是方法参数Ipad和Iphone。

  这次选择的结果的最终产物是产生了两条invokevirtual指令，两条指令的参数分别为常量池中指向	father.hardChoice(ipad）及 son.hardChoice(iphone)方法的符号引用。因为是根据两个宗量进行选择，所以Java语言的静态分派属于多分派类型。

* 再看看运行阶段虚拟机的选择，也就是动态分派的过程。

       在执行son.hardChoice(new Iphone())这句代码时，由于编译期已经决定目标方法的签名必须为hardChoice(Iphone)，虚拟机不会关心传递过来的参数到底是什么，因为这时参数的静态类型、实际类型都对方法的选择不会构成任何影响，唯一可以影响虚拟机选择的因素只有此方法的接受者的实际类型到底是Father还是Son。因为只有一个宗量作为选择依据，所以Java语言的动态分派属于单分派类型。



### 虚拟机动态分派的实现

* 由于动态分派是非常频繁的动作，而且动态分派的方法版本选择过程需要运行时在类的方法元数据中搜索合适的目标方法，因此在虚拟机的实际实现中基于性能的考虑，大部分实现都不会直接真正进行如此频繁的搜索。
* 对于以上的这种情况，最常用的“稳定优化”手段就是为类在方法区中建立一个虚方法表（Vritual Method Table，与此对应，invokeinterface 指令执行时也会用到接口方法表——Interface Method Table），使用虚方法表索引来代替元数据查找以提高性能。

###### 虚方法表结构：

![](http://img.blog.csdn.net/20140902155336264?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvenE2MDIzMTY0OTg=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

虚方法表中存放着各个方法的实际入口地址。如果某个方法在子类中没有被重写，那子类的虚方法表里面的地址入口和父类相同方法的地址入口时一致的，都指向父类的实现入口，如果子类重写了这个方法，子类方法表中的地址将会替换为指向子类实现版本的入口地址。图中Son重写了来之Father的全部方法，因此Son所以的方法表没有指向父类Father类型数据的箭头。但是Son和Father都没有重写来自Object的方法，所以它们的方法表中所有的从Object继承来的方法都指向了Object的数据类型。

方法表一般在类加载的连接阶段进行初始化，准备了类的变量初始化后，虚拟机会把该类的方法表也初始化完毕。方法表是分派调用的“稳定优化”手段，虚拟机除了使用方法表外，在条件允许的情况下，还会使用内联缓存（Inner Cache）和基于“类型继承关系分析”技术的守护内联（Guarded Inlining）。

