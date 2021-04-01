---
title: "Continue和break的label"
date: 2021-03-30T15:52:09+08:00
draft: false
toc: true
tags: ['Java','JDK']
---



最近在复习整理JDK（JDK版本：java-11-openjdk-11.0.6.10-2.windows.redhat.x86_64）源码时，在ArrayList的源码中，第2次看到了关于`break`的label的语法。

<!--more-->

## continue和break

`break`和`continue`在控制流程中用于中断循环，`break`用于退出**当前所在的循环**，`continue`用于跳过**当前循环步骤** **，执行下一次循环步骤**，都达到了中断控制流程的目的。

在一般的使用中，`break`和`continue`后面只有分号，没有其他标识。

下面程序片段为关于`continue`的代码片段，打印0~9中为奇数的数字

```java
for (int j = 0; j < 10; j++) {
    if(j%2 == 0) continue;
    System.out.println(j);
}
```

在循环中，当j为偶数时则执行continue，后面一句打印输出不执行，直接执行下一次j++。

## continue和break的label

最近在复习整理JDK（JDK版本：java-11-openjdk-11.0.6.10-2.windows.redhat.x86_64）源码时，在ArrayList的源码中，第2次看到了关于break的label的语法，第1次是在看Spring MVC的源码时（忘了整理，不记得是哪段代码），当时看到这个语法，查了一下，但并没有放在心上，也没有去整理。

### 代码实例

先来看1个代码例子，以下代码是ArrayList源码中的remove方法

```java
public boolean remove(Object o) {//删除o元素（在ArrayList中，可以有空的值）
    final Object[] es = elementData;
    final int size = this.size;
    int i = 0;
    //---label begin---
    found: {//在es中查找是否有o这个元素
        if (o == null) {//如果o为空
            for (; i < size; i++)
                if (es[i] == null)
                    break found;//在es中找到第1个元素为空，退出label即found括起来的代码块
        } else {
            for (; i < size; i++)//如果o不为空
                if (o.equals(es[i]))
                    break found;//在es找到了第1个和o相等的元素，退出label即found括起来的代码块
        }
        return false;
    }
    //---label end---
    fastRemove(es, i);//执行删除操作
    return true;
 }
```

如果不是很清楚这个label语法的作用，根据这个方法的需求，大概也能推理猜测到方法的执行过程。

执行过程参见代码注释。

整体看代码的话，`found`开始查找，`break found;`中断查找，从语法上确实比较容易理解。

再来看2个简单的代码例子，执行过程参见注释。

下面的代码片段来自Oracle的javase-tutorial[Language Basics->Branching Statements](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/branch.html)的[`BreakWithLabelDemo`](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/examples/BreakWithLabelDemo.java)

```java
class BreakWithLabelDemo {
    public static void main(String[] args) {//在二维数组中查找指定的数，找到则退出循环。
        int[][] arrayOfInts = { 
            { 32, 87, 3, 589 },
            { 12, 1076, 2000, 8 },
            { 622, 127, 77, 955 }
        };
        int searchfor = 12;//指定的数为12

        int i;
        int j = 0;
        boolean foundIt = false;
	//---label begin---
    search:
        for (i = 0; i < arrayOfInts.length; i++) {
            for (j = 0; j < arrayOfInts[i].length; j++) {
                if (arrayOfInts[i][j] == searchfor) {
                    foundIt = true;
                    break search;//找到12，退出search label的代码块
                }
            }
        }
     //---label end--- 

        if (foundIt) {
            System.out.println("Found " + searchfor + " at " + i + ", " + j);
        } else {
            System.out.println(searchfor + " not in the array");
        }
    }
}
```

同样地，还有1个关于`continue`的代码片段在Oracle的javase-tutorial[Language Basics->Branching Statements](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/branch.html)的[`ContinueWithLabelDemo`](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/examples/ContinueWithLabelDemo.java)

```java
public static void ContinueWithLabelDemo(){//子字符串匹配
    String searchMe = "Look for a substring in me";
    String substring = "sub";//子字符串
    boolean foundIt = false;

    int max = searchMe.length() -
        substring.length();
	//---label begin---
    test:
    for (int i = 0; i <= max; i++) {
        int n = substring.length();
        int j = i;
        int k = 0;
        while (n-- != 0) {
            if (searchMe.charAt(j++) != substring.charAt(k++)) {
                continue test;//若没有找到，跳转到外层的for循环，执行i++，继续下一次循环
            }
        }
        foundIt = true;
        break test;//匹配到了子字符串，退出test label代码块
    }
    //---label end---
    System.out.println(foundIt ? "Found it" : "Didn't find it");
}
```

- 在IDEA上，把鼠标点击`continue`后，会把外层循环的`for`，包括`continue`本身，`break`这3个关键字高亮。

### 语法规则

前面已经提到了关于`continue`和`break`的label语法。

`continue`和`break`后面接上需要跳转的label，即`continue label`/`break label`，前面代码中的`found`、`search` 、`test`称为label。而label具体指代的是大括号{}括起来的代码块。

标签label的语法可以更广泛，如下

> 事实上，可以把标签应用到任何语句中，甚至可以应用到`if`语句或者块语句中，如下所示
>
> ```java
> label:
> {
>  ...
>  if(condition) break label;//exits block
>  ...
> }
> //jumps here when the break statement executes
> 
> ```
>
> 因此，如果希望使用一条 goto 语句， 并将一个标签放在想要跳到的语句块之前， 就可以使用 break 语句！ 当然， 并不提倡使用这种方式。 另外需要注意， 只能跳出语句块，而不能跳入语句块。  
>
> -Java核心技术 卷1 基础知识（原书第10版）

但`continue`和`break`还是得遵守相应的规则，`continue`后面接的label代码块必须是循环，否则会报错。

### 作用

在tutorial中一句概述的话，把`break`的label作用概括得很清楚了，continue同理。

> An unlabeled `break` statement terminates the innermost `switch`, `for`, `while`, or `do-while` statement, but a labeled `break` terminates an outer statement. 
>
> -[Language Basics->Branching Statements](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/branch.html)

简单总结一下

- 没有label的`break`退出**当前循环**。而带label的`break`突破当前循环的限制，退出指定label**外层代码块**。

- 而没有label的`continue`退出**当前循环步骤**，执行下一次循环步骤。而带label的`continue`突破当前循环的限制，跳转到指定label**外层代码块**，执行下一次循环步骤。

## 带label的break的由来和goto的关系

其实带label的`break`这个语法很早就有，只是不常用而已，因为也仔细没有注意过，甚至有种是新语法的感觉。

参考如下

> 尽管 Java 的设计者将 goto 作为保留字， 但实际上并没有打算在语言中使用它。通常，使用 goto 语句被认为是一种拙劣的程序设计风格。 当然，也有一些程序员认为反对 goto 的呼声似乎有些过分（例如， Donald Knuth 就曾编著过一篇名为《 Structured Programming withgoto statements》的著名文章。) 这篇文章说：无限制地使用 goto 语句确实是导致错误的根源，但在有些情况下，偶尔使用 goto 跳出循环还是有益处的。Java 设计者同意这种看法，甚至在Java语言中增加了一条带标签的 break， 以此来支持这种程序设计风格。  
>
> -Java核心技术 卷1 基础知识（原书第10版）

## 总结

`continue`和`break`的label并不是新语法，而是一种对`goto`的改良，要抵制的是**滥用**`goto`，无限制地使用`goto`是导致错误的根源。同样地，滥用label也是不可取的。如果能在代码用上而且控制得好，可以增加代码的可读性。

另外label也可以用在任何语句中，还可以是`if`语句和块语句中。不过一般还是在循环中搭配`break`和`continue`使用的情况居多。



## 参考资料

1. https://stackoverflow.com/questions/6650239/java-continue-label-is-deprecated

2. https://stackoverflow.com/questions/14960419/is-using-a-labeled-break-a-good-practice-in-java

3. https://docs.oracle.com/javase/tutorial/java/nutsandbolts/branch.html

4. Java核心技术 卷1 基础知识（原书第10版）