### **MAT介绍**

MAT\(Memory Analyzer Tool\)，一个基于Eclipse的内存分析工具，是一个快速、功能丰富的JAVA heap分析工具，它可以帮助我们查找内存泄漏和减少内存消耗。使用内存分析工具从众多的对象中进行分析，快速的计算出在内存中对象的占用大小，看看是谁阻止了垃圾收集器的回收工作，并可以通过报表直观的查看到可能造成这种结果的对象。

## **MAT使用技巧进阶**

### **使用Android Studio Dump内存文件**

Android Studio的最新版本可以直接获取hprof文件：

![](http://androidperformance.com/images/MAT_Pro/MAT_1.png)



然后选择文件，点击右键转换成标准的hprof文件，就可以在MAT中打开了。

> 在使用使用Eclipse或者AndroidStudio抓内存之前，一定要手动点击 Initiate GC按钮手动触发GC，这样抓到的内存使用情况就是不包括Unreachable对象的。

![](http://androidperformance.com/images/MAT_Pro/MAT_2.png)

### **Unreachable对象**

Unreachable指的是可以被垃圾回收器回收的对象，但是由于没有GC发生，所以没有释放，这时抓的内存使用中的Unreachable就是这些对象。

![](http://androidperformance.com/images/MAT_Pro/MAT_3.png)

![](http://androidperformance.com/images/MAT_Pro/MAT_4.png)

点击Calculate Retained Size之后，会出现Retained Size这一列



![](http://androidperformance.com/images/MAT_Pro/MAT_6.png)![](http://androidperformance.com/images/MAT_Pro/MAT_5.png)

可以看到Unreachable Object的对象其Retained Heap值都为0.这也是正常的。

### **Histogram**

MAT中Histogram的主要作用是查看一个instance的数量，一般用来查看自己创建的类的实例的个数。

* 可以很容易的找出占用内存最多的几个对象，根据Percentage（百分比）来排序。
* 可以分不同维度来查看对象的Dominator Tree视图，Group by class、Group by class loader、Group by package

  和Histogram类似，时间久了，通过多次对比也可以把溢出对象找出来。

* Dominator Tree和Histogram的区别是站的角度不一样，Histogram是站在类的角度上去看，Dominator Tree是站的对象实例的角度上看，Dominator Tree可以更方便的看出其引用关系。
* ![](http://androidperformance.com/images/MAT_Pro/MAT_7.png)

通过查看Object的个数，结合代码就可以找出存在内存泄露的类（**即可达但是无用的对象，或者是可以重用但是重新创建的对象**）

Histogram中还可以对对象进行Group，更方便查看自己Package中的对象信息。

![](http://androidperformance.com/images/MAT_Pro/MAT_8.png)

### **Thread信息**

MAT中可以查看当前的Thread信息：

![](http://androidperformance.com/images/MAT_Pro/MAT_9.png)

从图中可以得到的信息：

1. 可以看到可能有内存问题的Thread：
2. ![](http://androidperformance.com/images/MAT_Pro/MAT_10.png)

1. 可以看到数量可能有问题的Thread![](http://androidperformance.com/images/MAT_Pro/MAT_11.png)

### **帮助信息**

MAT中的各个视图中，在每一个Item中点击右键会出现很多选项，很多时候我们需要依赖这些选项来进行分析：

![](http://androidperformance.com/images/MAT_Pro/MAT_12.png)

这些选项的具体含义则可以通过右键中的Search Queries这个选项\(上图中的倒数第四个选项\)进行搜索和查看，非常的有用。

![](http://androidperformance.com/images/MAT_Pro/MAT_13.png)

可以看到，所有的命令其实就是配置不同的SQL查询语句。

比如我们最常用的：

* **List objects -&gt; with incoming references**：查看这个对象持有的外部对象引用
* **List objects -&gt; with outcoming references**：查看这个对象被哪些外部对象引用
* **Path To GC Roots -&gt; exclude all phantim\/weak\/soft etc. references**：查看这个对象的GC Root，不包含虚、弱引用、软引用，剩下的就是强引用。从GC上说，除了强引用外，其他的引用在JVM需要的情况下是都可以 被GC掉的，如果一个对象始终无法被GC，就是因为强引用的存在，从而导致在GC的过程中一直得不到回收，因此就内存溢出了。
* **Path To GC Roots -&gt; exclude weak\/soft references**：查看这个对象的GC Root，不含弱引用和软引用所有的引用.
* **Merge Shortest path to GC root **：找到从GC根节点到一个对象或一组对象的共同路径

### **Debug Bitmap**

如果经常使用MAT分析内存，就会发现Bitmap所占用的内存是非常大的，这个和其实际显示面积是有关系的。在2K屏幕上，一张Bitmap能达到20MB的大小。

所以要是MAT提供了一种方法，可以将存储Bitmap的byte数组导出来，使用第三方工具打开。这个大大提高了我们分析内存泄露的效率。

关于这个方法的操作流程，可以参考这篇文章[还原MAT中的Bitmap图像](http://www.androidperformance.com/2015/04/11/AndroidMemory-Open-Bitmap-Object-In-MAT.html).

I

### **Debug ArrayList**

ArrayList是使用非常常用的一个数据结构，在MAT中，如果想知道ArrayList中有哪些数据，需要右键-&gt; List Objects -&gt; With outgoing references,然后可以看到下面的图：

![](http://androidperformance.com/images/MAT_Pro/MAT_14.png)

从上图可以看到，这个ArrayList的内容在一个array数组中，即暴漏了ArrayList的内部结构，查看的时候有点不方便，所以MAT提供了另外一种查看ArrayList内数据的方式：

![](http://androidperformance.com/images/MAT_Pro/MAT_15.png)

其结果非常直观：

![](http://androidperformance.com/images/MAT_Pro/MAT_16.png)

### **Big Drops In Dominator Tree**

Big Drops In Dominator Tree选项在右键-&gt;

> Displays memory accumulation points in the dominator tree. Displayed are objects with a big difference between the retained size of the parent and the children and the first “interesting” dominator of the accumulation point. These are places where the memory of many small objects is accumulated under one object.

