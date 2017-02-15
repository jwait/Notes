# Java String 的两种定义方式

String 类型是 Java 编程当中使用的较多的一种数据类型之一，如果需要定义一个 String 类型变量有两种方式：第一种是使用new 关键字，第二种则是较为简洁地使用双引号

两种定义方式的形式如下：

```java
String str1 = "abc";
String str2 = new string("abc");
```

看上去，以上两种定义方式都是定义了一个值为 `abc` 的 String 对象

那么实际上以上两种定义方式有什么区别呢？还是说以上两种定义方式是等价的？对于这个问题，先考虑以下的一段程序：

```java
String str1 = "cat";
String str2 = "cat";
String str3 = new String("cat");

System.out.println(str1 == str2);
System.out.ptintln(str1 == str3);
```

程序执行后输出的内容是：

```
true
false
```

到这里，或许就是让人困惑的地方了，由于 `==` 运算符进行的是内存地址的比较，因而根据输出的结果我们可以得出 `str1` 与  `str2` 引用的是同一个 String 对象，但 `str1`  与 `str3` 则是引用了不同的 String 对象。那么，既然 `str1` 、 `str2`、  `str3` 表示的都是 `cat` 这个字符串，但为什么以上语句会出现不同的比较结果呢？

简单来说，使用 `""` 虚拟机会隐式地创建 String 对象，并且相同的字符串必须引用相同的String；而使用 `new ` 是显式地创建一个新的 String 对象，与隐式创建的 String 对象无关



详细的，首先我们参考以下 **Java 语言规范** 以及 **Java 虚拟机规范**：

原文参照：http://rednaxelafx.iteye.com/blog/774673

The Java Virtual Machine Specification, Second Edition

> A new class instance may be implicitly created in the following situations: 
>
> ● Loading of a class or interface that contains a String literal may create a new String object [(§2.4.8)](http://java.sun.com/docs/books/jvms/second_edition/html/Concepts.doc.html#25486) to represent that literal. This may not occur if the a String object has already been created to represent a previous occurrence of that literal, or if the String.intern method has been invoked on a String object representing the same string as the literal.
>
> ● Execution of a string concatenation operator that is not part of a constant expression sometimes creates a new String object to represent the result. String concatenation operators may also create temporary wrapper objects for a value of a primitive type [(§2.4.1)](http://java.sun.com/docs/books/jvms/second_edition/html/Concepts.doc.html#19511).
>
> Each of these situations identifies a particular constructor to be called with specified arguments (possibly none) as part of the class instance creation process. 

> **5.1 The Runtime Constant Pool** 
>
> ... 
>
> ● A string literal [(§2.3)](http://java.sun.com/docs/books/jvms/second_edition/html/Concepts.doc.html#20359) is derived from a CONSTANT_String_info structure [(§4.4.3)](http://java.sun.com/docs/books/jvms/second_edition/html/ClassFile.doc.html#29297) in the binary representation of a class or interface. The CONSTANT_String_info structure gives the sequence of Unicode characters constituting the string literal. 
>
> ● The Java programming language requires that identical string literals (that is, literals that contain the same sequence of characters) must refer to the same instance of class String. In addition, if the method String.intern is called on any string, the result is a reference to the same class instance that would be returned if that string appeared as a literal. Thus, 
>
> ```java
> ("a" + "b" + "c").intern() == "abc"  
> ```
>
> must have the value true. 
>
> ● To derive a string literal, the Java virtual machine examines the sequence of characters given by the CONSTANT_String_info structure. 
>
>   ○ If the method String.intern has previously been called on an instance of class String containing a sequence of Unicode characters identical to that given by the CONSTANT_String_info structure, then the result of string literal derivation is a reference to that same instance of class String. 
>
>   ○ Otherwise, a new instance of class String is created containing the sequence of Unicode characters given by the CONSTANT_String_info structure; that class instance is the result of string literal derivation. Finally, the intern method of the new String instance is invoked. 
>
> ... 
>
> The remaining structures in the constant_pool table of the binary representation of a class or interface, the CONSTANT_NameAndType_info [(§4.4.6)](http://java.sun.com/docs/books/jvms/second_edition/html/ClassFile.doc.html#1327) and CONSTANT_Utf8_info [(§4.4.7)](http://java.sun.com/docs/books/jvms/second_edition/html/ClassFile.doc.html#7963) structures are only used indirectly when deriving symbolic references to classes, interfaces, methods, and fields, and when deriving string literals.

String.intern()的 JavaDoc：

> public String intern() 
>
>     Returns a canonical representation for the string object. 
>
>     A pool of strings, initially empty, is maintained privately by the class String. 
>
>     When the intern method is invoked, if the pool already contains a string equal to this String object as determined by the equals(Object) method, then the string from the pool is returned. Otherwise, this String object is added to the pool and a reference to this String object is returned. 
>
>     It follows that for any two strings s and t, s.intern() == t.intern() is true if and only if s.equals(t) is true. 
>
>     All literal strings and string-valued constant expressions are interned. String literals are defined in §3.10.5 of the Java Language Specification 
>
>     **Returns:** 
>         a string that has the same contents as this string, but is guaranteed to be from a pool of unique strings.



从以上的 Java 虚拟机规范中，我们可以得出的信息是:

一、String 类被隐式创建的情形：

* 在 Java 虚拟机 加载一个类或接口时，虚拟机会为这类或接口中的字符串字面量(String literal) 创建一个新的 String 对象来表示这个字面量。但是：
  * 若这个字符串字面量在之前已经出现过，也就是创建了对应的 String 对象，那么将不会为这个字符串字面量创建一个新的 String 对象
  * 若有一个String 对象表示的字符串与这个字符串字面量(String literal) 一样，且这个 String 对象调用了 `intern()` 方法，也不会创建一个新的 String 对象
* 在进行字符串拼接时，如果不是常量表达式拼接(即如："hello" + "world")，那么将会创建一个新的 String 对象来表示拼接的结果

二、运行时常量池(Runtime Constant Pool)中关于字符串字面量(String liteal)的内容

* 字符串字面量(String liteal)是从类或接口的二进制 class 文件中的 `CONSTANT_String_info` 数据结构中得到的，`CONSTANT_String_info` 包含了由 Unicode 码序列组成的字符串字面量

* Java 程序规定相同的字符串字面量(String literal)必须引用相同的 String 实例。此外，任意一个 String 调用了 String.intern() 方法，返回的将是与这个 String 值相同的字符串字面量(String literal) 所对应的 String对象的引用

  因此由：

  ```java
  ("a" + "b" + "c").intern() == "abc"
  ```

* 为了得到字符串字面量(String literal)，JVM 将会检查 `CONSTANT_String_info` 结构中给出的字符序列

  * 如果之前已经有一个包含相同字节序列的 String 类实例调用了 `String.intern()` 方法。那么这个字符串字面量(String literal)将引用这个 String 对象
  * 否则将创建一个包含 `CONSTANT_String_info` 给定字符序列的新的 String 对象。最后调用这个新的 String 类实例的 `intern()` 方法

三、 `String.intern()` 方法：

* 方法返回的是字符串对象规范化的表现形式

* 由 String 类私有维护一个初始为空的 `String Pool`

  当调用 `intern()` 方法时，如果池中已经存在一个与这个 String 对象相同的 String （值相同，通过 `equals()` 方法比较），那么池中的 String 将会返回，否则将这个 String 对象加入到 String pool 中，并返回这个对象的引用

  因此有：`s.intern() == t.intern()` 为真，当且仅当 `s.equals(t)` 为真

* 所有字符串字面量(String literal)和字符串的常量表达式(String-valued constant expressions) 都会被 intern，即被调用 `intern()` 方法



到这里应该对类加载到 String 对象被创建的过程有一点概念了，不过在总结创建过程之前，很有必要说明以下两个容易混淆的概念 `Runtime Constant Pool` 和 `String Pool` ，两者是不一样的

**Runtime Constant Pool**：

* Runtime Constant Pool ，也就是运行时常量池，是方法区的一部分。用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后进入方法区的运行时常量池中存放
* 由于运行时常量池是方法区的一部分，受到方法区的内存限制，当常量池无法再申请到内存是会抛出 `OutOfMemoryError` 异常

**String Pool**：

* String Pool，顾名思义是 String 的一个池， 位于堆内存中，专门用来存放 String 对象，每一个 String 对象都是唯一的。任何一个 String 对象执行 intern 操作后都会放入到这个 String Pool 中
* 由于 String 是不可变的，所以设置 String Pool 可以使 String 对象重用，从而节省内存空间



由于 String Pool 的存在，我们可以很容易地实现 String 对象的重用，从而节省内存。但是 `String str1 = "cat";` 语句中对 String 对象的创建再类加载阶段就完成，而不是等到语句执行时才创建对象

因此上面 String 对象的创建过程是：

1. 类加载阶段，Java 虚拟机从类或接口的 class 文件中的 `CONSTANT_String_info` 结构中读出一个字符串字面量(String literal)
2. 将这个字符串字面量(String literal)放入到运行时常量池(Runtime Constant Pool)中，同时检查 String Pool  中是否存在一个值与这个字符串字面量相同的 String 对象
3. 如果存在，则结束
4. 否则，由 JVM 隐式地为这个字符串字面量(String literal)创建一个 String 对象，然后调用这个新创建 String 对象的 `intern()` 方法，将这个 String 对象 intern 到 String Pool。此后如果有一个相同的字符串字面量，则不必再创建对象



而 `String str3 = new String("cat");` 这一语句的执行与普通的对象创建一样，是用户自定义对象创建语句。在程序执行到这一语句时，在堆内存中创建一个类的实例，并且在构造方法的参数中传入值。因此 `String str3 = new String("cat");` 实际上是在堆中创建了一个新的 String 对象，且值为 `cat`

这一过程从编译后的字节码可以看出：

​		![](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2016-10-9/StringTest%20%E7%BC%96%E8%AF%91%E5%90%8E%20class%20%E6%96%87%E4%BB%B6.PNG)

(首先，执行 new 命令，创建一个 String 对象，然后 dup，ldc 命令把值压入栈顶，也就是传入参数，然后再是执行对象初始化操作，与普通对象的创建一样)

因此可以认为 `String str3 = new String("cat");` 创建的是一个 copy String 对象

> public String(String original)
>
> Initializes a newly created `String` object so that it represents the same sequence of characters as the argument; in other words, the newly created string is a copy of the argument string. Unless an explicit copy of `original` is needed, use of this constructor is unnecessary since Strings are immutable.
>
> - Parameters: 
>
>   `original` - A `String` 





最终我们清晰地得到如下关系：

​		![](https://raw.githubusercontent.com/KEN-LJQ/MarkdownPics/master/Resource/2016-10-9/String-Pool-Java1-450x249.png)

`str1` 和 `str2` 实际上是引用在类加载阶段被创建并且 intern 到 String Pool 中的 String 对象，而 `str3` 则是引用在运行时执行该语句后在堆中创建的普通 String 对象，只不过是与`str1` 和 `str2` 的值相同而已



参考网址：

http://www.journaldev.com/797/what-is-java-string-pool

http://rednaxelafx.iteye.com/blog/774673

http://stackoverflow.com/questions/23252767/string-pool-vs-constant-pool