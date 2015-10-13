#How does a relational database work
by[`Christophe`](http://coding-geek.com/author/mawata/) | [sourcelink](http://coding-geek.com/how-databases-work/)    
<br>
When it comes to relational databases, I can’t help thinking that something is missing. They’re used everywhere. There are many different databases: from the small and useful SQLite to the powerful Teradata. But, there are only a few articles that explain how a database works. You can google by yourself “how does a relational database work” to see how few results there are. Moreover, those articles are short.  Now, if you look for the last trendy technologies (Big Data, NoSQL or JavaScript), you’ll find more in-depth articles explaining how they work.    
<br>
Are relational databases too old and too boring to be explained outside of university courses, research papers and books?    
<br>
<br>
<br>
![database providers](http://coding-geek.com/wp-content/uploads/2015/08/main_databases.jpg)    
<br>
<br>
As a developer, I HATE using something I don’t understand. And, if databases have been used for 40 years, there must be a reason. Over the years, I’ve spent hundreds of hours to really understand these weird black boxes I use every day. ***Relational Databases are*** very interesting because they’re ***based on useful and reusable concepts***. If understanding a database interests you but you’ve never had the time or the will to dig into this wide subject, you should like this article.
<br>
<br>
<br>
Though the title of this article is explicit, ***the aim of this article is NOT to understand how to use a database***. Therefore, ***you should already know how to write a simple join query and basic CRUD queries***; otherwise you might not understand this article. ***This is the only thing you need to know***, I’ll explain everything else.    
<br>   
I’ll start with some computer science stuff like time complexity. I know that some of you hate this concept but, without it, you can’t understand the cleverness inside a database. Since it’s a huge topic, ***I’ll focus on*** what I think is essential: ***the way a database handles an SQL query***. I’ll only present the basic concepts behind a database so that at the end of the article you’ll have ***a good idea of what’s happening under the hood***.
<br>
<br>
<br>
Since it’s a long and technical article that involves many algorithms and data structures, take your time to read it. Some concepts are more difficult to understand; you can skip them and still get the overall idea.    
<br>
For the more knowledgeable of you, this article is more or less divided into 3 parts:
* An overview of low-level and high-level database components
* An overview of the query optimization process
* An overview of the transaction and buffer pool management   

//TODO: here we should generate a content    
[TOC]
##Back to basics
##回归基础
A long time ago (in a galaxy far, far away….) developers had to know exactly the number of operations they were coding. They knew by heart their algorithms and data structures because they couldn’t afford to waste the CPU and memory of their slow computers.    
很久以前（估计有银河系诞生那么久远...），开发人员不得不精通非常多的编程操作。因为他们不能浪费他们龟速电脑上哪怕一丁点儿的CPU和内存，他们必须将这些算法和相应的数据结构深深的记在心里。
<br>
In this part, I’ll remind you about some of these concepts because they are essential to understand a database. I’ll also introduce the notion of ***database index***.    
在这个部分，我将带你们回忆一些这样的概念，因为它们对于理解数据库是非常必要的。我也将会介绍***数据库索引***这个概念。
<br>
<br>
<br>
###O(1)) vs O(n<sup>2</sup>)
Nowadays, many developers don’t care about time complexity … and they’re right!    
现在，许多开发者不再关心时间复杂度...他们是对的！
<br>
But when you deal with a large amount of data (I’m not talking about thousands) or if you’re fighting for milliseconds, it becomes critical to understand this concept. And guess what, databases have to deal with both situations! I won’t bore you a long time, just the time to get the idea. This will help us later to understand the concept of ***cost based optimization***.
但是当你们正面临着一个大数据量（我谈论的并不是几千这个级别的数据）的处理问题时或者正努力为毫秒级的性能提升拼命时，理解这个概念就非常的重要了。可是你们猜怎么着？数据库不得不处理这两种极端情况！我不会占用你们太多时间，只需要将这个点子讲清楚就够了。这将会帮助我们以后理解***成本导向最优化***的概念。
<br>
<br>
<br>
####The concept
####基本概念
The ***time complexity is used to see how long an algorithm will take for a given amount of data***. To describe this complexity, computer scientists use the mathematical big O notation. This notation is used with a function that describes how many operations an algorithm needs for a given amount of input data.    
***时间复杂度时用来衡量一个算法处理给定量的数据所消耗时间多少的***。为了描述这个复杂事物，计算机科学家们用数学上的大写字母O符号.这个符号用来描述了在方法中一个算法需要多少次操作才能处理完给定的输入数据量。
<br>
For example, when I say “this algorithm is in O( some_function() )”, it means that for a certain amount of data the algorithm needs some_function(a_certain_amount_of_data) operations to do its job.    
例如，当我说”这个算法是在O(some_funtion())“时，这意味着这个算法为了处理确定量的数据需要执行some_function(a_certain_amount_of_data)操作.
<br>
***What’s important*** is not the amount of data but ***the way the number of operations increases when the amount of data increases***. The time complexity doesn’t give the exact number of operations but a good idea.    
***最重要的***不是数据量，而是***随着数据量的增加，操作步骤需要随之变化的方式***。时间复杂度不是给出确切的操作数量而是一个概念。
<br>
![time costs](http://coding-geek.com/wp-content/uploads/2015/08/TimeComplexity.png)    
<br>
In this figure, you can see the evolution of different types of complexities. I used a logarithmic scale to plot it.     
在上图中，你可以看到不同类型的复杂度演变的方式。我用了对数尺度来描绘。
In other words, the number of data is quickly increasing from 1 to 1 billion. We can see that:
<br>
* The <font color="#00ff00" >O(1)</font> or constant complexity stays constant (otherwise it wouldn’t be called constant complexity).
* The <font color="#ff0000" >O(log(n))</font> ***stays low even with billions of data***.
* The worst complexity is the <font color="#ff00ff">O(n<sup>2</sup>)</font> ***where the number of operations quickly explodes***.
* The two other complexities are quickly increasing.

换句话讲，当数据的量从1到10亿，我们可以看到：
* <font color="#00ff00" >O(1)</font>即常数复杂度保持常数操作数（不然它就不叫常数复杂度了）。
* <font color="#ff0000" >O(log(n))</font> ***即使是上亿级的数据仍保持较低操作数***。
* 最差的复杂度是<font color="#ff00ff">O(n<sup>2</sup>)</font> ***,它的操作数是爆炸式增长***。
* 另外两种复杂度增长快速。

####Examples
####举例
With a low amount of data, the difference between O(1) and O(n<sup>2</sup>) is negligible. For example, let’s say you have an algorithm that needs to process 2000 elements.
<br>
* An O(1) algorithm will cost you 1 operation
* An O(log(n)) algorithm will cost you 7 operations
* An O(n) algorithm will cost you 2 000 operations
* An O(n*log(n)) algorithm will cost you 14 000 operations
* An O(n<sup>2</sup>) algorithm will cost you 4 000 000 operations    
<br>

当小数据量时，O(1)与O(n<sup>2</sup>)之间的差距是微乎其微的。例如，假设你需要处理2000条数据的算法。
<br>
* O(1)算法需要1次操作
* O(log(n))算法需要7次操作
* O(n)算法需要2000次操作
* O(n*log(n))算法需要14000次操作
* O(n<sup>2</sup>)算法需要4000000次操作
<br>

The difference between O(1) and O(n<sup>2</sup>) seems a lot (4 million) but you’ll lose at max 2 ms, just the time to blink your eyes. Indeed, current processors can handle [`hundreds of millions of operations per second`](https://en.wikipedia.org/wiki/Instructions_per_second). This is why performance and optimization are not an issue in many IT projects.    
O(1)与O(n<sup>2</sup>)之间的区别似乎非常大（4百万倍），但是你实际上最多多消耗2毫秒，和你眨眼的时间几乎相同。的确，现在的处理器能处理[`每秒数以百万计指令`](https://en.wikipedia.org/wiki/Instructions_per_second)。这就是为什么在许多IT工程中性能和优化并不是主要问题的原因。
<br>
<br>
<br>
As I said, it’s still important to know this concept when facing a huge number of data. If this time the algorithm needs to process 1 000 000 elements (which is not that big for a database):
<br>
* An O(1) algorithm will cost you 1 operation
* An O(log(n)) algorithm will cost you 14 operations
* An O(n) algorithm will cost you 1 000 000 operations
* An O(n*log(n)) algorithm will cost you 14 000 000 operations
* An O(n<sup>2</sup>) algorithm will cost you 1 000 000 000 000 operations    
<br>    

正如我所说，当面对海量数据时，了解这个概念还是非常重要的。如果这时算法需要处理1000000条数据（对于数据库来说，这还不算大）：
* O(1)算法需要1次操作
* O(log(n))算法需要14次操作
* O(n)算法需要1000000次操作
* O(n*log(n))算法需要14000000次操作
* O(n<sup>2</sup>)算法需要1000000000000次操作

I didn’t do the math but I’d say with the O(n<sup>2</sup>) algorithm you have the time to take a coffee (even a second one!). If you put another 0 on the amount of data, you’ll have the time to take a long nap.    
我没有详细算过，但是我想如果采用O(n<sup>2</sup>)算法，你可以有时间来杯咖啡了（甚至再来一杯！）。如果你又将数据数量级提升一个0，你可以有时间去打个盹儿了。
<br>
<br>
<br>
####Going deeper
####继续深入
To give you an idea:

* A search in a good hash table gives an element in O(1)
* A search in a well-balanced tree gives a result in O(log(n))
* A search in an array gives a result in O(n)
* The best sorting algorithms have an O(n*log(n)) complexity.
* A bad sorting algorithm have an O(n<sup>2</sup>) complexity   

给你一个概念：
* 从一个哈希表中进行元素查找操作的复杂度是O(1)
* 从一个平衡树中进行查找操作的复杂度时O(log(n))
* 从数组中进行一次查找操作的复杂度O(n)
* 最优的排序算法的复杂度是O(n*log(n))。
* 差的排序算法的复杂度是O(n<sup>2</sup>)    

Note: In the next parts, we’ll see these algorithms and data structures.    
注意：在之后的内容中，我们将会看到这些算法和数据结构。
<br>
<br>
<br>
There are multiple types of time complexity:
<br>
* the average case scenario
* the best case scenario
* and the worst case scenario    
<br>

存在着多种种类的时间复杂度：
* 平均情况
* 最优情况
* 以及最差情况

The time complexity is often the worst case scenario.    
时间复杂度经常是最差情况。
<br>
I only talked about time complexity but complexity also works for:
<br>
* the memory consumption of an algorithm.
* the disk I/O consumption of an algorithm.    
<br>
<br>
<br>

我仅讨论时间复杂度，实际上复杂度还适用于：
* 算法的内存消耗
* 算法的磁盘I/O消耗

Of course there are worse complexities than n<sup>2</sup>, like:
<br>
* n<sup>4</sup>: that sucks! Some of the algorithms I’ll mention have this complexity.
* 3<sup>n</sup>: that sucks even more! One of the algorithms we’re going to see in the middle of this article has this complexity (and it’s really used in many databases).
* factorial n : you’ll never get your results, even with a low amount of data.
* n<sup>n</sup>: if you end-up with this complexity, you should ask yourself if IT is really your field…    
<br>
<br>

当然也有比n<sup>2</sup>还差的复杂度情况，例如：
* n<sup>4</sup>：糟糕透了！我将会提到一些如此复杂度的算法。
* 3<sup>n</sup>：不能再糟了! 我们在本文中间部分将会看到这样复杂度的一个算法(而且在许多数据中，它确实在被使用着)。
* n的阶乘 : 即使是很小数量级的数据，你也将永远得不到你想要的结果。
* n<sup>n</sup>: 如果你最终的结果是这个算法复杂度，你应该好好问问自己到底是不是做IT的…    

Note: I didn’t give you the real definition of the big O notation but just the idea. You can read this article on [`Wikipedia`](https://en.wikipedia.org/wiki/Big_O_notation) for the real definition.    
注意：我并没有给你O符号的真正定义，而只是抛出这个概念。如果你想找到真正的定义，你可以阅读这篇[`WikiPedia材料`](https://en.wikipedia.org/wiki/Big_O_notation)。
<br>
<br>
<br>
###Merge Sort
###归并排序

What do you do when you need to sort a collection? What? You call the sort() function …  ok, good answer… But for a database you have to understand how this sort() function works.    
如果你需要排序一个集合，你会怎么做？什么？你会调用sort()函数... 好吧，真是个好答案...但是对于数据库来说，你必须懂得sort()函数是如何工作的。

There are several good sorting algorithms so I’ll focus on the most important one: ***the merge sort***. You might not understand right now why sorting data is useful but you should after the part on query optimization. Moreover, understanding the merge sort will help us later to understand a common database join operation called the ***merge join***.    
因为有太多好的排序算法，所以我将专注于最重要的一个：***归并排序***。此时此刻你可能不是很明白为什么数据排序会有用，但是当完成这个部分的查询优化后，你肯定会懂得。进一步来说，掌握归并排序将有助于后续我们对一般数据库中合并连接操作的理解。

 

####Merge
####合并

Like many useful algorithms, the merge sort is based on a trick: merging 2 sorted arrays of size N/2 into a N-element sorted array only costs N operations. This operation is called a merge.    
如同许多有用的算法，归并排序是基础的技巧：合并2个长度为N/2的有序数组为一个有N个元素的有序数组仅消耗N次操作。这个操作称为一次合并。

Let’s see what this means with a simple example:    
让我们通过一个简单例子来看看其含义：
![Merge](http://coding-geek.com/wp-content/uploads/2015/08/merge_sort_3.png)
You can see on this figure that to construct the final sorted array of 8 elements, you only need to iterate one time in the 2 4-element arrays. Since both 4-element arrays are already sorted:

* 1) you compare both current elements in the 2 arrays (current=first for the first time)
* 2) then take the lowest one to put it in the 8-element array
* 3) and go to the next element in the array you took the lowest element
* and repeat 1,2,3 until you reach the last element of one of the arrays.
* Then you take the rest of the elements of the other array to put them in the 8-element array.    
<br>

你能从图中看到最终排序好8个元素的数组的结构，你仅需要重复访问一次2个4元素数组。因为这2个4元素数组已经排序好了：
* 1) 你需要比较两个数组当前的元素（第一次的时候current=first）
* 2) 接下来将最小的那个放进8元素数组中
* 3) 将你提取最小元素的那个数组指向下一个元素
* 重复1，2，3步骤，直到你到达任何一个数组的最后一个元素.
* 接下来，你需要将另外一个数组的剩余元素放进8元素数组中。



This works because both 4-element arrays are sorted and therefore you don’t need to “go back” in these arrays.   
这个算法之所以生效是因为4元素数组都是已经排序好的，因此你不必在这些数组中进行"回退"。



Now that we’ve understood this trick, here is my pseudocode of the merge sort.    
现在我们懂得了这个技巧，如下所示是我的合并排序伪代码。
```
array mergeSort(array a)
   if(length(a)==1)
      return a[0];
   end if
 
   //recursive calls
   [left_array right_array] := split_into_2_equally_sized_arrays(a);
   array new_left_array := mergeSort(left_array);
   array new_right_array := mergeSort(right_array);
 
   //merging the 2 small ordered arrays into a big one
   array result := merge(new_left_array,new_right_array);
   return result;
```
The merge sort breaks the problem into smaller problems then finds the results of the smaller problems to get the result of the initial problem (note: this kind of algorithms is called divide and conquer). If you don’t understand this algorithm, don’t worry; I didn’t understand it the first time I saw it. If it can help you, I see this algorithm as a two-phase algorithm:    
归并排序将问题拆分为更小的问题，再求解这些小问题结果，从而获得最初的问题结果（注意：这类算法叫做分治法）。如果你不懂这个算法，不用担心；我最初看这个算法时也是不懂。我将这个算法看作两个阶段算法，希望对你们有所帮助：

* The division phase where the array is divided into smaller arrays
* The sorting phase where the small arrays are put together (using the merge) to form a bigger array.    


* 将一个数组才分为更小的数组称为分解阶段    
* 将小的数组组合在一起（使用合并）组成更大的数组称为排序阶段。
 

####Division phase    
####分解阶段
![Division phase](http://coding-geek.com/wp-content/uploads/2015/08/merge_sort_1.png)    
During the division phase, the array is divided into unitary arrays using 3 steps. The formal number of steps is log(N)  (since N=8, log(N) = 3).    
在分解阶段中，数组被拆分为单一的数组用了3步。步骤数量的表达式为log(N)(由于 N=8，log(N) ＝ 3)。

How do I know that?    
我是怎么知道的呢？

~~I’m a genius~~ In one word: mathematics. The idea is that each step divides the size of the initial array by 2. The number of steps is the number of times you can divide the initial array by two. This is the exact definition of logarithm (in base 2).    
~~我是个天才！~~总之：数学。每一步的核心是将最初的数组长度对半拆分。步骤数量就是你能二分原始数组的次数。这就是对数的定义（以2为底）。

 

####Sorting phase    
####排序阶段
![Sorting phase](http://coding-geek.com/wp-content/uploads/2015/08/merge_sort_2.png)    
In the sorting phase, you start with the unitary arrays. During each step, you apply multiple merges and the overall cost is N=8 operations:    
<br>
* In the first step you have 4 merges that cost 2 operations each
* In the second step you have 2 merges that cost 4 operations each
* In the third step you have 1 merge that costs 8 operations    

在排序阶段中，你将从单一数组开始。在每一步中，你使用多重汇集。总共需要N=8次操作：    
* 第一步，你将做4次合并，每次需要2步操作
* 第二步，你将进行2次合并，每次需要4步操作
* 第三步，你将做1次合并，每次需要8步操作

Since there are log(N) steps, ***the overall costs N * log(N) operations***.    
因为总共有log(N)个步骤，***总共需要N * log(N)次操作***。

 

####The power of the merge sort     
####归并排序的威力    

Why this algorithm is so powerful?    
为什么这个算法有如此威力？    


Because:    
缘由如下：    

* You can modify it in order to reduce the memory footprint, in a way that you don’t create new arrays but you directly modify the input array.
Note: this kind of algorithms is called [`in-place`](https://en.wikipedia.org/wiki/In-place_algorithm).

* You can modify it in order to use disk space and a small amount of memory at the same time without a huge disk I/O penalty. The idea is to load in memory only the parts that are currently processed. This is important when you need to sort a multi-gigabyte table with only a memory buffer of 100 megabytes.
Note: this kind of algorithms is called [`external sorting`](https://en.wikipedia.org/wiki/External_sorting).

* You can modify it to run on multiple processes/threads/servers.
For example, the distributed merge sort is one of the key components of [`Hadoop`](https://hadoop.apache.org/docs/stable/api/org/apache/hadoop/mapreduce/Reducer.html) (which is THE framework in Big Data).

* This algorithm can turn lead into gold (true fact!).

* 你可以将它改造为低内存占用型，通过不再建立一个新的数组而是直接修改输入数组。
注意：这种算法称为[原地算法](https://en.wikipedia.org/wiki/In-place_algorithm)
* 你可以将它改造为使用磁盘空间和更小的内存占用的同时，避免大量的磁盘I/O消耗。这个算法的理念是每次只加载当前处理的部分数据进入内存。当你只有100M内存空间却需要对数G大小的表进行排序时，这个算法将十分重要。
注意：这种算法称为[外部排序](https://en.wikipedia.org/wiki/External_sorting)。
* 你可以将这个算法改造为运行在多处理器/线程/服务器。
例如，分布式归并排序就是[`Hadoop`](https://hadoop.apache.org/docs/stable/api/org/apache/hadoop/mapreduce/Reducer.html)的核心模块（一种大数据框架）。
* 这个算法能点石成金（真的！）。   


This sorting algorithm is used in most (if not all) databases but it’s not the only one. If you want to know more, you can read this [`research paper`](http://wwwlgis.informatik.uni-kl.de/archiv/wwwdvs.informatik.uni-kl.de/courses/DBSREAL/SS2005/Vorlesungsunterlagen/Implementing_Sorting.pdf) that discusses the pros and cons of the common sorting algorithms in a database.    
这个排序算法应用于大多数（好吧，如果不是全部）数据库，但是它不是唯一的。如果你想了解更多，你可以阅读这个[研究材料](http://wwwlgis.informatik.uni-kl.de/archiv/wwwdvs.informatik.uni-kl.de/courses/DBSREAL/SS2005/Vorlesungsunterlagen/Implementing_Sorting.pdf)，这里面讨论了数据库中所使用到的通用排序算法的优缺点。
 

###Array, Tree and Hash table
###数组，树以及哈希表

Now that we understand the idea behind time complexity and sorting, I have to tell you about 3 data structures. It’s important because they’re ***the backbone of modern databases***. I’ll also introduce the notion of ***database index***.    
我们已经了解时间复杂度和排序背后的机理，我必须给你讲3种数据结构。它们十分重要，因为它们是***现代数据库的支柱***。同时我也会介绍***数据库索引***。



####Array

The two-dimensional array is the simplest data structure. A table can be seen as an array. For example:    
二维数组是最简单的数据结构。表格也能看成是一个数组。如下：
![Array](http://coding-geek.com/wp-content/uploads/2015/08/array.png)    
This 2-dimensional array is a table with rows and columns:
<br>
* Each row represents a subject
* The columns the features that describe the subjects.
* Each column stores a certain type of data (integer, string, date …).    

二维数组就是一个行列表：
* 每行表示一个对象
* 列表示描述对象的特性
* 每列存储一种特定类型的数据(整型，字符串，日期 ...)。    


Though it’s great to store and visualize data, when you need to look for a specific value it sucks.    
尽管这样存储和数据可视化都非常好，但是当你面对特殊数据时，这个就很糟了。

For example, ***if you want to find all the guys*** who work in the UK, you’ll have to look at each row to find if the row belongs to the UK. ***This will cost you N operations*** (N being the number of rows) which is not bad but could there be a faster way? This is where trees come into play.    
例如，***如果你想找到所有工作在英国的人***，你将不得不查看每一行看这一行是否属于英国。***这将消耗你N步操作***（N是行数），这并不算太坏，但是否又有更好的方式呢？这就是为什么要引入tree。 


 

Note: Most modern databases provide advanced arrays to store tables efficiently like heap-organized tables or index-organized tables. But it doesn’t change the problem of fast searching for a specific condition on a group of columns.     
注意：大多数现代数据库提供了增强型数组来高效的存储表单，例如：堆组织表或索引组织表。但是他并没有解决特殊值在列集合中的快速查找问题。


####Tree and database index
####树和数据库索引

A binary search tree is a binary tree with a special property, the key in each node must be:    
<br>
* greater than all keys stored in the left sub-tree
* smaller than all keys stored in the right sub-tree    


二叉搜索树时一种带有特殊属性的二叉树，每个节点的键值必须满足：
<br>
* 大于所有左子树的键值
* 小于所有右子树的键值
 

Let’s see what it means visually    
让我们直观的看看上面的含义

**The idea**    
**概念**
![Binary Search Tree](http://coding-geek.com/wp-content/uploads/2015/08/BST.png)
 

This tree has N=15 elements. Let’s say I’m looking for 208:
<br>
* I start with the root whose key is 136. Since 136<208, I look at the right sub-tree of the node 136.
* 398>208 so, I look at the left sub-tree of the node 398
* 250>208 so, I look at the left sub-tree of the node 250
* 200<208 so, I look at the right sub-tree of the node 200. But 200 doesn’t have a right subtree, ***the value doesn’t exist*** (because if it did exist it would be in the right subtree of 200)    

这棵树有 N=15 个节点构成。假设我们搜索208：
<br>
* 我将从根节点的键值136开始，因为136<208，所以我们查找136节点的右子树。
* 398>208 所以，我们查找398节点的左子树
* 250>208 所以，我们查找250节点的左子树
* 200<208 所以，我们查找200节点的右子树。但是200节点没有右子树，***这个值不存在***（如果它存在，那么它必定在200节点的右子树中)

Now let’s say I’m looking for 40    
<br>
* I start with the root whose key is 136. Since 136>40, I look at the left sub-tree of the node 136.
* 80>40 so, I look at the left sub-tree of the node 80
* 40= 40, ***the node exists***. I extract the id of the row inside the node (it’s not in the figure) and look at the table for the given row id.
* Knowing the row id let me know where the data is precisely on the table and therefore I can get it instantly.    

接下来假设我们查找40    
* 我将从根节点的键值136开始，因为136>40，所以我们查找136节点的左子树。
* 80>40 所以，我们查找80节点的右子树
* 40=40，***节点存在***。提取出节点的行号(这个没在图上)，然后根据行号查询数据表。
* 获得了行号，我们就可以知道数据在表上的精确位置，因此我们就能立即获取到数据。


In the end, both searches cost me the number of levels inside the tree. If you read carefully the part on the merge sort you should see that there are log(N) levels. So the ***cost of the search is log(N)***, not bad!    
最后，这两个查询都消耗树的层数次操作。如果你仔细地阅读了归并排序部分，那么就应该知道这里是log(N)层级。所以知道***搜索算法的时间复杂度是log(N)***，不错！

 

***Back to our problem***    
***回到我们的问题上***

But this stuff is very abstract so let’s go back to our problem. Instead of a stupid integer, imagine the string that represents the country of someone in the previous table. Suppose you have a tree that contains the column “country” of the table:
<br>
* If you want to know who is working in the UK
* you look at the tree to get the node that represents the UK
* inside the “UK node” you’ll find the locations of the rows of the UK workers.

但是这些东西还是比较抽象，我们回到我们具体的问题中。取代了前一张表中呆滞的整型，假想用字符串来表示某人的国籍。假设你又一棵包含了表格中“国籍”列的树：
<br>
* 如果你想知道谁在英国工作
* 你查找这棵树来获取代表英国的节点
* 在“英国节点”中，你将找到英国工人的行地址。    

This search only costs you log(N) operations instead of N operations if you directly use the array. What you’ve just imagined was a ***database index***.    
这个查找仅需要log(N)次操作而不是像使用数组一样需要N次操作。你们刚才猜想的就是***数据库索引***。


You can build a tree index for any group of columns (a string, an integer, 2 strings, an integer and a string, a date …) as long as you have a function to compare the keys (i.e. the group of columns) so that you can establish an ***order among the keys*** (which is the case for any basic types in a database).    
只要你有比较键值（例如 列组）的方法来建立***键值顺序***（这对于数据库中的任何一个基本类型都很重要），你可以建立任意列组的树形索引（字符串，整型，2个字符串，一个整型和一个字符串，日期...）。

 

####B+Tree Index
####B+树索引

Although this tree works well to get a specific value, there is a BIG problem when you need to ***get multiple elements between two values***. It will cost O(N) because you’ll have to look at each node in the tree and check if it’s between these 2 values (for example, with an in-order traversal of the tree). Moreover this operation is not disk I/O friendly since you’ll have to read the full tree. We need to find a way to efficiently do a ***range query***. To answer this problem, modern databases use a modified version of the previous tree called B+Tree. In a B+Tree:
<br>
* only the lowest nodes (the leaves) store information (the location of the rows in the associated table)
* the other nodes are just here to route to the right node during the search.    

尽管树形对于获取特殊值表现良好，但是当你需要***获取在两个值范围之间的多条数据***时还是存在着一个大问题。因为你必须查询树种的每个节点看其是否在两值范围之间（例如，顺序遍历整棵树）。更糟的是这种操作很是占用磁盘I/O，因为你将不得不读取整棵树。我们需要找到一种有效的方式来做***范围查询***。为了解决这个问题，现代数据库用了一个之前树形结构的变形，叫做B+树。在B+树中：
<br>
* 仅只有最底层的节点(叶子节点)存储信息（相关表的行坐标）
* 其他节点仅在搜索过程中起导向到对应节点的作用。    

![B+ Tree](http://coding-geek.com/wp-content/uploads/2015/08/database_index.png)
As you can see, there are more nodes (twice more). Indeed, you have additional nodes, the “decision nodes” that will help you to find the right node (that stores the location of the rows in the associated table). But the search complexity is still in O(log(N)) (there is just one more level). The big difference is that ***the lowest nodes are linked to their successors***.    

如你所见，这将引入更多的节点（两倍多）。的确，你需要更多额外的节点，这些“决策节点”来帮助你找到目标节点（存储了相关表的行坐标信息的节点）。但是搜索的复杂度仍然是O(log(N))（仅仅是多了一层）。最大的区别在于***最底层的节点指向了目标***。   

With this B+Tree, if you’re looking for values between 40 and 100:

* You just have to look for 40 (or the closest value after 40 if 40 doesn’t exist) like you did with the previous tree.
* Then gather the successors of 40 using the direct links to the successors until you reach 100.    


Let’s say you found M successors and the tree has N nodes. The search for a specific node costs log(N) like the previous tree. But, once you have this node, you get the M successors in M operations with the links to their successors. ***This search only costs M + log(N)*** operations vs N operations with the previous tree. Moreover, you don’t need to read the full tree (just M + log(N) nodes), which means less disk usage. If M is low (like 200 rows) and N large (1 000 000 rows) it makes a BIG difference.    

假设你使用B+树来搜索40到100之间的值：
<br>
* 你必须像上一树形结构中一样查找40的值（或者如果40不存在，就找最接近40的值）。
* 40的结果集合直接链接了直到100的结果。

假设你找到了M个结果，并且整棵树有N个节点。如同上一树形结构一样查找特殊节点需要log(N)步操作。但是一旦你找到这个节点，你就可以通过指向他们结果集的M步操作来获取M个结果。***这样的查找方式仅需要M + log(N)步操作***相对于上一棵树形结构中的N步操作。更好的是你不必讲整棵树读取进来（只需要读取M + log(N) 个节点），这意味着更少的磁盘消耗。如果M足够小（比如200行）而N足够大（1 000 000行），这会产生巨大的差别。


 

But there are new problems (again!). If you add or remove a row in a database (and therefore in the associated B+Tree index):

* you have to keep the order between nodes inside the B+Tree otherwise you won’t be able to find nodes inside the mess.
* you have to keep the lowest possible number of levels in the B+Tree otherwise the time complexity in O(log(N)) will become O(N).    

但是又会产生新的问题（又来了！）。如果你在数据库中增加或删除一行（与此同时在相关的B+树索引中也要进行相应操作）：
* 你必须保证B+树中的节点顺序，否则你不能在混乱中找到目标节点。
* 你必须保证B+树中尽可能最小的层数，否则时间复杂度会从O(log(N))变为O(N)。

I other words, the B+Tree needs to be self-ordered and self-balanced. Thankfully, this is possible with smart deletion and insertion operations. But this comes with a cost: the insertion and deletion in a B+Tree are in O(log(N)). This is why some of you have heard that ***using too many indexes is not a good idea***. Indeed, ***you’re slowing down the fast insertion/update/deletion of a row*** in a table since the database needs to update the indexes of the table with a costly O(log(N)) operation per index. Moreover, adding indexes means more workload for the ***transaction manager*** (we will see this manager at the end of the article).

换句话说，B+树需要是自生顺序的和自平衡的。幸好使用智能删除和插入操作，这些都是可行的。但是这就引入了一个消耗：在一个B+树中的插入操作和删除操作的复杂度都是O(log(N))。这就是为什么你们有些人听说的***使用太多的索引并不是一个好办法***的原因。确实，***你降低了在表中行的快速插入/更新/删除***，以为数据库要为每个索引更新数据表的索引集都需要消耗O(log(N))次操作。更糟的是，添加索引意味着***事务管理***(我们将在本文最后看到这个管理)更多的工作量。

For more details, you can look at the Wikipedia [`article about B+Tree`](https://en.wikipedia.org/wiki/B%2B_tree). If you want an example of a B+Tree implementation in a database, look at [`this article`](http://blog.jcole.us/2013/01/07/the-physical-structure-of-innodb-index-pages/) and [`this article`](http://blog.jcole.us/2013/01/10/btree-index-structures-in-innodb/) from a core developer of MySQL. They both focus on how innoDB (the engine of MySQL) handles indexes.    
更多详情，可以查看维基百科[`B+树资料`](https://en.wikipedia.org/wiki/B%2B_tree)。如果你想知道数据库中B+树的实现细节，请查看来自MySQL核心开发者的[`博文`](http://blog.jcole.us/2013/01/07/the-physical-structure-of-innodb-index-pages/)和[`博文`](http://blog.jcole.us/2013/01/10/btree-index-structures-in-innodb/)。这两个材料都是聚焦于innoDB(MySQL数据库引擎)如何处理索引。    

Note: I was told by a reader that, because of low-level optimizations, the B+Tree needs to be fully balanced.    
注意：因为我被一个读者告知，由于低级优化，B+树需要完全平衡。

 

####Hash table
####哈希表
Our last important data structure is the hash table. It’s very useful when you want to quickly look for values.  Moreover, understanding the hash table will help us later to understand a common database join operation called ***the hash join***. This data structure is also used by a database to store some internal stuff (like the ***lock table*** or the ***buffer pool***, we’ll see both concepts later)    
我们最后一个重要的数据结构是哈希表。当你想快速查找值的时候这会非常有用。更好的是了解哈希表有助于掌握数据库通用连接操作中的***哈希连接***。这个数据结构也用于数据库存储一些中间量（如我们稍后会提到的***锁表***或***缓冲池***概念）。

The hash table is a data structure that quickly finds an element with its key. To build a hash table you need to define:
<br>
* ***a key*** for your elements
* ***a hash function*** for the keys. The computed hashes of the keys give the locations of the elements (called buckets).
* ***a function to compare the keys***. Once you found the right bucket you have to find the element you’re looking for inside the bucket using this comparison.    

哈希表是一种利用其键值快速查找元素的数据结构。为了建立哈希表，你需要定义：
<br>
* 为元素建立的***键***
* 为键建立的***哈希方法***。为键算出的哈希值可以定位元素（称为哈希桶）。
* ***键之间的比较方法***。一旦你找到了目标桶，你必须使用这个比较方法来查找桶内的元素。
 

***A simple example***    
***一个简单例子***

Let’s have a visual example:    
让我们来看看图形示例：
![Hash Map](http://coding-geek.com/wp-content/uploads/2015/08/hash_table.png)
This hash table has 10 buckets. Since I’m lazy I only drew 5 buckets but I know you’re smart so I let you imagine the 5 others. The Hash function I used is the modulo 10 of the key. In other words I only keep the last digit of the key of an element to find its bucket:

* if the last digit is 0 the element ends up in the bucket 0,
* if the last digit is 1 the element ends up in the bucket 1,
* if the last digit is 2 the element ends up in the bucket 2,
* …    

这个哈希表有10个哈希桶。换句话说，我只使用元素的最后一个数字来查找它的哈希桶：
* 如果元素的最后一个数字是0，那么就存在于0号哈希桶中，
* 如果元素的最后一个数字是1，那么就存在于1号哈希桶中，
* 如果元素的最后一个数字是2，那么就存在于2号哈希桶中，
* ...


The compare function I used is simply the equality between 2 integers.    
我所使用的比较方法仅仅是简单的比较2个整型是否相等。    
Let’s say you want to get the element 78:
<br>
* The hash table computes the hash code for 78 which is 8.
* It looks in the bucket 8, and the first element it finds is 78.
* It gives you back the element 78
* ***The search only costs 2 operations*** (1 for computing the hash value and the other for finding the element inside the bucket).
<br>

假设你想查找一个78的元素：
<br>
* 哈希表首先计算出78的哈希值是8。
* 接下来它查看8号哈希桶，并找到第一个元素就是78。
* 它就返回给你78的元素
* ***这次查找仅需要2步操作***（1步是计算哈希值而另一步则是查找哈希桶里面的元素）。

Now, let’s say you want to get the element 59:
<br>
* The hash table computes the hash code for 59 which is 9.
* It looks in the bucket 9, and the first element it finds is 99. Since 99!=59, element 99 is not the right element.
* Using the same logic, it looks at the second element (9), the third (79), … , and the last (29).
* The element doesn’t exist.
* ***The search costs 7 operations***.
 
接下来，假设你想找到59的元素：
* 哈希表首先计算59的哈希值为9。
* 它查找9号哈希桶，第一个找到的元素是99。因为99!=59，99元素不是目标元素。
* 使用相同的逻辑，它找到第二个元素(9)，第三个(79)，...，直到最后一个(29)。
* 该元素不存在。

***A good hash function***    
***优秀的哈希方法***

As you can see, depending on the value you’re looking for, the cost is not the same!    
如你所见，根据你查找的值不同，消耗也是不同的！    


If I now change the hash function with the modulo 1 000 000 of the key (i.e. taking the last 6 digits), the second search only costs 1 operation because there are no elements in the bucket 000059. ***The real challenge is to find a good hash function that will create buckets that contain a very small amount of elements***.    
如果现在我改用键值除以1 000 000的哈希方法（也就是取最后6位数字），第二个查找方法仅需要1步操作，因为不存在000059号的哈希桶。***真正的挑战是找到一个创建能容纳足够小元素的哈希桶的哈希方法***。


In my example, finding a good hash function is easy. But this is a simple example, finding a good hash function is more difficult when the key is:

* a string (for example the last name of a person)
* 2 strings (for example the last name and the first name of a person)
* 2 strings and a date (for example the last name, the first name and the birth date of a person)
* …

在我的例子中，找到一个好的哈希方法是非常容易的。但是由于这是个简单的例子，当面对如下键时，找到哈希方法就非常困难了：
* 字符串(例如人的姓氏)
* 2个字符串(例如人的姓氏和名字)
* 2个字符串和一个日期(例如人的姓氏，名字以及生日)
* ...    


***With a good hash function, the search in a hash table is in O(1)***.    
***如果有一好的哈希方法，在哈希表中查找的复杂度将是O(1)***。

 

***Array vs hash table***    
***数组与哈希表的比较***

Why not using an array?    
为什么不使用数组?    
Hum, you’re asking a good question.    
嗯, 你问了一个好问题。    

* A hash table can be ***half loaded in memory*** and the other buckets can stay on disk.
* With an array you have to use a contiguous space in memory. If you’re loading a large table it’s ***very difficult to have enough contiguous space***.
* With a hash table you can ***choose the key you want*** (for example the country AND the last name of a person).    

* 哈希表能***一半在内存中加载***，而其余的哈希桶保存在磁盘上。
* 如果使用数组，你必须使用内存中的连续内存。如果你正在加载一张较大的表，系统是***很难分配出足够大的连续空间的***。
* 如果使用哈希表，你可以***任意选择你想要的键***(例如：国家 AND 姓氏)。 

For more information, you can read my article on the [`Java HashMap`](http://coding-geek.com/how-does-a-hashmap-work-in-java/) which is an efficient hash table implementation; you don’t need to understand Java to understand the concepts inside this article.   
想要更多的信息，你可以阅读我的博文一个高效哈希表的实现[`Java HashMap`](http://coding-geek.com/how-does-a-hashmap-work-in-java/)；你可以读懂这个文章内容而不必掌握Java。

 

##Global overview
We’ve just seen the basic components inside a database. We now need to step back to see the big picture.

A database is a collection of information that can easily be accessed and modified. But a simple bunch of files could do the same. In fact, the simplest databases like SQLite are nothing more than a bunch of files. But SQLite is a well-crafted bunch of files because it allows you to:

use transactions that ensure data are safe and coherent
quickly process data even when you’re dealing with millions of data
 

More generally, a database can be seen as the following figure:
![Global Overview](http://coding-geek.com/wp-content/uploads/2015/08/global_overview.png)
Before writing this part, I’ve read multiple books/papers and every source had its on way to represent a database. So, don’t focus too much on how I organized this database or how I named the processes because I made some choices to fit the plan of this article. What matters are the different components; the overall idea is that a database is divided into multiple components that interact with each other.

The core components:

The process manager: Many databases have a pool of processes/threads that needs to be managed. Moreover, in order to gain nanoseconds, some modern databases use their own threads instead of the Operating System threads.
The network manager: Network I/O is a big issue, especially for distributed databases. That’s why some databases have their own manager.
File system manager: Disk I/O is the first bottleneck of a database. Having a manager that will perfectly handle the Operating System file system or even replace it is important.
The memory manager: To avoid the disk I/O penalty a large quantity of ram is required. But if you handle a large amount of memory, you need an efficient memory manager. Especially when you have many queries using memory at the same time.
Security Manager: for managing the authentication and the authorizations of the users
Client manager: for managing the client connections
…
The tools:

Backup manager: for saving and restoring a database.
Recovery manager: for restarting the database in a coherent state after a crash
Monitor manager: for logging the activity of the database and providing tools to monitor a database
Administration manager: for storing metadata (like the names and the structures of the tables) and providing tools to manage databases, schemas, tablespaces, …
…
The query Manager:

Query parser: to check if a query is valid
Query rewriter: to pre-optimize a query
Query optimizer: to optimize a query
Query executor: to compile and execute a query
The data manager:

Transaction manager: to handle transactions
Cache manager: to put data in memory before using them and put data in memory before writing them on disk
Data access manager: to access data on disk
 

For the rest of this article, I’ll focus on how a database manages an SQL query through the following processes:

the client manager
the query manager
the data manager (I’ll also include the recovery manager in this part)
 

Client manager
![Client manager](http://coding-geek.com/wp-content/uploads/2015/08/client_manager.png)
The client manager is the part that handles the communications with the client. The client can be a (web) server or an end-user/end-application. The client manager provides different ways to access the database through a set of well-known APIs: JDBC, ODBC, OLE-DB …

It can also provide proprietary database access APIs.

 

When you connect to a database:

The manager first checks your authentication (your login and password) and then checks if you have the authorizations to use the database. These access rights are set by your DBA.
Then, it checks if there is a process (or a thread) available to manage your query.
It also checks if the database if not under heavy load.
It can wait a moment to get the required resources. If this wait reaches a timeout, it closes the connection and gives a readable error message.
Then it sends your query to the query manager and your query is processed
Since the query processing is not an “all or nothing” thing, as soon as it gets data from the query manager, it stores the partial results in a buffer and start sending them to you.
In case of problem, it stops the connection, gives you a readable explanation and releases the resources.
 

Query manager
![Query manager](http://coding-geek.com/wp-content/uploads/2015/08/query_manager.png)
This part is where the power of a database lies. During this part, an ill-written query is transformed into a fast executable code. The code is then executed and the results are returned to the client manager. It’s a multiple-step operation:

the query is first parsed to see if it’s valid
it’s then rewritten to remove useless operations and add some pre-optimizations
it’s then optimized to improve the performances and transformed into an execution and data access plan.
then the plan is compiled
at last, it’s executed
In this part, I won’t talk a lot about the last 2 points because they’re less important.

 

After reading this part, if you want a better understanding I recommend reading:

The initial research paper (1979) on cost based optimization: Access Path Selection in a Relational Database Management System. This article is only 12 pages and understandable with an average level in computer science.
A very good and in-depth presentation on how DB2 9.X optimizes queries here
A very good presentation on how PostgreSQL optimizes queries here. It’s the most accessible document since it’s more a presentation on “let’s see what query plans PostgreSQL gives in these situations“ than a “let’s see the algorithms used by PostgreSQL”.
The official SQLite documentation about optimization. It’s “easy” to read because SQLite uses simple rules. Moreover, it’s the only official documentation that really explains how it works.
A good presentation on how SQL Server 2005 optimizes queries here
A white paper about optimization in Oracle 12c here
2 theoretical courses on query optimization from the authors of the book “DATABASE SYSTEM CONCEPTS” here and there. A good read that focuses on disk I/O cost but a good level in CS is required.
Another theoretical course that I find more accessible but that only focuses on join operators and disk I/O.
 

Query parser

Each SQL statement is sent to the parser where it is checked for correct syntax. If you made a mistake in your query the parser will reject the query. For example, if you wrote “SLECT …” instead of “SELECT …”,  the story ends here.

But this goes deeper. It also checks that the keywords are used in the right order. For example a WHERE before a SELECT will be rejected.

Then, the tables and the fields inside the query are analyzed. The parser uses the metadata of the database to check:

If the tables exist
If the fields of the tables exist
If the operations for the types of the fields are possible (for example you can’t compare an integer with a string, you can’t use a substring() function on an integer)
 

Then it checks if you have the authorizations to read (or write) the tables in the query. Again, these access rights on tables are set by your DBA.

During this parsing, the SQL query is transformed into an internal representation (often a tree)

If everything is ok then the internal representation is sent to the query rewriter.

 

Query rewriter

At this step, we have an internal representation of a query. The aim of the rewriter is:

to pre-optimize the query
to avoid unnecessary operations
to help the optimizer to find the best possible solution
 

The rewriter executes a list of known rules on the query. If the query fits a pattern of a rule, the rule is applied and the query is rewritten.  Here is a non-exhaustive list of (optional) rules:

View merging: If you’re using a view in your query, the view is transformed with the SQL code of the view.
Subquery flattening: Having subqueries is very difficult to optimize so the rewriter will try to modify a query with a subquery to remove the subquery.
For example

SELECT PERSON.*
FROM PERSON
WHERE PERSON.person_key IN
(SELECT MAILS.person_key
FROM MAILS
WHERE MAILS.mail LIKE 'christophe%');
Will be replaced by

SELECT PERSON.*
FROM PERSON, MAILS
WHERE PERSON.person_key = MAILS.person_key
and MAILS.mail LIKE 'christophe%';
Removal of unnecessary operators: For example if you use a DISTINCT whereas you have a UNIQUE constraint that prevents the data from being non-unique, the DISTINCT keyword is removed.
Redundant join elimination: If you have twice the same join condition because one join condition is hidden in a view or if by transitivity there is a useless join, it’s removed.
Constant arithmetic evaluation: If you write something that requires a calculus, then it’s computed once during the rewriting. For example WHERE AGE > 10+2 is transformed into WHERE AGE > 12 and TODATE(“some date”) is transformed into the date in the datetime format
(Advanced) Partition Pruning: If you’re using a partitioned table, the rewriter is able to find what partitions to use.
(Advanced) Materialized view rewrite: If you have a materialized view that matches a subset of the predicates in your query, the rewriter checks if the view is up to date and modifies the query to use the materialized view instead of the raw tables.
(Advanced) Custom rules: If you have custom rules to modify a query (like Oracle policies), then the rewriter executes these rules
(Advanced) Olap transformations: analytical/windowing functions, star joins, rollup … are also transformed (but I’m not sure if it’s done by the rewriter or the optimizer, since both processes are very close it must depends on the database).
 

This rewritten query is then sent to the query optimizer where the fun begins!

 

Statistics

Before we see how a database optimizes a query we need to speak about statistics because without them a database is stupid. If you don’t tell the database to analyze its own data, it will not do it and it will make (very) bad assumptions.

But what kind of information does a database need?

I have to (briefly) talk about how databases and Operating systems store data. They’re using a minimum unit called a page or a block (4 or 8 kilobytes by default). This means that if you only need 1 Kbytes it will cost you one page anyway. If the page takes 8 Kbytes then you’ll waste 7 Kbytes.

 

Back to the statistics! When you ask a database to gather statistics, it computes values like:

The number of rows/pages in a table
For each column in a table:
distinct data values
the length of data values (min, max, average)
data range information (min, max, average)
Information on the indexes of the table.
These statistics will help the optimizer to estimate the disk I/O, CPU and memory usages of the query.

The statistics for each column are very important. For example if a table PERSON needs to be joined on 2 columns: LAST_NAME, FIRST_NAME. With the statistics, the database knows that there are only 1 000 different values on FIRST_NAME and 1 000 000 different values on LAST_NAME. Therefore, the database will join the data on LAST_NAME, FIRST_NAME instead of FIRST_NAME,LAST_NAME because it produces way less comparisons since the LAST_NAME are unlikely to be the same so most of the time a comparison on the 2 (or 3) first characters of the LAST_NAME is enough.

 

But these are basic statistics. You can ask a database to compute advanced statistics called histograms.  Histograms are statistics that inform about the distribution of the values inside the columns. For example

the most frequent values
the quantiles
…
These extra statistics will help the database to find an even better query plan. Especially for equality predicate (ex: WHERE AGE = 18 ) or range predicates (ex: WHERE  AGE > 10 and AGE <40 ) because the database will have a better idea of the number rows concerned by these predicates (note: the technical word for this concept is selectivity).

 

The statistics are stored in the metadata of the database. For example you can see the statistics for the (non-partitioned) tables:

in USER/ALL/DBA_TABLES and USER/ALL/DBA_TAB_COLUMNS for Oracle
in SYSCAT.TABLES and SYSCAT.COLUMNS for DB2.
 

The statistics have to be up to date. There is nothing worse than a database thinking a table has only 500 rows whereas it has 1 000 000 rows. The only drawback of the statistics is that it takes time to compute them. This is why they’re not automatically computed by default in most databases. It becomes difficult with millions of data to compute them. In this case, you can choose to compute only the basics statistics or to compute the stats on a sample of the database.

For example, when I was working on a project dealing with hundreds of millions rows in each tables, I chose to compute the statistics on only 10%, which led to a huge gain in time. For the story it turned out to be a bad decision because occasionally the 10% chosen by Oracle 10G for a specific column of a specific table were very different from the overall 100% (which is very unlikely to happen for a table with 100M rows). This wrong statistic led to a query taking occasionally 8 hours instead of 30 seconds; a nightmare to find the root cause. This example shows how important the statistics are.

 

Note: Of course, there are more advanced statistics specific for each database. If you want to know more, read the documentations of the databases. That being said, I’ve tried to understand how the statistics are used and the best official documentation I found was the one from PostgreSQL.

 

Query optimizer
![Query optimizer](http://coding-geek.com/wp-content/uploads/2015/08/15-McDonalds_CBO.jpg)
All modern databases are using a Cost Based Optimization (or CBO) to optimize queries. The idea is to put a cost an every operation and find the best way to reduce the cost of the query by using the cheapest chain of operations to get the result.

 

To understand how a cost optimizer works I think it’s good to have an example to “feel” the complexity behind this task. In this part I’ll present you the 3 common ways to join 2 tables and we will quickly see that even a simple join query is a nightmare to optimize. After that, we’ll see how real optimizers do this job.

 

For these joins, I’ll focus on their time complexity but a database optimizer computes their CPU cost, disk I/O cost and memory requirement. The difference between time complexity and CPU cost is that time cost is very approximate (it’s for lazy guys like me). For the CPU cost, I should count every operation like an addition, an “if statement”, a multiplication, an iteration … Moreover:

Each high level code operation has a specific number of low level CPU operations.
The cost of a CPU operation is not the same (in terms of CPU cycles) whether you’re using an Intel Core i7, an Intel Pentium 4, an AMD Opteron…. In other words it depends on the CPU architecture.
 

Using the time complexity is easier (at least for me) and with it we can still get the concept of CBO. I’ll sometimes speak about disk I/O since it’s an important concept. Keep in mind that the bottleneck is most of the time the disk I/O and not the CPU usage.

 

Indexes

We talked about indexes when we saw the B+Trees. Just remember that these indexes are already sorted.

FYI, there are other types of indexes like bitmap indexes. They don’t offer the same cost in terms of CPU, disk I/O and memory than B+Tree indexes.

Moreover, many modern databases can dynamically create temporary indexes just for the current query if it can improve the cost of the execution plan.

 

Access Path

Before applying your join operators, you first need to get your data. Here is how you can get your data.

Note: Since the real problem with all the access paths is the disk I/O, I won’t talk a lot about time complexity.

 

Full scan

If you’ve ever read an execution plan you must have seen the word full scan (or just scan). A full scan is simply the database reading a table or an index entirely. In terms of disk I/O, a table full scan is obviously more expensive than an index full scan.

 

Range Scan

There are other types of scan like index range scan. It is used for example when you use a predicate like “WHERE AGE > 20 AND AGE <40”.

Of course you need have an index on the field AGE to use this index range scan.

We already saw in the first part that the time cost of a range query is something like log(N) +M, where N is the number of data in this index and M an estimation of the number of rows inside this range. Both N and M values are known thanks to the statistics (Note: M is the selectivity for the predicate AGE >20 AND AGE<40). Moreover, for a range scan you don’t need to read the full index so it’s less expensive in terms of disk I/O than a full scan.

 

Unique scan

If you only need one value from an index you can use the unique scan.

 

Access by row id

Most of the time, if the database uses an index, it will have to look for the rows associated to the index. To do so it will use an access by row id.

 

For example, if you do something like

SELECT LASTNAME, FIRSTNAME from PERSON WHERE AGE = 28
If you have an index for person on column age, the optimizer will use the index to find all the persons who are 28 then it will ask for the associate rows in the table because the index only has information about the age and you want to know the lastname and the firstname.

 

But, if now you do something like

SELECT TYPE_PERSON.CATEGORY from PERSON ,TYPE_PERSON
WHERE PERSON.AGE = TYPE_PERSON.AGE
The index on PERSON will be used to join with TYPE_PERSON but the table PERSON will not be accessed by row id since you’re not asking information on this table.

Though it works great for a few accesses, the real issue with this operation is the disk I/O. If you need too many accesses by row id the database might choose a full scan.

 

Others paths

I didn’t present all the access paths. If you want to know more, you can read the Oracle documentation. The names might not be the same for the other databases but the concepts behind are the same.

 

Join operators

So, we know how to get our data, let’s join them!

I’ll present the 3 common join operators: Merge Join, Hash Join and Nested Loop Join. But before that, I need to introduce new vocabulary: inner relation and outer relation. A relation can be:

a table
an index
an intermediate result from a previous operation (for example the result of a previous join)
When you’re joining two relations, the join algorithms manage the two relations differently. In the rest of the article, I’ll assume that:

the outer relation is the left data set
the inner relation is the right data set
For example, A JOIN B is the join between A and B where A is the outer relation and B the inner relation.

Most of the time, the cost of A JOIN B is not the same as the cost of B JOIN A.

In this part, I’ll also assume that the outer relation has N elements and the inner relation M elements. Keep in mind that a real optimizer knows the values of N and M with the statistics.

Note: N and M are the cardinalities of the relations.

 

Nested loop join

The nested loop join is the easiest one.
![Nest Loop join](http://coding-geek.com/wp-content/uploads/2015/08/nested_loop_join.png)
Here is the idea:

for each row in the outer relation
you look at all the rows in the inner relation to see if there are rows that match
Here is a pseudo code:

nested_loop_join(array outer, array inner)
  for each row a in outer
    for each row b in inner
      if (match_join_condition(a,b))
        write_result_in_output(a,b)
      end if
    end for
   end for
Since it’s a double iteration, the time complexity is O(N*M)

 

In term of disk I/O, for each of the N rows in the outer relation, the inner loop needs to read M rows from the inner relation. This algorithm needs to read N + N*M rows from disk. But, if the inner relation is small enough, you can put the relation in memory and just have M +N reads. With this modification, the inner relation must be the smallest one since it has more chance to fit in memory.

In terms of time complexity it makes no difference but in terms of disk I/O it’s way better to read only once both relations.      

Of course, the inner relation can be replaced by an index, it will be better for the disk I/O.

 

Since this algorithm is very simple, here is another version that is more disk I/O friendly if the inner relation is too big to fit in memory. Here is the idea:

instead of reading both relation row by row,
you read them bunch by bunch and keep 2 bunches of rows (from each relation) in memory,
you compare the rows inside the two bunches and keep the rows that match,
then you load new bunches from disk and compare them
and so on until there are no bunches to load.
Here is a possible algorithm:

// improved version to reduce the disk I/O.
nested_loop_join_v2(file outer, file inner)
  for each bunch ba in outer
  // ba is now in memory
    for each bunch bb in inner
        // bb is now in memory
        for each row a in ba
          for each row b in bb
            if (match_join_condition(a,b))
              write_result_in_output(a,b)
            end if
          end for
       end for
    end for
   end for
 

With this version, the time complexity remains the same, but the number of disk access decreases:

With the previous version, the algorithm needs N + N*M accesses (each access gets one row).
With this new version, the number of disk accesses becomes number_of_bunches_for(outer)+ number_of_ bunches_for(outer)* number_of_ bunches_for(inner).
If you increase the size of the bunch you reduce the number of disk accesses.
Note: Each disk access gathers more data than the previous algorithm but it doesn’t matter since they’re sequential accesses (the real issue with mechanical disks is the time to get the first data).

 

Hash join

The hash join is more complicated but gives a better cost than a nested loop join in many situations.
![Hash Join](http://coding-geek.com/wp-content/uploads/2015/08/hash_join.png)
The idea of the hash join is to:

1) Get all elements from the inner relation
2) Build an in-memory hash table
3) Get all elements of the outer relation one by one
4) Compute the hash of each element (with the hash function of the hash table) to find the associated bucket of the inner relation
5) find if there is a match between the elements in the bucket and the element of the outer table
In terms of time complexity I need to make some assumptions to simplify the problem:

The inner relation is divided into X buckets
The hash function distributes hash values almost uniformly for both relations. In other words the buckets are equally sized.
The matching between an element of the outer relation and all elements inside a bucket cost the number of elements inside the buckets.
The time complexity is (M/X) * (N/X) + cost_to_create_hash_table(M) + cost_of_hash_function*N

If the Hash function creates enough small-sized buckets then the time complexity is O(M+N)

 

Here is another version of the hash join which is more memory friendly but less disk I/O friendly. This time:

1) you compute the hash tables for both the inner and outer relations
2) then you put them on disk
3) then you compare the 2 relations bucket by bucket (with one loaded in-memory and the other read row by row)
 

Merge join

The merge join is the only join that produces a sorted result.

Note: In this simplified merge join, there are no inner or outer tables; they both play the same role. But real implementations make a difference, for example, when dealing with duplicates.

The merge join can be divided into of two steps:

(Optional) Sort join operations: Both the inputs are sorted on the join key(s).
Merge join operation: The sorted inputs are merged together.
 

Sort

We already spoke about the merge sort, in this case a merge sort in a good algorithm (but not the best if memory is not an issue).

But sometimes the data sets are already sorted, for example:

If the table is natively ordered, for example an index-organized table on the join condition
If the relation is an index on the join condition
If this join is applied on an intermediate result already sorted during the process of the query
 

Merge join
![Merge join](http://coding-geek.com/wp-content/uploads/2015/08/merge_join.png)
This part is very similar to the merge operation of the merge sort we saw. But this time, instead of picking every element from both relations, we only pick the elements from both relations that are equals. Here is the idea:

1) you compare both current elements in the 2 relations (current=first for the first time)
2) if they’re equal, then you put both elements in the result and you go to the next element for both relations
3) if not, you go to the next element for the relation with the lowest element (because the next element might match)
4) and repeat 1,2,3 until you reach the last element of one of the relation.
This works because both relations are sorted and therefore you don’t need to “go back” in these relations.

This algorithm is a simplified version because it doesn’t handle the case where the same data appears multiple times in both arrays (in other words a multiple matches). The real version is more complicated “just” for this case; this is why I chose a simplified version.

 

If both relations are already sorted then the time complexity is O(N+M)

If both relations need to be sorted then the time complexity is the cost to sort both relations: O(N*Log(N) + M*Log(M))

 

For the CS geeks, here is a possible algorithm that handles the multiple matches (note: I’m not 100% sure about my algorithm):

mergeJoin(relation a, relation b)
  relation output
  integer a_key:=0;
  integer b_key:=0;
 
  while (a[a_key]!=null and b[b_key]!=null)
    if (a[a_key] < b[b_key]) a_key++; else if (a[a_key] > b[b_key])
      b_key++;
    else //Join predicate satisfied
      write_result_in_output(a[a_key],b[b_key])
      //We need to be careful when we increase the pointers
      integer a_key_temp:=a_key;
      integer b_key_temp:=b_key;
      if (a[a_key+1] != b[b_key])
        b_key_temp:= b_key + 1;
      end if
      if (b[b_key+1] != a[a_key])
        a_key_temp:= a_key + 1;
      end if
      if (b[b_key+1] == a[a_key] && b[b_key] == a[a_key+1])
        a_key_temp:= a_key + 1;
        b_key_temp:= b_key + 1;
      end if
      a_key:= a_key_temp;
      b_key:= b_key_temp;
    end if
  end while
 

Which one is the best?

If there was a best type of joins, there wouldn’t be multiple types. This question is very difficult because many factors come into play like:

The amount of free memory: without enough memory you can say goodbye to the powerful hash join (at least the full in-memory hash join)
The size of the 2 data sets. For example if you have a big table with a very small one, a nested loop join will be faster than a hash join because the hash join has an expensive creation of hashes. If you have 2 very large tables the nested loop join will be very CPU expensive.
The presence of indexes. With 2 B+Tree indexes the smart choice seems to be the merge join
If the result need to be sorted: Even if you’re working with unsorted data sets, you might want to use a costly merge join (with the sorts) because at the end the result will be sorted and you’ll be able to chain the result with another merge join (or maybe because the query asks implicitly/explicitly for a sorted result with an ORDER BY/GROUP BY/DISTINCT operation)
If the relations are already sorted: In this case the merge join is the best candidate
The type of joins you’re doing: is it an equijoin (i.e.: tableA.col1 = tableB.col2)? Is it an inner join, an outer join, a cartesian product or a self-join? Some joins can’t work in certain situations.
The distribution of data. If the data on the join condition are skewed (For example you’re joining people on their last name but many people have the same), using a hash join will be a disaster because the hash function will create ill-distributed buckets.
If you want the join to be executed by multiple threads/process
 

For more information, you can read the DB2, ORACLE or SQL Server documentations.

 

Simplified example

We’ve just seen 3 types of join operations.

Now let’s say we need to join 5 tables to have a full view of a person. A PERSON can have:

multiple MOBILES
multiple MAILS
multiple ADRESSES
multiple BANK_ACCOUNTS
In other words we need a quick answer for the following query:

SELECT * from PERSON, MOBILES, MAILS,ADRESSES, BANK_ACCOUNTS
WHERE
PERSON.PERSON_ID = MOBILES.PERSON_ID
AND PERSON.PERSON_ID = MAILS.PERSON_ID
AND PERSON.PERSON_ID = ADRESSES.PERSON_ID
AND PERSON.PERSON_ID = BANK_ACCOUNTS.PERSON_ID
As a query optimizer, I have to find the best way to process the data. But there are 2 problems:

What kind of join should I use for each join?
I have 3 possible joins (Hash Join, Merge Join, Nested Join) with the possibility to use 0,1 or 2 indexes (not to mention that there are different types of indexes).

What order should I choose to compute the join?
For example, the following figure shows different possible plans for only 3 joins on 4 tables
![Join Ordering problem](http://coding-geek.com/wp-content/uploads/2015/08/join_ordering_problem.png)
So here are my possibilities:

1) I use a brute force approach
Using the database statistics, I compute the cost for every possible plan and I keep the best one. But there are many possibilities. For a given order of joins, each join has 3 possibilities: HashJoin, MergeJoin, NestedJoin. So, for a given order of joins there are 34 possibilities. The join ordering is a permutation problem on a binary tree and there are (2*4)!/(4+1)! possible orders. For this very simplified problem, I end up with 34*(2*4)!/(4+1)! possibilities.

In non-geek terms, it means 27 216 possible plans. If I now add the possibility for the merge join to take 0,1 or 2 B+Tree indexes, the number of possible plans becomes 210 000. Did I forget to mention that this query is VERY SIMPLE?

2) I cry and quit this job
It’s very tempting but you wouldn’t get your result and I need money to pay the bills.

3) I only try a few plans and take the one with the lowest cost.
Since I’m not superman, I can’t compute the cost of every plan. Instead, I can arbitrary choose a subset of all the possible plans, compute their costs and give you the best plan of this subset.

4) I apply smart rules to reduce the number of possible plans.
There are 2 types of rules:

I can use “logical” rules that will remove useless possibilities but they won’t filter a lot of possible plans. For example: “the inner relation of the nested loop join must be the smallest data set”

I accept not finding the best solution and apply more aggressive rules to reduce a lot the number of possibilities. For example “If a relation is small, use a nested loop join and never use a merge join or a hash join”

 

In this simple example, I end up with many possibilities. But a real query can have other relational operators like OUTER JOIN, CROSS JOIN, GROUP BY, ORDER BY, PROJECTION, UNION, INTERSECT, DISTINCT … which means even more possibilities.

So, how a database does it?

 

Dynamic programming, greedy algorithm and heuristic

A relational database tries the multiple approaches I’ve just said. The real job of an optimizer is to find a good solution on a limited amount of time.

Most of the time an optimizer doesn’t find the best solution but a “good” one.

For small queries, doing a brute force approach is possible. But there is a way to avoid unnecessary computations so that even medium queries can use the brute force approach. This is called dynamic programming.

 

Dynamic Programming

The idea behind these 2 words is that many executions plan are very similar. If you look at the following plans:
![overlapping_trees](http://coding-geek.com/wp-content/uploads/2015/08/overlapping_trees.png)
They share the same (A JOIN B) subtree. So, instead of computing the cost of this subtree in every plan, we can compute it once, save the computed cost and reuse it when we see this subtree again. More formally, we’re facing an overlapping problem. To avoid the extra-computation of the partial results we’re using memoization.

Using this technique, instead of having a (2*N)!/(N+1)! time complexity, we “just” have 3N. In our previous example with 4 joins, it means passing from 336 ordering to 81. If you take a bigger query with 8 joins (which is not big), it means passing from 57 657 600 to 6561.

 

For the CS geeks, here is an algorithm I found on the formal course I already gave you. I won’t explain this algorithm so read it only if you already know dynamic programming or if you’re good with algorithms (you’ve been warned!):

procedure findbestplan(S)
if (bestplan[S].cost infinite)
   return bestplan[S]
// else bestplan[S] has not been computed earlier, compute it now
if (S contains only 1 relation)
         set bestplan[S].plan and bestplan[S].cost based on the best way
         of accessing S  /* Using selections on S and indices on S */
     else for each non-empty subset S1 of S such that S1 != S
   P1= findbestplan(S1)
   P2= findbestplan(S - S1)
   A = best algorithm for joining results of P1 and P2
   cost = P1.cost + P2.cost + cost of A
   if cost < bestplan[S].cost
       bestplan[S].cost = cost
      bestplan[S].plan = “execute P1.plan; execute P2.plan;
                 join results of P1 and P2 using A”
return bestplan[S]
 

For bigger queries you can still do a dynamic programming approach but with extra rules (or heuristics) to remove possibilities:

If we analyze only a certain type of plan (for example: the left-deep trees) we end up with n*2n instead of 3n
![left deep tree](http://coding-geek.com/wp-content/uploads/2015/08/left-deep-tree.png)
If we add logical rules to avoid plans for some patterns (like “if a table as an index for the given predicate, don’t try a merge join on the table but only on the index”) it will reduce the number of possibilities without hurting to much the best possible solution.
If we add rules on the flow (like “perform the join operations BEFORE all the other relational operations”) it also reduces a lot of possibilities.
…
 

Greedy algorithms

But for a very big query or to have a very fast answer (but not a very fast query), another type of algorithms is used, the greedy algorithms.

The idea is to follow a rule (or heuristic) to build a query plan in an incremental way. With this rule, a greedy algorithm finds the best solution to a problem one step at a time.  The algorithm starts the query plan with one JOIN. Then, at each step, the algorithm adds a new JOIN to the query plan using the same rule.

 

Let’s take a simple example. Let’s say we have a query with 4 joins on 5 tables (A, B, C, D and E). To simplify the problem we just take the nested join as a possible join. Let’s use the rule “use the join with the lowest cost”

we arbitrary start on one of the 5 tables (let’s choose A)
we compute the cost of every join with A (A being the inner or outer relation).
we find that A JOIN B gives the lowest cost.
we then compute the cost of every join with the result of A JOIN B (A JOIN B being the inner or outer relation).
we find that (A JOIN B) JOIN C gives the best cost.
we then compute the cost of every join with the result of the (A JOIN B) JOIN C …
….
At the end we find the plan (((A JOIN B) JOIN C) JOIN D) JOIN E)
Since we arbitrary started with A, we can apply the same algorithm for B, then C then D then E. We then keep the plan with the lowest cost.

By the way, this algorithm has a name: it’s called the Nearest neighbor algorithm.

I won’t go into details, but with a good modeling and a sort in N*log(N) this problem can easily be solved. The cost of this algorithm is in O(N*log(N)) vs O(3N) for the full dynamic programming version. If you have a big query with 20 joins, it means 26 vs 3 486 784 401, a BIG difference!

 

The problem with this algorithm is that we assume that finding the best join between 2 tables will give us the best cost if we keep this join and add a new join.  But:

even if A JOIN B gives the best cost between A, B and C
(A JOIN C) JOIN B might give a better result than (A JOIN B) JOIN C.
To improve the result, you can run multiple greedy algorithms using different rules and keep the best plan.

 

Other algorithms

[If you’re already fed up with algorithms, skip to the next part, what I’m going to say is not important for the rest of the article]

The problem of finding the best possible plan is an active research topic for many CS researchers. They often try to find better solutions for more precise problems/patterns. For example,

if the query is a star join (it’s a certain type of multiple-join query), some databases will use a specific algorithm.
if the query is a parallel query, some databases will use a specific algorithm
…
 

Other algorithms are also studied to replace dynamic programming for large queries. Greedy algorithms belong to larger family called heuristic algorithms. A greedy algorithm follows a rule (or heuristic), keeps the solution it found at the previous step and “appends” it to find the solution for the current step. Some algorithms follow a rule and apply it in a step-by-step way but don’t always keep the best solution found in the previous step. They are called heuristic algorithms.

For example, genetic algorithms follow a rule but the best solution of the last step is not often kept:

A solution represents a possible full query plan
Instead of one solution (i.e. plan) there are P solutions (i.e. plans) kept at each step.
0) P query plans are randomly created
1) Only the plans with the best costs are kept
2) These best plans are mixed up to produce P news plans
3) Some of the P new plans are randomly modified
4) The step 1,2,3 are repeated T times
5) Then you keep the best plan from the P plans of the last loop.
The more loops you do the better the plan will be.

Is it magic? No, it’s the laws of nature: only the fittest survives!

FYI, genetic algorithms are implemented in PostgreSQL but I wasn’t able to find if they’re used by default.

There are other heuristic algorithms used in databases like Simulated Annealing, Iterative Improvement, Two-Phase Optimization… But I don’t know if they’re currently used in enterprise databases or if they’re only used in research databases.

 

Real optimizers

[You can skip to the next part, what I’m going to say is not important]

But, all this blabla is very theoretical. Since I’m a developer and not a researcher, I like concrete examples.

Let’s see how the SQLite optimizer works. It’s a light database so it uses a simple optimization based on a greedy algorithm with extra-rules to limit the number of possibilities:

SQLite chooses to never reorder tables in a CROSS JOIN operator
joins are implemented as nested joins
outer joins are always evaluated in the order in which they occur
…
Prior to version 3.8.0, SQLite uses the “Nearest Neighbor” greedy algorithm when searching for the best query plan
Wait a minute … we’ve already seen this algorithm! What a coincidence!

Since version 3.8.0 (released in 2015), SQLite uses the “N Nearest Neighbors” greedy algorithm when searching for the best query plan
 

Let’s see how another optimizer does his job. IBM DB2 is like all the enterprise databases but I’ll focus on this one since it’s the last one I’ve really used before switching to Big Data.

If we look at the official documentation, we learn that the DB2 optimizer let you use 7 different levels of optimization:

Use greedy algorithms for the joins
0 – minimal optimization, use index scan and nested-loop join and avoid some Query Rewrite
1 – low optimization
2 – full optimization
Use dynamic programming for the joins
3 – moderate optimization and rough approximation
5 – full optimization, uses all techniques with heuristics
7 – full optimization similar to 5, without heuristics
9 – maximal optimization spare no effort/expense considers all possible join orders, including Cartesian products
We can see that DB2 uses greedy algorithms and dynamic programming. Of course, they don’t share the heuristics they use since the query optimizer is the main power of a database.

FYI, the default level is 5. By default the optimizer uses the following characteristics:

All available statistics, including frequent-value and quantile statistics, are used.
All query rewrite rules (including materialized query table routing) are applied, except computationally intensive rules that are applicable only in very rare cases.
Dynamic programming join enumeration is used, with:
Limited use of composite inner relation
Limited use of Cartesian products for star schemas involving lookup tables
A wide range of access methods is considered, including list prefetch (note: will see what is means), index ANDing (note: a special operation with indexes), and materialized query table routing.
By default, DB2 uses dynamic programming limited by heuristics for the join ordering.

The others conditions (GROUP BY, DISTINCT…) are handled by simple rules.

 

Query Plan Cache

Since the creation of a plan takes time, most databases store the plan into a query plan cache to avoid useless re-computations of the same query plan. It’s kind of a big topic since the database needs to know when to update the outdated plans. The idea is to put a threshold and if the statistics of a table have changed above this threshold then the query plan involving this table is purged from the cache.

 

Query executor

At this stage we have an optimized execution plan. This plan is compiled to become an executable code. Then, if there are enough resources (memory, CPU) it is executed by the query executor. The operators in the plan (JOIN, SORT BY …) can be executed in a sequential or parallel way; it’s up to the executor. To get and write its data, the query executor interacts with the data manager, which is the next part of the article.

 

Data manager
![Data manager](http://coding-geek.com/wp-content/uploads/2015/08/data_manager.png)
At this step, the query manager is executing the query and needs the data from the tables and indexes. It asks the data manager to get the data, but there are 2 problems:

Relational databases use a transactional model. So, you can’t get any data at any time because someone else might be using/modifying the data at the same time.
Data retrieval is the slowest operation in a database, therefore the data manager needs to be smart enough to get and keep data in memory buffers.
In this part, we’ll see how relational databases handle these 2 problems. I won’t talk about the way the data manager gets its data because it’s not the most important (and this article is long enough!).

 

Cache manager

As I already said, the main bottleneck of databases is disk I/O. To improve performance, modern databases use a cache manager.
![Cache manager](http://coding-geek.com/wp-content/uploads/2015/08/cache_manager.png)
Instead of directly getting the data from the file system, the query executor asks for the data to the cache manager. The cache manager has an in-memory cache called buffer pool. Getting data from memory dramatically speeds up a database. It’s difficult to give an order of magnitude because it depends on the operation you need to do:

sequential access (ex: full scan) vs random access (ex: access by row id),
read vs write
and the type of disks used by the database:

7.2k/10k/15k rpm HDD
SSD
RAID 1/5/…
but I’d say memory is 100 to 100k times faster than disk.

But, this leads to another problem (as always with databases…). The cache manager needs to get the data in memory BEFORE the query executor uses them; otherwise the query manager has to wait for the data from the slow disks.

 

Prefetching

This problem is called prefetching. A query executor knows the data it’ll need because it knows the full flow of the query and has knowledge of the data on disk with the statistics. Here is the idea:

When the query executor is processing its first bunch of data
It asks the cache manager to pre-load the second bunch of data
When it starts processing the second bunch of data
It asks the CM to pre-load the third bunch and informs the CM that the first bunch can be purged from cache.
…
The CM stores all these data in its buffer pool. In order to know if a data is still needed, the cache manager adds an extra-information about the cached data (called a latch).

 

Sometimes the query executor doesn’t know what data it’ll need and some databases don’t provide this functionality. Instead, they use a speculative prefetching (for example: if the query executor asked for data 1,3,5 it’ll likely ask for 7,9,11 in a near future) or a sequential prefetching (in this case the CM simply loads from disks the next contiguous data after the ones asked).

 

To monitor how well the prefetching is working, modern databases provide a metric called buffer/cache hit ratio. The hit ratio shows how often a requested data has been found in the buffer cache without requiring disk access.

Note: a poor cache hit ratio doesn’t always mean that the cache is ill-working. For more information, you can read the Oracle documentation.

 

But, a buffer is a limited amount of memory. Therefore, it needs to remove some data to be able to load new ones. Loading and purging the cache has a cost in terms of disk and network I/O. If you have a query that is often executed, it wouldn’t be efficient to always load then purge the data used by this query. To handle this problem, modern databases use a buffer replacement strategy.

 

Buffer-Replacement strategies

Most modern databases (at least SQL Server, MySQL, Oracle and DB2) use an LRU algorithm.

 

LRU

LRU stands for Least Recently Used. The idea behind this algorithm is to keep in the cache the data that have been recently used and, therefore, are more likely to be used again.

Here is a visual example:
![LRU](http://coding-geek.com/wp-content/uploads/2015/08/LRU.png)
For the sake of comprehension, I’ll assume that the data in the buffer are not locked by latches (and therefore can be removed). In this simple example the buffer can store 3 elements:

1: the cache manager uses the data 1 and puts the data into the empty buffer
2: the CM uses the data 4 and puts the data into the half-loaded buffer
3: the CM uses the data 3 and puts the data into the half-loaded buffer
4: the CM uses the data 9. The buffer is full so data 1 is removed since it’s the last recently used data. Data 9 is added into the buffer
5: the CM uses the data 4. Data 4 is already in the buffer therefore it becomes the first recently used data again.
6: the CM uses the data 1. The buffer is full so data 9 is removed since it’s the last recently used data. Data 1 is added into the buffer
…
This algorithm works well but there are some limitations. What if there is a full scan on a large table? In other words, what happens when the size of the table/index is above the size of the buffer? Using this algorithm will remove all the previous values in the cache whereas the data from the full scan are likely to be used only once.

 

Improvements

To prevent this to happen, some databases add specific rules. For example according to Oracle documentation:

“For very large tables, the database typically uses a direct path read, which loads blocks directly […], to avoid populating the buffer cache. For medium size tables, the database may use a direct read or a cache read. If it decides to use a cache read, then the database places the blocks at the end of the LRU list to prevent the scan from effectively cleaning out the buffer cache.”

There are other possibilities like using an advanced version of LRU called LRU-K. For example SQL Server uses LRU-K for K =2.

This idea behind this algorithm is to take into account more history. With the simple LRU (which is also LRU-K for K=1), the algorithm only takes into account the last time the data was used. With the LRU-K:

It takes into account the K last times the data was used.
A weight is put on the number of times the data was used
If a bunch of new data is loaded into the cache, the old but often used data are not removed (because their weights are higher).
But the algorithm can’t keep old data in the cache if they aren’t used anymore.
So the weights decrease over time if the data is not used.
The computation of the weight is costly and this is why SQL Server only uses K=2. This value performs well for an acceptable overhead.

For a more in-depth knowledge of LRU-K, you can read the original research paper (1993): The LRU-K page replacement algorithm for database disk buffering.

 

Other algorithms

Of course there are other algorithms to manage cache like

2Q (a LRU-K like algorithm)
CLOCK (a LRU-K like algorithm)
MRU (most recently used, uses the same logic than LRU but with another rule)
LRFU (Least Recently and Frequently Used)
…
Some databases let the possibility to use another algorithm than the default one.

 

Write buffer

I only talked about read buffers that load data before using them. But in a database you also have write buffers that store data and flush them on disk by bunches instead of writing data one by one and producing many single disk accesses.

 

Keep in mind that buffers store pages (the smallest unit of data) and not rows (which is a logical/human way to see data). A page in a buffer pool is dirty if the page has been modified and not written on disk. There are multiple algorithms to decide the best time to write the dirty pages on disk but it’s highly linked to the notion of transaction, which is the next part of the article.

 

Transaction manager

Last but not least, this part is about the transaction manager. We’ll see how this process ensures that each query is executed in its own transaction. But before that, we need to understand the concept of ACID transactions.

 

I’m on acid

An ACID transaction is a unit of work that ensures 4 things:

Atomicity: the transaction is “all or nothing”, even if it lasts 10 hours. If the transaction crashes, the state goes back to before the transaction (the transaction is rolled back).
Isolation: if 2 transactions A and B run at the same time, the result of transactions A and B must be the same whether A finishes before/after/during transaction B.
Durability: once the transaction is committed (i.e. ends successfully), the data stay in the database no matter what happens (crash or error).
Consistency: only valid data (in terms of relational constraints and functional constraints) are written to the database. The consistency is related to atomicity and isolation.
![dollar low](http://coding-geek.com/wp-content/uploads/2015/08/dollar_low.jpg)
During the same transaction, you can run multiple SQL queries to read, create, update and delete data. The mess begins when two transactions are using the same data. The classic example is a money transfer from an account A to an account B.  Imagine you have 2 transactions:

Transaction 1 that takes 100$ from account A and gives them to account B
Transaction 2 that takes 50$ from account A and gives them to account B
If we go back to the ACID properties:

Atomicity ensures that no matter what happens during T1 (a server crash, a network failure …), you can’t end up in a situation where the 100$ are withdrawn from A and not given to B (this case is an inconsistent state).
Isolation ensures that if T1 and T2 happen at the same time, in the end A will be taken 150$ and B given 150$ and not, for example, A taken 150$ and B given just $50 because T2 has partially erased the actions of T1 (this case is also an inconsistent state).
Durability ensures that T1 won’t disappear into thin air if the database crashes just after T1 is committed.
Consistency ensures that no money is created or destroyed in the system.
 

[You can skip to the next part if you want, what I’m going to say is not important for the rest of the article]

Many modern databases don’t use a pure isolation as a default behavior because it comes with a huge performance overhead. The SQL norm defines 4 levels of isolation:

Serializable (default behaviour in SQLite): The highest level of isolation. Two transactions happening at the same time are 100% isolated. Each transaction has its own “world”.
Repeatable read (default behavior in MySQL): Each transaction has its own “world” except in one situation. If a transaction ends up successfully and adds new data, these data will be visible in the other and still running transactions. But if A modifies a data and ends up successfully, the modification won’t be visible in the still running transactions. So, this break of isolation between transactions is only about new data, not the existing ones.
For example,  if a transaction A does a “SELECT count(1) from TABLE_X” and then a new data is added and committed in TABLE_X by Transaction B, if transaction A does again a count(1) the value won’t be the same.

This is called a phantom read.

Read committed (default behavior in Oracle, PostgreSQL and SQL Server): It’s a repeatable read + a new break of isolation. If a transaction A reads a data D and then this data is modified (or deleted) and committed by a transaction B, if A reads data D again it will see the modification (or deletion) made by B on the data.
This is called a non-repeatable read.

Read uncommitted: the lowest level of isolation. It’s a read committed + a new break of isolation. If a transaction A reads a data D and then this data D is modified by a transaction B (that is not committed and still running), if A reads data D again it will see the modified value. If transaction B is rolled back, then data D read by A the second time doesn’t make no sense since it has been modified by a transaction B that never happened (since it was rolled back).
This is called a dirty read.

 

Most databases add their own custom levels of isolation (like the snapshot isolation used by PostgreSQL, Oracle and SQL Server). Moreover, most databases don’t implement all the levels of the SQL norm (especially the read uncommitted level).

The default level of isolation can be overridden by the user/developer at the beginning of the connection (it’s a very simple line of code to add).

 

Concurrency Control

The real issue to ensure isolation, coherency and atomicity is the write operations on the same data (add, update and delete):

if all transactions are only reading data, they can work at the same time without modifying the behavior of another transaction.
if (at least) one of the transactions is modifying a data read by other transactions, the database needs to find a way to hide this modification from the other transactions. Moreover, it also needs to ensure that this modification won’t be erased by another transaction that didn’t see the modified data.
This problem is a called concurrency control.

The easiest way to solve this problem is to run each transaction one by one (i.e. sequentially). But that’s not scalable at all and only one core is working on the multi-processor/core server, not very efficient…

The ideal way to solve this problem is, every time a transaction is created or cancelled:

to monitor all the operations of all the transactions
to check if the parts of 2 (or more) transactions are in conflict because they’re reading/modifying the same data.
to reorder the operations inside the conflicting transactions to reduce the size of the conflicting parts
to execute the conflicting parts in a certain order (while the non-conflicting transactions are still running concurrently).
to take into account that a transaction can be cancelled.
More formally it’s a scheduling problem with conflicting schedules. More concretely, it’s a very difficult and CPU-expensive optimization problem. Enterprise databases can’t afford to wait hours to find the best schedule for each new transaction event. Therefore, they use less ideal approaches that lead to more time wasted between conflicting transactions.

 

Lock manager

To handle this problem, most databases are using locks and/or data versioning. Since it’s a big topic, I’ll focus on the locking part then I’ll speak a little bit about data versioning.

 

Pessimistic locking

The idea behind locking is:

if a transaction needs a data,
it locks the data
if another transaction also needs this data,
it’ll have to wait until the first transaction releases the data.
This is called an exclusive lock.

But using an exclusive lock for a transaction that only needs to read a data is very expensive since it forces other transactions that only want to read the same data to wait. This is why there is another type of lock, the shared lock.

With the shared lock:

if a transaction needs only to read a data A,
it “shared locks” the data and reads the data
if a second transaction also needs only to read data A,
it “shared locks” the data and reads the data
if a third transaction needs to modify data A,
it “exclusive locks” the data but it has to wait until the 2 other transactions release their shared locks to apply its exclusive lock on data A.
Still, if a data as an exclusive lock, a transaction that just needs to read the data will have to wait the end of the exclusive lock to put a shared lock on the data.
![Lock manager](http://coding-geek.com/wp-content/uploads/2015/08/lock_manager.png)
The lock manager is the process that gives and releases locks. Internally, it stores the locks in a hash table (where the key is the data to lock) and knows for each data:

which transactions are locking the data
which transactions are waiting for the data
 

Deadlock

But the use of locks can lead to a situation where 2 transactions are waiting forever for a data:
![Dead Lock](http://coding-geek.com/wp-content/uploads/2015/08/deadlock.png)
In this figure:

transaction A has an exclusive lock on data1 and is waiting to get data2
transaction B has an exclusive lock on data2 and is waiting to get data1
This is called a deadlock.

During a deadlock, the lock manager chooses which transaction to cancel (rollback) in order to remove the deadlock. This decision is not easy:

Is it better to kill the transaction that modified the least amount of data (and therefore that will produce the least expensive rollback)?
Is it better to kill the least aged transaction because the user of the other transaction has waited longer?
Is it better to kill the transaction that will take less time to finish (and avoid a possible starvation)?
In case of rollback, how many transactions will be impacted by this rollback?
 

But before making this choice, it needs to check if there are deadlocks.

The hash table can be seen as a graph (like in the previous figures). There is a deadlock if there is a cycle in the graph. Since it’s expensive to check for cycles (because the graph with all the locks is quite big), a simpler approach is often used: using a timeout. If a lock is not given within this timeout, the transaction enters a deadlock state.

 

The lock manager can also check before giving a lock if this lock will create a deadlock. But again it’s computationally expensive to do it perfectly. Therefore, these pre-checks are often a set of basic rules.

 

Two-phase locking

The simplest way to ensure a pure isolation is if a lock is acquired at the beginning of the transaction and released at the end of the transaction. This means that a transaction has to wait for all its locks before it starts and the locks held by a transaction are released when the transaction ends. It works but it produces a lot of time wasted to wait for all locks.

A faster way is the Two-Phase Locking Protocol (used by DB2 and SQL Server) where a transaction is divided into 2 phases:

the growing phase where a transaction can obtain locks, but can’t release any lock.
the shrinking phase where a transaction can release locks (on the data it has already processed and won’t process again), but can’t obtain new locks.
![two phase locking](http://coding-geek.com/wp-content/uploads/2015/08/two-phase-locking.png)
The idea behind these 2 simple rules is:

to release the locks that aren’t used anymore to reduce the wait time of other transactions waiting for these locks
to prevent from cases where a transaction gets data modified after the transaction started and therefore aren’t coherent with the first data the transaction acquired.
 

This protocol works well except if a transaction that modified a data and released the associated lock is cancelled (rolled back). You could end up in a case where another transaction reads the modified value whereas this value is going to be rolled back. To avoid this problem, all the exclusive locks must be released at the end of the transaction.

 

A few words

Of course a real database uses a more sophisticated system involving more types of locks (like intention locks) and more granularities (locks on a row, on a page, on a partition, on a table, on a tablespace) but the idea remains the same.

I only presented the pure lock-based approach. Data versioning is another way to deal with this problem.

The idea behind versioning is that:

every transaction can modify the same data at the same time
each transaction has its own copy (or version) of the data
if 2 transactions modify the same data, only one modification will be accepted, the other will be refused and the associated transaction will be rolled back (and maybe re-run).
It increases the performance since:

reader transactions don’t block writer transactions
writer transactions don’t block reader transactions
there is no overhead from the “fat and slow” lock manager
Everything is better than locks except when 2 transactions write the same data. Moreover, you can quickly end up with a huge disk space overhead.

 

Data versioning and locking are two different visions: optimistic locking vs pessimistic locking. They both have pros and cons; it really depends on the use case (more reads vs more writes). For a presentation on data versioning, I recommend this very good presentation on how PostgreSQL implements multiversion concurrency control.

Some databases like DB2 (until DB2 9.7) and SQL Server (except for snapshot isolation) are only using locks. Other like PostgreSQL, MySQL and Oracle use a mixed approach involving locks and data versioning. I’m not aware of a database using only data versioning (if you know a database based on a pure data versioning, feel free to tell me).

[UPDATE 08/20/2015] I was told by a reader that:

Firebird and Interbase use versioning without record locking.
Versioning has an interesting effect on indexes: sometimes a unique index contains duplicates, the index can have more entries than the table has rows, etc.

 

If you read the part on the different levels of isolation, when you increase the isolation level you increase the number of locks and therefore the time wasted by transactions to wait for their locks. This is why most databases don’t use the highest isolation level (Serializable) by default.

As always, you can check by yourself in the documentation of the main databases (for example MySQL, PostgreSQL or Oracle).

 

Log manager

We’ve already seen that to increase its performances, a database stores data in memory buffers. But if the server crashes when the transaction is being committed, you’ll lose  the data still in memory during the crash, which breaks the Durability of a transaction.

You can write everything on disk but if the server crashes, you’ll end up with the data half written on disk, which breaks the Atomicity of a transaction.

Any modification written by a transaction must be undone or finished.

To deal with this problem, there are 2 ways:

Shadow copies/pages: Each transaction creates its own copy of the database (or just a part of the database) and works on this copy. In case of error, the copy is removed. In case of success, the database switches instantly the data from the copy with a filesystem trick then it removes the “old” data.
Transaction log: A transaction log is a storage space. Before each write on disk, the database writes an info on the transaction log so that in case of crash/cancel of a transaction, the database knows how to remove (or finish) the unfinished transaction.
 

WAL

The shadow copies/pages creates a huge disk overhead when used on large databases involving many transactions. That’s why modern databases use a transaction log. The transaction log must be stored on a stable storage. I won’t go deeper on storage technologies but using (at least) RAID disks is mandatory to prevent from a disk failure.

Most databases (at least Oracle, SQL Server, DB2, PostgreSQL, MySQL and SQLite) deal with the transaction log using the Write-Ahead Logging protocol (WAL). The WAL protocol is a set of 3 rules:

1) Each modification into the database produces a log record, and the log record must be written into the transaction log before the data is written on disk.
2) The log records must be written in order; a log record A that happens before a log record B must but written before B
3) When a transaction is committed, the commit order must be written on the transaction log before the transaction ends up successfully.
![log manager](http://coding-geek.com/wp-content/uploads/2015/08/log_manager.png)
This job is done by a log manager. An easy way to see it is that between the cache manager and the data access manager (that writes data on disk) the log manager writes every update/delete/create/commit/rollback on the transaction log before they’re written on disk. Easy, right?

 

WRONG ANSWER! After all we’ve been through, you should know that everything related to a database is cursed by the “database effect”. More seriously, the problem is to find a way to write logs while keeping good performances. If the writes on the transaction log are too slow they will slow down everything.

 

ARIES

In 1992, IBM researchers “invented” an enhanced version of WAL called ARIES. ARIES is more or less used by most modern databases. The logic might not be the same but the concepts behind ARIES are used everywhere. I put the quotes on invented because, according to this MIT course, the IBM researchers did “nothing more than writing the good practices of transaction recovery”. Since I was 5 when the ARIES paper was published, I don’t care about this old gossip from bitter researchers. In fact, I only put this info to give you a break before we start this last technical part. I’ve read a huge part of the research paper on ARIES and I find it very interesting! In this part I’ll only give you an overview of ARIES but I strongly recommend to read the paper if you want a real knowledge.

 

ARIES stands for Algorithms for Recovery and Isolation Exploiting Semantics.

The aim of this technique is double:

1) Having good performances when writing logs
2) Having a fast and reliable recovery
 

There are multiple reasons a database has to rollback a transaction:

Because the user cancelled it
Because of server or network failures
Because the transaction has broken the integrity of the database (for example you have a UNIQUE constraint on a column and the transaction adds a duplicate)
Because of deadlocks
 

Sometimes (for example, in case of network failure), the database can recover the transaction.

How is that possible? To answer this question, we need to understand the information stored in a log record.

 

The logs

Each operation (add/remove/modify) during a transaction produces a log. This log record is composed of:

LSN: A unique Log Sequence Number. This LSN is given in a chronological order*. This means that if an operation A happened before an operation B the LSN of log A will be lower than the LSN of log B.
TransID: the id of the transaction that produced the operation.
PageID: the location on disk of the modified data. The minimum amount of data on disk is a page so the location of the data is the location of the page that contains the data.
PrevLSN: A link to the previous log record produced by the same transaction.
UNDO: a way to remove the effect of the operation
For example, if the operation is an update, the UNDO will store either the value/state of the updated element before the update (physical UNDO) or the reverse operation to go back at the previous state (logical UNDO)**.

REDO: a way replay the operation
Likewise, there are 2 ways to do that. Either you store the value/state of the element after the operation or the operation itself to replay it.

…: (FYI, an ARIES log has 2 others fields: the UndoNxtLSN and the Type).
 

Moreover, each page on disk (that stores the data, not the log) has id of the log record (LSN) of the last operation that modified the data.

*The way the LSN is given is more complicated because it is linked to the way the logs are stored. But the idea remains the same.

**ARIES uses only logical UNDO because it’s a real mess to deal with physical UNDO.

Note: From my little knowledge, only PostgreSQL is not using an UNDO. It uses instead a garbage collector daemon that removes the old versions of data. This is linked to the implementation of the data versioning in PostgreSQL.

 

To give you a better idea, here is a visual and simplified example of the log records produced by the query “UPDATE FROM PERSON SET AGE = 18;”. Let’s say this query is executed in transaction 18.
![ARIES log](http://coding-geek.com/wp-content/uploads/2015/08/ARIES_logs.png)
Each log has a unique LSN. The logs that are linked belong to the same transaction. The logs are linked in a chronological order (the last log of the linked list is the log of the last operation).

 

Log Buffer

To avoid that log writing becomes a major bottleneck, a log buffer is used.
![ARIES log writing](http://coding-geek.com/wp-content/uploads/2015/08/ARIES_log_writing.png)
When the query executor asks for a modification:

1) The cache manager stores the modification in its buffer.
2) The log manager stores the associated log in its buffer.
3) At this step, the query executor considers the operation is done (and therefore can ask for other modifications)
4) Then (later) the log manager writes the log on the transaction log. The decision when to write the log is done by an algorithm.
5) Then (later) the cache manager writes the modification on disk. The decision when to write data on disk is done by an algorithm.
 

When a transaction is committed, it means that for every operation in the transaction the steps 1, 2, 3,4,5 are done. Writing in the transaction log is fast since it’s just “adding a log somewhere in the transaction log” whereas writing data on disk is more complicated because it’s “writing the data in a way that it’s fast to read them”.

 

STEAL and FORCE policies

For performance reasons the step 5 might be done after the commit because in case of crashes it’s still possible to recover the transaction with the REDO logs. This is called a NO-FORCE policy.

A database can choose a FORCE policy (i.e. step 5 must be done before the commit) to lower the workload during the recovery.

Another issue is to choose whether the data are written step-by-step on disk (STEAL policy) or if the buffer manager needs to wait until the commit order to write everything at once (NO-STEAL). The choice between STEAL and NO-STEAL depends on what you want: fast writing with a long recovery using UNDO logs or fast recovery?

 

Here is a summary of the impact of these policies on recovery:

STEAL/NO-FORCE needs UNDO and REDO: highest performances but gives more complex logs and recovery processes (like ARIES). This is the choice made by most databases. Note: I read this fact on multiple research papers and courses but I couldn’t find it (explicitly)  on the official documentations.
STEAL/ FORCE needs only UNDO.
NO-STEAL/NO-FORCE needs only REDO.
NO-STEAL/FORCE needs nothing: worst performances and a huge amount of ram is needed.
 

The recovery part

Ok, so we have nice logs, let’s use them!

Let’s say the new intern has crashed the database (rule n°1: it’s always the intern’s fault). You restart the database and the recovery process begins.

 

ARIES recovers from a crash in three passes:

1) The Analysis pass: The recovery process reads the full transaction log* to recreate the timeline of what was happening during the crash. It determines which transactions to rollback (all the transactions without a commit order are rolled back) and which data needed to be written on disk at the time of the crash.
2) The Redo pass: This pass starts from a log record determined during analysis, and uses the REDO to update the database to the state it was before the crash.
During the redo phase, the REDO logs are processed in a chronological order (using the LSN).

For each log, the recovery process reads the LSN of the page on disk containing the data to modify.

If LSN(page_on_disk)>=LSN(log_record), it means that the data has already been written on disk before the crash (but the value was overwritten by an operation that happened after the log and before the crash) so nothing is done.

If LSN(page_on_disk)<LSN(log_record) then the page on disk is updated.

The redo is done even for the transactions that are going to be rolled back because it simplifies the recovery process (but I’m sure modern databases don’t do that).

3) The Undo pass: This pass rolls back all transactions that were incomplete at the time of the crash. The rollback starts with the last logs of each transaction and processes the UNDO logs in an anti-chronological order (using the PrevLSN of the log records).
 

During the recovery, the transaction log must be warned of the actions made by the recovery process so that the data written on disk are synchronized with what’s written in the transaction log. A solution could be to remove the log records of the transactions that are being undone but that’s very difficult. Instead, ARIES writes compensation logs in the transaction log that delete logically the log records of the transactions being removed.

When a transaction is cancelled “manually” or by the lock manager (to stop a deadlock) or just because of a network failure, then the analysis pass is not needed. Indeed, the information about what to REDO and UNDO is available in 2 in-memory tables:

a transaction table (stores the state of all current transactions)
a dirty page table (stores which data need to be written on disk).
These tables are updated by the cache manager and the transaction manager for each new transaction event. Since they are in-memory, they are destroyed when the database crashes.

The job of the analysis phase is to recreate both tables after a crash using the information in the transaction log. *To speed up the analysis pass, ARIES provides the notion of checkpoint. The idea is to write on disk from time to time the content of the transaction table and the dirty page table and the last LSN at the time of this write so that during the analysis pass, only the logs after this LSN are analyzed.

 

To conclude
Before writing this article, I knew how big the subject was and I knew it would take time to write an in-depth article about it. It turned out that I was very optimistic and I spent twice more time than expected, but I learned a lot.

If you want a good overview about databases, I recommend reading the research paper “Architecture of a Database System “. This is a good introduction on databases (110 pages) and for once it’s readable by non-CS guys. This paper helped me a lot to find a plan for this article and it’s not focused on data structures and algorithms like my article but more on the architecture concepts.

 

If you read this article carefully you should now understand how powerful a database is. Since it was a very long article, let me remind you about what we’ve seen:

an overview of the B+Tree indexes
a global overview of a database
an overview of the cost based optimization with a strong focus on join operators
an overview of the buffer pool management
an overview of the transaction management
But a database contains even more cleverness. For example, I didn’t speak about some touchy problems like:

how to manage clustered databases and global transactions
how to take a snapshot when the database is still running
how to efficiently store (and compress) data
how to manage memory
 

So, think twice when you have to choose between a buggy NoSQL database and a rock-solid relational database. Don’t get me wrong, some NoSQL databases are great. But they’re still young and answering specific problems that concern a few applications.

 

To conclude, if someone asks you how a database works, instead of running away you’ll now be able to answer:
![magic](http://coding-geek.com/wp-content/uploads/2015/08/magic_low2.gif)
Otherwise you can give him/her this article.

##总体结构

我们已经理解了数据库使用的基本组件，我们需要回头看看这个总体结构图。<br/>
数据库就是一个文件集合，而这里信息可以被方便读写和修改。通过一些文件，也可以达成相同的目的（便于读写和修改）。事实上，一些简单的数据库比如SQLite就仅仅使用了一些文件。但是SQLite是一些经过了良好设计的文件,因为它提供了以下功能：<br/>
+   使用事务能够保证数据安全和一致性
+   即使处理百万计的数据也能高效处理

一般而言，数据库的结构如下图所示:
<br/>
![database architecture](media/global_overview.png)
<br/>
在开始写之前，我曾经看过很多的书和论文，而这些资料都从自己的方式来说明数据库。所以，清不要太过关注我怎么组织数据库的结构和对这些过程的命名，因为我选择了这些来配置文章的规划。不管这些不同模块有多不一样，但是他们总体的观点是**数据库被划分为多个相互交互的模块**。<br/>

<u>核心模块:</u>
+  **进程管理器**: 很多的数据库都有进程/线程池需要管理，另外，为了达到纳秒级（切换），一些现代的数据库使用自己实现线程而不是系统线程。
+  **网络管理器**: 网络IO是一个大问题，尤其是分布式数据库。这就是一些数据库自己实现管理器的原因。
+  **文件系统管理器: 磁盘IO是数据库的第一性能瓶颈**。文件系统管理器太重要了，他要去完美使用OS文件系统，甚至自己取而代之。
+  **内存管理器**: 为了避免磁盘IO带来是惩罚，我们需要很大的内存。但是为了有效的使用这些内存，你需要一个有效率的内存管理器。尤其在多个耗内存的查询操作同时进行的时候。
+  **安全管理器**: 为了管理用户的验证和授权。
+  **客户端管理器**: 为了管理客户端连接..
+  .......

<u>工具类:</u>
+  **备份工具**: 保存和恢复一个数据库。
+  **恢复工具**: 使据据库在崩溃重启之后，重新达到一致性的状态。
+  **监控工具**: 记录数据库所有的行为，需要提供一个监控工具去监控数据库。
+  **管理工具**: 保存元数据（比如表的结构和名字），并提供工具去管理数据库，模式，表空间等等。
+  ......

<u>查询管理器:</u>
+  **查询解析器**: 确认查询是否合法
+  **查询重写器**: 优化查询的预处理
+  **查询优化器**: 优化查询语句
+  **查询执行器**: 编译执行一个查询
+  ......

<u>数据管理器:</u>
+  **事务管理器**: 管理事务
+  **缓存管理器**: 在使用数据或者修改数据之前，将数据载入到内存，
+  **数据访问**: 访问磁盘上数据


本文剩下部分，我将关注于数据库如何处理SQL查询的过程：
+  客户端管理器
+  查询管理器
+  数据管理器（我也将在这里介绍恢复管理工具）

<br/>
##客户端管理器
<br/>
![Client manager](media/client_manager.png)
<br/>
客户端管理器是处理和客户端交互的部分。一个客户端可能是（网页）服务器或者终端用户或者终端程序。客户端管理器提供不同的方法（广为人知的API: JDBC, ODBC, OLE-DB）来访问数据库。
当然它也提供数据库特有的数据库APIs。
<br/>
<br/>
当我们连接数据库：
+  管理器首先验证我们的身份（通过用户名和密码）接着确认我们是否有使用数据库的授权，这些访问授权是你们的DBA设置的。
+  接着，管理器确认是否有空闲的进程(或者线程)来处理你的这次请求。
+  管理器也要确认数据库是否过载。
+  管理器在得到请求的资源（进程/线程）的时候，当等待超时，他就关闭这个连接，并返回一个易读的出错信息。
+  得到进程/线程之后，就**把这个请求传递给查询管理器**，这次请求处理继续进行。
+  查询过程不是一个all or nothing的过程，当从查询管理器获取数据之后，就立刻将**这些不完全的结果存到内存中，并开始传送数据**。

+  当遇到失败，他就中断连接，返回给你一个**易读的说明**，并释放使用到的资源。

<br/>

##查询管理器
![Query manager](media/query_manager.png)
<br/>
**这部分是数据库的重点所在**。在本节中，一个写的不怎么好的查询请求将转化成一个**飞快**执行指令代码。接着执行这个指令代码，并返回结果给客户端管理器。这是一个多步骤的操作。
+  查询语句将被**解析**，看它是否有效。
+  接着在它之上去除无用的操作语句，并添加与处理语句，重写出来。
+  为了优化这个查询，提供查询性能，将它转化成一个可执行的数据访问计划。
+  编译这个计划。
+  最后，执行它。
<br/>
这部分，我不打算就爱那个很多在最后两点上，因为他们不是那么重要。
<br/>
<br/>

阅读完这部分之后，你将容易理解我推荐你读的这些材料：
<br/>

+  最初的基于成本优化的研究论文： ![Access Path Selection in a Relational Database Management System](http://www.cs.berkeley.edu/~brewer/cs262/3-selinger79.pdf). 
这篇文章只有12页，在计算机科学领域是一片相对易懂的论文。

+  针对DB2 9.X查询优化的非常好，非常深深入的文档![here](http://infolab.stanford.edu/~hyunjung/cs346/db2-talk.pdf)

+  针对PostgreSQL查询优化的非常好的文档![here](http://momjian.us/main/writings/pgsql/optimizer.pdf)。这是非常容易理解的文档，它更展示的是“PostgreSQL在不同场景下，使用相应的查询计划”，而不是“PostgreSQL使用的算法”。

+  SQLite关于优化的官方![SQLite documentation](https://www.sqlite.org/optoverview.html) 文档。非常容易阅读，因为SQLite使用的非常简单的规则。此外，这是为唯一一个真正解释如何使用优化规则的文档。

+  针对SQL Server 2005查询优化的非常好的文档![here](https://blogs.msdn.com/cfs-filesystemfile.ashx/__key/communityserver-components-postattachments/00-08-50-84-93/QPTalk.pdf)

+  Oracle 12c 优化白皮书 ![here](http://www.oracle.com/technetwork/database/bi-datawarehousing/twp-optimizer-with-oracledb-12c-1963236.pdf)


+  “DATABASE SYSTEM CONCEPTS”作者写的两个关于查询优化的2个理论课程![here](codex.cs.yale.edu/avi/db-book/db6/slide-dir/PPT-dir/ch12.ppt) and ![here](codex.cs.yale.edu/avi/db-book/db6/slide-dir/PPT-dir/ch13.ppt). 关注于磁盘I/O一个很好的读物，但是需要一定的计算机科学功底。

+  另一个非常易于理解的，关注于联合操作符，磁盘IO的 ![理论课](https://www.informatik.hu-berlin.de/de/forschung/gebiete/wbi/teaching/archive/sose05/dbs2/slides/09_joins.pdf)。

<br/>
##查询解析器
解析器会将每一条SQL语句检验，查看语法正确与否。如果你在SQL语句中犯了一些错误，解析器将阻止这个查询。比如你将"SELECT...."写成了"SLECT ...."，这次查询就到此为止了。
<br/>
说的深一点，他会检查关键字使用前后位置是否正确。比如阻止WHERE 在SELECT之前的查询语句。
<br/>
之后，查询语句中的表名，字段名要被解析。解析器就要使用数据库的元数据来验证：

+  **表**是否存在
+  表中**字段**是否存在
+  根据字段的类型，对字段的**操作可以**（比如你不能将数字和字符串进行比较，你不能针对数字使用substring()函数）


之后确认你是否有**权限**去读/写这些表。再次说明，DBA设置这些读写权限。
在解析过程中，SQL查询语句将被转换成一个数据库的一种内部表示(一般是树 译者注：ast)
如果一切进行顺利，之后这种表示将会传递给查询重写器
<br/>
##查询重写器
在这一步，我们已经得到了这个查询内部的表示。重写器的目的在:  

+   预先优化查询
+   去除不必要的操作
+   帮助优化器找到最佳的可行方案

<br/>
重写器执行一系列广为人知的查询规则。如果这个查询匹配了规则的模型，这个规则就要生效，同时重写这个查询。下列有几个(可选的)规则：

+  **视图合并：**如果你在查询仲使用了一个视图，这个视图将会被翻译成视图的SQL代码。
+  **子查询整理**：如果查询仲有子查询非常难以优化，冲洗器可能会去除这个查询的子查询。

例子如下：
<pre><code>
SELECT PERSON.*  
FROM PERSON  
WHERE PERSON.person_key IN  
(SELECT MAILS.person_key  
FROM MAILS  
WHERE MAILS.mail LIKE 'christophe%');  
</code></pre>
将会改写成：
<pre><code>
SELECT PERSON.*  
FROM PERSON, MAILS  
WHERE PERSON.person_key = MAILS.person_key  
and MAILS.mail LIKE 'christophe%';  
</code></pre>

+   **去除非必须操作符**： 比如如果你想让数据唯一，而使用DISTINCT的与此同时还使用一个UNIQUE约束。这样DISTINCT关键字就会被去除。
+   **消除重复连接**：如果查询中有两个一样的join条件，无效的join条件将被移除掉。造成两个一样join的原因是一次join的条件隐含在(view)视图中，也可能是因为传递性。
+   **确定的数值计算：** 如果你写的查询需要一些计算，那么这些计算将在重写过程。去个例子"WHERE AGE > 10 + 2"将会转换成 "WHERE AGE > 12"，TODATE("some date")将转化成datetime格式的日期。
+  "(高端功能)分区选择:" 如果你正在使用一个分过去的表，冲洗器会找到你要使用哪一个分区。
+  "(高端功能)实体化视图:"如果你的查询语句实体化视图

这时候，重写的查询传递给查询优化器。
好戏开场了。
<br/>
##统计
在看优化查询之前，我们必须要说一下**统计**，因为**统计是数据库的智慧之源**。如果你不告诉数据如何分析数据库自己的数据，它将不能完成或者进行非常坏的推测。
<br/>
数据库需要什么样的信息？
<br/>
我必须简要的谈一下，数据库和操作系统如何存储数据。他们使用一个称为**page**或者block(通常4K或者8K字节)的最小存储单元。这意味着如果你需要1K字节(需要存储)，将要使用一个page。如果一个页大小为8K，你会浪费其他的7K。
**注：**   

计算机内存使用的存储单元为page，文件系统的存储单元成为block    
K  -\> 1024    
4K -\> 4096    
8K -\> 8192   
<br/>
继续我们的统计话题！你需要数据库去收集统计信息，他将会计算这些信息：  

+  table中，行/page的数量
+  table中，列信息：
+  +  数据值distinct值
   +  数据值的长度(最小，最大，平均值)
   +  数据范围信息(最小，最大，平均值)

+  table的索引(indexes)信息  


**这些统计将帮助优化器去计算磁盘IO，CPU和查询使用的内存量**
<br/>
这些每一列的统计是非常重要的，比如：如果一个表 PERSON需要连接(join)两个列：LAST_ANME，RIRST_NAME。有这些统计信息，数据库就会知道RIRST_NAME只有1000个不同的值，LAST_NAME不同的值将会超过100000个。因此，数据库将会连接(join)数据使用LAST_ANME，RIRST_NAME而不是FIREST_NAME,LAST_NAME，因为LAST_NAME更少的重复，一般比较2-3个字符已经足够区别了。这样就会更少的比较。

<br/>
这只是基本的统计，你能让数据库计算**直方图**这种更高级的统计。直方图能够统计列中数据的分布情况。比如：

+  最常见的值
+  分布情况
+  .....
<br/>
这些额外的统计将能帮助数据库找到最优的查询计划。特别对等式查询计算(例：WHERE AGE = 18)或者范围查询计算(例：WEHRE AGE \> 10 and ARG \< 40)因为数据更明白这些查询计算涉及的行数（注：科技界把这种思路叫做选择性）。
<br/>
<br/>
这些统计数据存在数据的元数据。比如你能这些统计数据在这些(没有分区的)表中

+  Oracle的表USER/ALL/DBA_TABLES 和 USER/ALL/DBA_TAB_COLUMNS
+  DB2的表SYSCAT.TABLES 和 SYSCAT.COLUMNS  
 
<br/>
这些**统计信息必须时时更新**。如果出现数据库的表中有1000 000行数据而数据库只认为有500行，那就太糟糕了。统计这些数据有一个缺陷就是：**要耗费时间去计算**。这就是大多数数据库没有默认自动进行统计计算的原因。当有数以百万计的数据存在，确实很难进行计算。在这种情况下，你可以选择进行基本统计或者数据中抽样统计一些状态。
<br/>
比如：我正在进行一个计算表的行数达到亿级的统计工程，即使我只计算其中10%的数据，这也要耗费大量的时间。例子，这不是一个好的决定，因为有时候Oracle 10G在特定表特定列选择的这10%的数据统计的数据和全部100%统计的数据差别极大（一个表中有一亿行数据是很罕见的）。这就是一个错误的统计将会导致原本30s的查询却要耗费8个小时；找到导致的原因也是一个噩梦。这个例子战士了统计是多么的重要。
<br/>
<br/>
注：当然每种数据库都有他自己更高级的统计。如果你想知道更多请好好阅读这些数据库的文档。值得一提的是，我以前尝试去了解这些统计是如何用的，我发现了这个最好的官方文档 ![one from PostgreSQL](http://www.postgresql.org/docs/9.4/static/row-estimation-examples.html)
<br/>
##查询优化器
<br/>
![CBO](media/15-McDonalds_CBO.jpg)
<br/>
所有的现代数据库都使用**基于成本优化(CBO)**的优化技术去优化查询。这个方法认为每一个操作都有成本，通过最少成本的操作链得到结果的方式，找到最优的方法去减少每个查询的成本。
<br/>
<br/>
为了明白成本优化器的工作，最好的例子是"感受"一个任务背后的复杂性。这个部分我将展示3个常用方法去连接(join)两个表。我们会快速明白一个简单连接查询是多么的那一优化。之后，我们将会看到真正的优化器是如何工作的。
<br/>
我将关注这些连接查询的时间复杂度而不是数据库优化器计算他们CPU成本，磁盘IO成本和内存使用。时间复杂度和CPU成本区别是，时间复杂度是估算的（这是想我这样懒人的工具）。对于CPU成本，我还要累加每一个操作一个加法、一个if语句，一个乘法，一个迭代...
此外：

+  一个高等级代码操作代表着一系列低等CPU操作。
+  一个CPU操作的成本不是一样的（CPU周期）。不管我们使用i7,P4,amd的Operon。一言以蔽之，这个取决于CPU架构。


使用时间复杂度太简单(起码对我来说)。使用它我们能轻易明白CBO的思路。我们需要讨论一下磁盘IO，这个也是一个重要的概念。记住：**通常情况，性能瓶颈在磁盘IO而不是CPU使用**。
<br/>
<br/>
##索引
<br/>
我们讨论的索引就是我们看到的B+树。记得吗？**索引都是有序的**。说明一下，也有一些其他索引比如**bitmap 索引**，他们需要更少的成本在CPU，磁盘IO和内存，相对于B+树索引。
此外，很多现代数据库当前查询**动态创建临时索引**，如果这个技术能够为优化查询计划成本。
<br/>
##访问路径
<br/>
在执行join之前，你必须得到你的数据。这里就是你如何得到数据的方法。
注：所有访问路径的问题都是磁盘IO，我将不介绍太多时间复杂度的东西。
<br/>

**全扫描**
<br/>
如果已经看个一个执行计划，你一定看过一个词**full scan**(或者just scan)。全扫描简单的说就是数据库读整个表或者这个的索引。**对磁盘IO来说，整表扫描可是性能耗费的要比整个索引扫描多得多**。
<br/>

**范围扫描**
<br/>
还有其他的扫描方式比如**索引范围扫描**。举一个它使用的例子，我们使用一些像"WHERE AGE \> 20 AND AGE \<40"计算的时候，范围就会使用。
<br/>
当然我们在字段AGE上有索引，就会使用**索引范围扫描**。
<br/>
我们已经在第一章节看到这个范围查询的时间复杂度就是Log(N)+M，这个N就是索引数据。M就是一个范围内行的数目的估算。**因为统计N和M都是已知**（注：M就是范围计算 AGE \>20 AND AGE\<40的选择性）。
此外，对一个范围查询来说，你不需要读取整个索引，所以**在磁盘IO上，有比全扫描有更好的性能**。
<br/>

**唯一扫描**
<br/>
你只需要索引中得到一个值，我们称之为唯一扫描
<br/>

**通过rowid访问**
<br/>
在大部分时间里，数据库使用索引，数据库会查找关联到索引的行。通过rowid访问可以达到相同的目的。
<br/>
举个例子，如果你执行
>  SELECT LASTNAME, FIRSTNAME from PERSON WHERE AGE = 28  

如果你有一个索引在列age上，优化器将会使用索引为你找到所有年龄在28岁的人，数据库会查找关联的行。因为索引只有age信息，而我们想知道lastname和firstname。
<br/>
但是，如果你要做这个查询
>SELECT TYPE_PERSON.CATEGORY from PERSON ,TYPE_PERSON WHERE PERSON.AGE = TYPE_PERSON.AGE

<br/>
PERSON上的索引会用来连接TYPE\_PERSON。但是PERSON将不会通过rowid进行访问。因为我们没有获取这个表的信息。
<br/>
即使这个查询在某些访问能够工作的很好，这个查询真真正的问题是磁盘IO。如果你需要通过rowid访问太多的行，数据库可能会选择全扫描。
<br/>

**其他方法**
<br/>
我不能列举所有的访问方法。如果你需要知道的更多，你可以去看![Oracle documentation]()。名字可能和其他数据库不一样，但是背后的机制是一样的。
<br/>

**连接操作符**
<br/>
我们知道如何获取我们的数据，我们连接他们！
<br/>
我列举3个常见的连接操作：归并连接，哈希连接和嵌套循环连接。再次之前，我需要介绍几个新名词：内部关系和外部关系。一个关系（应用在）：  

+  一张表
+  一个索引
+  一个中间结果通过明确的操作（比如一个明确连接结果）

当你连接两个关系，join运算不同的方式管理两种关系。在剩下的文章里边，我假设：  

+  外部关系是左侧数据集合
+  内部关系是右侧数据集合

举例, A join B 就是一个A-B连接查询，A是外部关系，B是内部关系。
<br/>
通常，**A join B的成本和B join A的成本是不一样的**。
<br/>
**在这部分，我假设外部关系有N个元素，内部关系有M个元素**。记住，一个真正的优化器通过统计知道N和M的值。
<br/>
注：N和M都是关系的基数。
<br/>

**嵌套循环连接**
<br/>
嵌套循环连接是最简单的。
<br/>
![Nested Loop Join](media/)
<br/>
这是思路：  

+  找外部关系中的每个元素
+  你将查找内部关系的所有行，确认有没有行是匹配的。

这是伪代码
<pre><code>
nested_loop_join(array outer, array inner)
    for each row a in outer
        for each row b in inner
            if (match_join_condition(a,b))
                write_result_in_output(a,b)
            end if
        end for
   end for
</code></pre>
这就是双重循环，**时间复杂度是O(N\*M)**
<br/>
从磁盘IO来说，外部关系的N行数据每一个行，内部循环需要读取M行数据。这个算法需要读N+N\*M行数据从磁盘上。但是，如果内部关系足够小，你就能把这个关系放在内存中这样就只有M+N
次读取数据。通过这个修改，**内部关系必须是最小的那个**，因为这样这个算法，才能有最大的机会在内存操作。
<br/>
从时间复杂度来说，它没有任何区别，但是在磁盘IO上，这个是更好的读取方法对于两者。
<br/>
当然，内部关系将会使用索引，这样对磁盘IO将会更好。
<br/>
<br/>
因为这个算法是非常简单，这也是对磁盘IO更好的版本，如果内部关系能够完全存放在内存中。这就是思路：  

+  不用读取一行一行的读取数据。
+  你批量的读取数据，保持两块数据（两种关系）在内存中。
+  你比较块中的每行数据，记录匹配的行
+  从磁盘读取新块，并比较数据
+  持续执行，直到数据执行完。

这是可行的算法:
<br/>

<pre><code>
// improved version to reduce the disk I/O.
nested_loop_join_v2(file outer, file inner)
    for each bunch ba in outer
    // ba is now in memory
        for each bunch bb in inner
        // bb is now in memory
            for each row a in ba
                for each row b in bb
                    if (match_join_condition(a,b))
                        write_result_in_output(a,b)
                    end if
                end for
            end for
        end for
    end for
</code></pre>

<br/>
**这个版本，时间复杂度是一样的，磁盘访问数据降低**：

+  前一个版本，这个算法需要N + N\*M 次访问（一次读一行）
+  新版本中，磁盘访问次数成了number_of_bunches_for(outer)+ number_of_ bunches_for(outer)\* number_of_ bunches_for(inner)。
+  如果你增加每个块的数量，就减少了磁盘访问次数。


注：比起前一个算法，一个数据库访问收集越多的数据。如果是顺序访问还不重要。（机械磁盘的真正问题是第一次获取数据的时间。）
<br/>
<br/>
**哈希连接**
<br/>
相对于嵌套循环连接，哈希连接更加复杂，但是有更好的性能，在很多情况下。
![Hash Join](media/hash_join.png)
<br/>
哈希连接的思路为：  

+  1）获取所有的内部关系的所有元素。
+  2)创建一个内存hash表
+  3)一个一个的获取所有的外部关系元素。
+  4)针对每个内部关系元素计算每个元素的哈希值(通过哈希函数)找到关联域。
+  5)找到外部表格元素匹配的关联域的元素。
<br/>
从时间复杂度来说，我必须先做一些假设来简化问题：
+  内部关系元素被分割到X个域中。
+  哈希方法分布的哈希范围对于两个关系是一致的。换而言之，域的大小是一样的。
+  匹配外部关系的一个元素和域中所有元素的成本为域中元素的个数。

<br/>
时间复杂度是(M/X)\*N +cost_to_create_hash_table(M) + cost_of_hash_function\*N
<br/>
如果哈希函数创建足够小的域，这个复杂度为**时间复杂度为O(M+N)**
<br/>
<br/>
这就是另一个版本的哈希连接，它更多的内存，和更少的磁盘IO。 

+  1)你计算出内部关系哈希表和外部关系的哈希表
+  2)然后把他们放在磁盘上。
+  3)然后你就可以一个一个比较两个哈希表的域(一个完全载入内存，一个是一行一行的读)。

<br/>
**归并连接**
<br/>
**归并连接是唯一产生有序结果的连接**
<br/>
注：在这个简化的归并连接，没有内部表和外部表的区别。他们是同样的角色。但是实际实现中又一些区别。比如：处理赋值的时候。
<br/>
归并连接可以分为两个步骤：
<br/>  

1. (可选项)排序操作:两个输入项都是在连接键上已经排好序。  
2. 归并连接操作：将排序好序的两个输入项合并在一起。

<u>排序</u>
<br>
我们已经说过了归并排序，从这里来说，归并排序是一个好的算法(如果有足够内存，还有性能更好的算法)。
<br/>
但是有时数据集已经是排好序的。比如：

+  如果表是自然排序的，比如一个在连接键上使用了索引的表。
+  如果关系就是连接条件的索引
+  连接操作要使用查询过程中的一个已经排好序的中间结果。

<br/>

<u>归并连接</u>
<br/>
![merge join](media/merge_join.png)
<br/>
这一部分比起归并排序简单多了。但是这次，不需要挑选每一个元素，我只需要挑选两者相等的元素。思路如下：  

+  1)如果你比较当前的两个关系的元素(第一次比较，当前元素就是第一个元素)。
+  2)如果他们相等，你就把两个元素放入结果集中，然后获取两个关系的下一个元素。
+  3)如果不相等，你就获取较小元素的关系下一个元素(因为下一个元素较大，他们可能会相等)。
+  4)重复 1，2，3步。一直到已经其中一个关系已经比较了全部的元素。


这样执行是因为两个关系都是排好序的，你不需要回头找元素。
<br/>
这个算法是简化之后的算法。因为它没有处理两个关系中都会出现多个相同值的情况。实际的版本就是在这个情况上变得复杂了。这也是我选了一个简化的版本。
<br/>
<br/>
在两个关系都是排好序的情况下，**时间复杂度为O(M+N)**
两个关系都需要排序的情况下时间复杂度加上排序的消耗 **O(N\*Log(N) + M\*Log(M))**
<br/>
对于专注于计算机的极客，这是一个处理多个匹配算法（注：这个算法我不能确定是100%正确的）。
<pre><code>
mergeJoin(relation a, relation b)
    relation output
    integer a_key:=0;
    integer b_key:=0;
    
    while (a[a_key]!=null and b[b_key]!=null)
        if (a[a_key] < b[b_key]) a_key++; else if (a[a_key] > b[b_key])
            b_key++;
        else //Join predicate satisfied
            write_result_in_output(a[a_key],b[b_key])
            //We need to be careful when we increase the pointers
            integer a_key_temp:=a_key;
            integer b_key_temp:=b_key;
            if (a[a_key+1] != b[b_key])
                b_key_temp:= b_key + 1;
            end if
            if (b[b_key+1] != a[a_key])
                a_key_temp:= a_key + 1;
            end if
            if (b[b_key+1] == a[a_key] && b[b_key] == a[a_key+1])
                a_key_temp:= a_key + 1;
                b_key_temp:= b_key + 1;
            end if
            a_key:= a_key_temp;
            b_key:= b_key_temp;
        end if
    end while
</code></pre>
<br/>
**哪一个是最好的连接算法**
<br/>
如果有一个最好的连接算法，那么它不会有这么多种连接算法。选择出一个最好的连接算法，太困难。他有那么评判标准：
<br/>
+  **内存消耗：**没有足够内存，你一定确定以及肯定用不了强大的哈希连接（起码也是全内存的哈希连接）。
+  **两个数据集合的大小**。如果你有一个很大的表和一个很小的表做连接查询，嵌套循环连接要比哈希连接还要快。因为哈希连接在建立哈希表的时候，消耗太大。如果你有两个很大的表，嵌套循环连接就会耗死你的CPU。
+  **索引的存在**。有了两个B+树的索引，归并连接就是显而易见的选择。
+  **要求结果排序**；如果你要连接两个无序的数据集，你想使用一个非常耗费性能的归并连接因为你需要结果有序，之后，你就可以拿着这个结果和其他的（表）进行归并联合。(或者是查询任务要求一个有序结果通过order by / group by / distinct操作符)
+  **两个排好序的关系**：归并排序，归并排序，归并排序。
+  你使用的连接的种类：是**相等连接**（例子： tableA.col1 = tableB.col2）？内连接？外连接？笛卡尔乘积？自连接？在某些情况下，连接也是无效的。
+  你想让**多进程/多线程**来执行连接操作。
<br/>
<br/>
更多内容，请看![DB2](https://www-01.ibm.com/support/knowledgecenter/SSEPGG_9.7.0/com.ibm.db2.luw.admin.perf.doc/doc/c0005311.html),![ORACLE](http://docs.oracle.com/cd/B28359_01/server.111/b28274/optimops.htm#i76330),![SQL Server](https://technet.microsoft.com/en-us/library/ms191426%28v=sql.105%29.aspx)的文档。
<br/>
<br/>
**例子**
<br/>
我们已经见过了3种连接操作。
<br/>
现在如果你需要看到一个人的全部信息，要连接5张表。一个人可能有：
多个手机电话
多个邮箱
多个地址
多个银行账户
<br/>
总而言之，这么多的信息，需要一个这样的查询：
<br/>
<pre><code>
SELECT * from PERSON, MOBILES, MAILS,ADRESSES, BANK_ACCOUNTS
WHERE
SPERSON.PERSON_ID = MOBILES.PERSON_ID
SAND PERSON.PERSON_ID = MAILS.PERSON_ID
SAND PERSON.PERSON_ID = ADRESSES.PERSON_ID
SAND PERSON.PERSON_ID = BANK_ACCOUNTS.PERSON_ID
</code></pre>
<br/>
如果一个查询优化器，我得找到最好的方法处理这些数据。这里就有一个问题：

+  我该选择那种连接查询？
<br/>
我有三种备选的连接查询（哈希，归并，嵌套循环），因为他们能够使用0，1，2个索引（先不提有不同的索引）。
<br/>
+  我们选择表来做连接查询的顺序？
<br/>
举个例子，下图展示了4个表上的3次连接操作的可行的执行计划：
<br/>
![Join Ordering Problem](media/join_ordering_problem.png)
<br/>
我可能会这么做：
+  1）我用了暴力破解的方法
<br/>
通过数据库的统计，我可以**计算每一个执行计划的成本**之后，选择那个最优解。但是有太多的可行方法了。就给定的连接查询的顺序而言，每一次连接有三种选择，哈希连接，归并连接，嵌套连接。所以对于确定顺序的连接就有3的4次方的方法。连接的顺序是一个**二叉树置换问题**，它有(2\*4)!/(4+1)!种可行方法。在这个问题上，我们有34\*(2\*4)!/(4+1)!种方法。
<br/>
更直观的数字是，27216个方法。如果我把使用了0，1，2个索引的可能性增加到这个问题上，这个数字了21000种。看到这个简单查询，傻眼不？
+  2）把我搞哭了，不干这个事儿了。
<br/>
这个提议很吸引人。但是你得不到结果，我还指望它挣钱呢。
+  3）我就找几个执行计划试试，用其中最好性能的那个。  
我不是超人，我可算不出来每一个执行计划的成本。于是，我就**从所有可能的执行计划中随意选了一些**，计算他们的成本，给你其中性能最好的哪个。
+  4）我使用了**更聪明的规则减少了可行的执行计划**。

<br/>
这里就有2种规则：
<br/>
逻辑：我可以删除没有用的可能，但是不能过滤很多的可能。
比如：使用嵌套循环连接的内部关系一定是最小的数据集。
<br/>
我可以接受不是最优解。使用更加有约束性的条件，减少更多的可行方法。比如：如果一个关系很小，使用嵌套循环查询，而不是归并、哈希查询。
<br/>
<br/>
在这个简单例子中，我得到了那么多的可行方法。但是**一个现实的查询还有其他的关系操作符**，像 OUTER JOIN, CROSS JOIN, GROUP BY, ORDER BY, PROJECTION, UNION, INTERSECT, DISTINCT …这意味着更多更多的可行方法。
<br/>
这个数据库是怎么做的呢？
<br/>
<br/>
##动态规划，贪婪算法和启发式算法
<br/>
我已经提到一个数据库要尝试很多种方法。真正的优化就是在一定时间内找到一个好的解。
<br/>
**大多数情况下，优化器找到的是一个次优解，找不到最优解**。
<br/>
小一点的查询，暴力破解的方式也是可行的。但是有一种方法避免了很多的重复计算。这个算法就是动态规划。
<br/>
**动态规划**
<br/>
动态规划的着眼点是有不同的执行计划的有些步骤是一样的。如果你看这些下边的这些执行计划：
<br/>
![overlapping trees](media/overlapping_trees.png)
<br/>
他们使用了相同的子树(A JOIN B)，所以每一个执行计划都会计算这个操作。在这儿，我们计算一次，保存这个结果，等到重新计算它的时候，就可以直接用这个结果。更加正式的说，我们遇到一些有部分重复计算的问题，为了避免额外的计算，我们用内存保存重复计算的值。
<br/>
使用这个技术，我们仅仅有了3^N的时间复杂度，而不是(2\*N)!/(N+1)!。在上个例子中的4个连接操作，通过使用动态规划，备选计划从336减少到81。如果我们使用一个更大的嗯 **8连接的查询，就会从57657个选择减少到6561**。
<br/>
对于玩计算机的极客们，我之前看到了一个算法，在这个![课上](codex.cs.yale.edu/avi/db-book/db6/slide-dir/PPT-dir/ch13.ppt)。提醒：我不准备在这里具体解释这个算法，如果你已经了解了动态规划或者你很擅长算法。
<pre><code>
procedure findbestplan(S)
   if (bestplan[S].cost infinite)
       return bestplan[S]
    // else bestplan[S] has not been computed earlier, compute it now
   if (S contains only 1 relation)
         set bestplan[S].plan and bestplan[S].cost based on the best way
         of accessing S  /* Using selections on S and indices on S */
   else for each non-empty subset S1 of S such that S1 != S
   P1= findbestplan(S1)
   P2= findbestplan(S - S1)
   A = best algorithm for joining results of P1 and P2
   cost = P1.cost + P2.cost + cost of A
   if cost < bestplan[S].cost
       bestplan[S].cost = cost
       bestplan[S].plan = “execute P1.plan; execute P2.plan;
                 join results of P1 and P2 using A”
    return bestplan[S]
</code></pre>
<br/>
对于更大的查询，我们不但使用动态规划，还要更多的规则（或者**启发式算法**）去减少无用解：

+  比如我们在分析一个执行计划（下例：left-deep tree）我们就从3^n减少到了 N\* 2^n
<br/>
![left-deep-tree](media/left-deep-tree.png)
<br/>
+  我们增加一些逻辑条件，减少某些情况下下的计划。（比如：在给定一个表有给定条件需要的索引，就不再尝试归并连接，直接使用索引）他也能减少很多的情况，而损害最后得到最好的结果。
+  如果我们在过程中增加一些条件(比如：执行连接操作之前，执行其他的关系查询)他也能减少很多的可能情况。
+  .....


**贪婪算法**
<br/>
对于一个非常大规模的请求但是需要极其快速获得答案（这个查询并不是很快速的查询），要使用的就是另一种类型的算法，贪婪算法。
<br/>
这个思路是根据一个准则（或者说是**启发式**）逐步的去创建执行计划。通过这个准则，贪婪算法一次只得到一个步骤的最优解。  
贪婪算法从一个JOIN来开始一次执行计划，找到这个JOIN查询的最优解。之后，找到每个步骤JOIN的最优解，然后增加到执行计划。
<br/>
<br/>
让我们开始这个简单的例子。比如：我们有一个查询，有5张表的4次join操作(A, B, C, D, E)。为了简化这个问题，我们使用嵌套循环连接。我们使用这个准则：**使用最低成本的JOIN**。  

+  随意选择一张表（选择A）
+  我们计算每一个表JOIN A的成本。(A是外连接的内部关系)
+  我们得到了 A JOIN B是性能最好的。（A JOIN B结果为AB）
+  我们计算每一张表 JOIN AB的成本。（AB在这个计算中作为外连接的内关系）
+  我们得到 AB JOIN C 是性能最好的。（AB JOIN C 结果为ABC）
+  我们计算剩下的每张表 JOIN ABC的成本.....
+  .......
+   最后，我们找到了这个结果的存续为(((A JOIN B) JOIN C) JOIN D ) JOIN E  


因为我是随意的从A开始，当然我们也可以指定从B，或者C，D，E 开始我们的算法。我们也是通过这个过程得到性能最好的执行计划。
<br/>
这个算法有一个名字，叫做![最邻近节点算法](https://en.wikipedia.org/wiki/Nearest_neighbour_algorithm)。
<br/>
我不深入的讲解算法的细节了。在一个良好的设计模型情况下，达到N\*log(N)时间复杂度，这个问题将会被很好的解决。**这个算法的时间复杂度是O(N\*log(N))，对于全部动态计算的算法为O(3^N)**。如果你有一个达到20个join的查询，这就意味着26 vs 3 486 784 401，这是一个天上地下的差别。
<br/>
这个算法的问题在于我们假设我们在两张表中已经使用了最好的连接方法的选择，通过这个连接方式，已经给我们最好的连接成本。但是：

+  如果A JOIN B拥有最好的效率在A， B ，C的连接的第一个步骤中。
+  (A JOIN C) JOIN B这个连接可能拥有比(A JOIN B)JOIN C更好的性能。


为了优化性能，你可以运行多个贪婪算法使用不同的规则，最后选择最好的执行计划。
<br/>
**其他算法**
<br/>
[如果你已经充分了解了这些算法，你就可以跳跃过下一部分。我想说的是：它不会影响剩下的文章的阅读]
<br/>
对于很多计算机研究学者而言，获得最好的执行计划是一个非常活跃的研究领域。他们经常尝试在更加实用的场景和问题上，找到更好的解决方案。比如：

+  如果是一个star join(一种特定类型的多连接查询)，数据库将会使用一个特定的算法。
+  如果是一个并行的查询，一些数据库会使用特定的算法。
+  ....

注：
star join不太确定是什么连接查询。
<br/>
这些算法在大规模的查询的情况下，可以用来取代动态规划。贪婪算法属于**启发式算法**这一大类。贪婪算法是遵从一个规则(思路),在找到当前步骤的解决方法，并将之前步骤的解决方法结合在一起。一些算法遵从这个规则，一步一步的执行，但并不一定使用之前步骤的最优解。这个叫做启发式算法。
<br/>
比如，**遗传算法**遵从这样的规则--最后一步的最优解通常是不保留的：

+  一个解决方案是可能的执行计划的全部步骤
+  每一步保存P个方案(或计划)，而不是一个方案(或 计划)
+  0) P个计划是随机创建的
+  1) 只有最优的计划才能被保留
+  2) 混合计算这些计划，然后产生P个新的计划
+  3) P个计划中的一些会被随机的修改
+  4) 然后将步骤1，2，3重复执行T次。
+  5) 在最后一次循环中从P个计划中，保留最好计划。

<br/>
越多的循环次数，会得到更好的执行计划的。
<br/>
这是魔术吗？不，这是自然的规则：只有最适合的才会存在。
<br/>
另外，![PostgreSQL](http://www.postgresql.org/docs/9.4/static/geqo-intro.html)已经实现了遗传算法，但是我还不了解是否默认使用了这个算法。
<br/>
数据库使用了其他的启发式算法，比如Annealing, Iterative Improvement, Two-Phase Optimization… 但是我不了解这些算法是否用在企业数据库中，或者还在处于数据库研究的状态上。
<br/>
**实际的优化器**
<br/>
[可以跳过这一章，这一章对于本文不重要，也不影响阅读]  
我已经说了这么多，而且这么理论，但是我是一个开发人员，不是一个研究人员。我更喜欢**实实在在的例子**
<br/>
让我看一下 ![SQLite 优化器](https://www.sqlite.org/optoverview.html)是如何工作的。SQLite是一个轻量级的数据库，它使用了非常简单优化方法，具体是基于贪婪算法，添加额外的约束，以减少可选解数量。

+  SQLite在CROSS JOIN时候，不对表进行重新排序。
+  **实现joins都是嵌套循环连接**
+  outer joins 通常是按照顺序计算
+  .....
+  在3.8.0版本之前，**SQLite计算最优的查询计划时候，使用"最近邻节点"贪婪算法**
<br/>
等一下....我们已经知道这个算法那了！很巧合啊。
+  从3.8.0版本以后(2015年发布)，sqlite 在查找最优解的时候使用了![N Nearest Neighbors](https://www.sqlite.org/queryplanner-ng.html)+**贪婪算法**

<br/>
好吧，让我了解一下其他优化怎么来工作。IBM DB2像其他的企业级数据库一样，它是我关注大数据之前专注的最后一个数据库。
<br/>
如果我们查看了DB2的官方文档，我们会了解到DB2的优化器有7个优化层次。

+  在joins操作时候，使用贪婪算法
+  +  0 - 最少的优化，仅仅使用索引扫描和嵌套循环连接，避免查询重写。
+  +  1 - 低层次优化
+  +  2 - 全优化
+  使用动态规划来计算连接方案
+  +  3 - 保守的优化和粗略的邻近估计
+  +  5 - 全优化，使用启发性算法的所有技术。
+  +  7 - 类似于第5个层次，不使用启发性算法
+  +  9 - 不遗余力的最大的优化**考虑所有的可能连接顺序，包括笛卡尔乘积**

<br/>
我们可以了解到**DB2使用了贪婪算法**。当然，自从查询优化器成为数据库的一大动力的时候，他们不再分享他们使用的启发式算法。
<br/>
说一句，**默认的优化层次是5**。缺省情况下，优化器使用以下数据：

+  **所有有效的统计**，包括使用常用值(frequent-value)和分位数统计信息
+  实施**所有查询重写规则**（包括具体化查询表的路由选择），不过一些计算密集型的规则只有在很少的情况才会被使用。
+  使用**动态规划的连接列举**，当然有一些限制使用的地方：
+  +  合成的内部关系
+  +  星型模式下笛卡尔乘积，包括了查找表。
+  考虑很多的访问方法，包括列表预读，index anding(注：一个作用于indexes的特殊操作)和具体化的表的路由选择。

<br/>
默认情况下， **DB2 在选择连接顺序时候，使用启发算法约束的动态规划**
<br/>
其他的SQL选择条件可以使用简单的规则
<br/>
**查询计划缓存**
<br/>
创建一个执行计划是需要时间的，大多数的数据库都把这些查询计划存储在**查询计划缓存**中，减少重新计算这些相同的查询计划的消耗。只是一个非常大的课题，因为数据库必须知道什么时候替换掉已经过期无用的计划。这个方法是创建一个阀值，当统计信息中表结构产生了变化，高于这个阀值，数据库就必须将涉及这张表的查询计划删除，净化缓存。
<br/>
##查询执行器
<br/>
辛辛苦苦到了这个环节，我们已经获得优化过的执行计划。这个计划已经是编译成了可执行代码。如果有了足够的资源(内存，CPU)，查询执行器就会执行这个计划。计划中的操作(JOIN, SORT BY....)可以顺序执行，也可以并行执行，全看执行器。为了获取和写入数据，执行器必须和数据管理器打交道，也就是下一节的内容。
<br/>
# 数据管理器
<br/>
![数据管理器](media/data_manager.png)
<br/>
到了这一步，查询管理器执行查询，需要从表，索引获取数据。它从数据管理器请求数据，有2个问题：

+  关系型数据库使用事务模型。所以，我们不能想什么时候获取，就能马上获取数据滴。因为这个时候，其他人可能正在使用或者修改我们要求的数据。
+  **获取数据是数据库最慢的操作**，因为数据管理器需要足够的智慧去获取数据并在内存中保存数据。

在这个部分，我们将会看到关系型数据库如何处理这2个问题。我们不会讨论管理器如何获取数据，因为这个不是非常重要（本文现在也太长了）。
<br/>
##缓存管理器
<br/>
我已经说过，数据库最大的瓶颈就是磁盘I/O。为了提高性能，现代数据库使用缓存管理器。
<br/>
![缓存管理器](media/cache_manager.png)
<br/>
相比于直接从文件系统获取数据，查询执行器从缓存管理器中请求数据。缓存管理器有一个常驻内存的缓存叫做**内存池**，直接从内存获取数据能够极大极大的提高数据库的性能。但是很难说明性能提升的量级，因为这个跟你的执行操作息息相关。

+  顺序读取(比如：全扫描)和随机读取(比如：通过rowid访问)
+  读和写
<br/>
也跟数据库服务器使用的磁盘类型关系很大
<br/>
+  7200转/1W转/1W5转 机械硬盘
+  SSD
+  RAID 1/5/....


但是**内存要比磁盘操作快100到10W倍**
<br/>
这个速度的差异引出了另一个问题（数据库也有这个问题）。缓存管理器需要在查询执行器使用之前加载数据到内存，不然查询管理器必须等待从慢腾腾磁盘上获取数据。
<br/>
##预加载
<br/>
这个问题叫做预加载，查询执行器知道它需要那些数据。因为它已经知道了查询的整个流程，通过统计数据已经了解磁盘上的数据。这里有一个加载思路：  

+  当查询执行器处理它的第一组数据
+  它通知缓存管理器预先加载第二组数据
+  当他开始处理第二组数据
+  它继续通知缓存管理器预先加载第三组数据，并通知缓存管理器从缓存中清除第一组数据。


缓存管理器在内存池中存储所有这些数据。为了确定数据需要与否，管理器附加缓存中的数据额外的管理信息(称为latch)。
注：latch真的没法翻译了。
<br/>
有时，查询执行器不知道他需要什么样的数据，因为数据库并不提供这项功能。相应的，数据使用推测性预加载(比如：如果查询执行器要求数据1,3,5，他们他很可能继续请求7,9,11)或者连续预加载(这个例子中，缓存管理器从磁盘上顺序加载数据，根据执行器的请求)
<br/>
为了显示预加载的工作情况，现代数据提供一个衡量参数叫做**buffer/cache hit ratio**请求数据在缓存中概率。
<br/>
注：很低的缓存命中率不意味着缓存工作的不好。更多的信息可以参考![Oracle文档](http://docs.oracle.com/database/121/TGDBA/tune_buffer_cache.htm#TGDBA294)。
<br/>
如果缓存是一个非常少的内存。那么，他就需要清除一部分数据，才能加载新的数据。数据的加载和清除，都需要消耗成本在磁盘和网络I/O上。如果一个查询经常执行，频繁的加载/清除数据对于这个查询也不是非常有效率的。为了解决这个问题，现代数据库使用内存更新策略。
<br/>
**内存更新策略**
<br/>
现在数据库(SQL Server，MySQL，Oracle和DB2)使用LRU算法。
<br/>
**LUR**
<br/>
**LRU**全称是Least Recently Used，近期最少使用算法。算法思路是最近使用的数据应该驻留在缓存，因为他们是最有可能继续使用的数据。
<br/>
这是可视化的例子：
<br/>
![LRU](MEDIA/LRU.png)
<br/>
综合考虑，我们假设缓存中的数据没有被latches锁定（这样可以被清除）。这个简单的例子中，缓存可以存储3个元素：

+  1：缓存管理器使用数据 1，将这个数据放入空的内存
+  2：管理器使用数据 4，将这个数据放入半加载的内存
+  3：管理器使用数据 3，把这个数据放入半加载的内存
+  4：管理器使用数据 9，内存已经满了，**数据 1就要被移除因为他是最久没用的数据。**数据 9放入内存。
+  5： 管理器使用数据 4，**数据4已经在内存了，因为4重新成为最近使用的数据**
+  6： 管理器使用数据 1，内存已经满了，**数据9被移除，因为它是最久没用的数据**，数据1 放入内存。
+   ....

这个算法工作的很好，但是有一些限制。如果在一个大表上进行全扫描，怎么办？话句话说，如果表/索引的大小超过的内存的大小，怎么办？使用这个算法就会移除缓存中之前的所有数据，然而全扫描的数据只使用一次。
<br/>
**增强方法**
<br/>
为了避免这个这个问题，一些数据增加了一些规则。比如 Oracle请看![Oracle文档](http://docs.oracle.com/database/121/CNCPT/memory.htm#CNCPT1222)
>  “For very large tables, the database typically uses a direct path read, which loads blocks directly […], to avoid populating the buffer cache. For medium size tables, the database may use a direct read or a cache read. If it decides to use a cache read, then the database places the blocks at the end of the LRU list to prevent the scan from effectively cleaning out the buffer cache.”
(对于表非常大的情况，数据典型处理方法，直接地址访问，直接加载磁盘的数据块，减少填充缓存的环节。对于中型的表，数据块使用直接读地盘，或者读缓存。如果选择了读缓存，数据库将数据块放在LRU列表的最后，防止扫描将这个数据块清除)
<br/>
现在有更多的选择了，比如LRU的新版本，叫做LRU-K，在SQL-Server中使用了LRU-K，K=2.
<br/>
算法的思路是记录更多的历史信息。对于简单的LRU(也可以认为是LRU-K，K=1),算法仅仅记录了最后一个数据使用的信息。对于LRU-K

+  记录**K次数据的使用信息**
+  **权重**是基于数据使用次数。
+  如果一组新数据载入缓存，最常用的老数据不会被移除(因为它的权重更高)。
+  但是算法不会保留不再被使用的老数据。
+  所以**不使用数据的情况下，权重根据时间衰减**。

权重的计算是非常耗费成本的，所以SQL-Server仅仅使用了K=2.总体来看，使用这个值，运行情况也是可以接受的。
<br/>
关于LRU-K，更多更深入的信息信息，你可以阅读研究文档(1993):![数据库缓存的LRU-K页面替换算法](http://www.cs.cmu.edu/~christos/courses/721-resources/p297-o_neil.pdf)
<br/>
**其他算法**
<br/>
当然，管理缓存许多其他的算法比如：

+  2Q（类似于LRU-K算法）
+  CLOCK（类似于LRU-K算法）
+  MRU（最近最多使用算法，使用和LRU相同逻辑，使用规则不同）
+  LRFU（最近最少，最频繁使用算法）
+  .......


一些数据库提供了使用其他算法而不是默认算法的方法。
<br/>
**写缓存**
<br/>
我仅仅讨论了读缓存--使用数据之前，载入数据。但是数据库中，你还必须有写缓存，这样你可以一次批量的写入磁盘。而不是写一次数据就写一次磁盘，减少单次的磁盘访问。
<br/>
记住缓存保存**数据页**（Pages，数据的最小单元）而不是行（这是人/逻辑看待数据的方式）。如果一个页被修改而没有写入磁盘，那么缓存池中的这个数据页是**脏**的。选择写入磁盘时间的算法有很多，但是他们都和事务的概念息息相关。这就是本文下一章要讲述的。
<br/>
##事务管理器
<br/>
本文最后，也就是本章内容就是事务管理器。我们将看到一个进程如何保证每一个查询都是在自己的事务中执行的。但是在此之前，我们必须了解事务 ACID的概念。
<br/>
I‘m on acid
<br/>
事务是关系型数据库的**工作单元**，它有四个性质：

+  原子性(Atomicity)：事务是要么全做，要么不做，即便它执行了10个小时。如果事务执行失败，数据库的状态将回到事务执行之前的状态（事务回滚）。
+  隔离性(Isolation)：如果事务A和B同时执行，那么不管事务A还是事务B先完成，结果总是一样的。
+  持久性(Durability)：一旦事务已经**提交**（成功结束）,数据将存在数据库中，不管任何事情发生（崩溃或者错误）。
+  一致性(Consistency)：只有有效的数据（依照关系约束和功能约束）写入数据库。一致性和原子性、隔离性有一些联系。


![一美元](media/dollar_low.jpg)
<br/>
在一个事务里边，你可以有很多SQL语句去对数据增删改查。混乱的开始：两个事务同时使用相同的数据。典型例子：一个转账从账户A到账户B。想象一下，你有两个事务：

+  事务1 从账户A转走100美元到账户B
+  事务2 从账户A转走50美元到账户B


如果我们回到事务的ACID性质：

+  **原子性**确保不管在T1过程中发生什么(系统崩溃，网络异常)，你都不会造成A转走了100$,却没有给B。（这个就是不一致的状态）
+  **隔离性**确保即使T1和T2同时执行，最终结果就是A将会转走150$,B得到150$。而不是其他的状态。比如：A转走了150$，B得到了只有50$因为T2抹去了一部分T1的行为。(这个情况也是不一致的状态)。
+  **持久性**确保在T1提交之后，即使数据库崩溃，T1也不会凭空消失。
+  **一致性**确保没有钱在系统中多了或者少了。
<br/>
[如果你想，你可以略过本章剩下的内容，它对本文并不是很重要]
<br/>
很多现代的数据库默认并没有使用完全的隔离，因为他会带来巨大的性能问题。正常情况下，SQL规范定义了4个层次的隔离：
+  **串行化**（SQLite的默认选择）：最高级别的隔离。同时发生的两个事务是100%的隔离的。每一个事务都有自己的"世界"。
+  **可重复读**（MySQL的默认选择）：每一个事务都有自己的"世界"，除了一种情况。如果一个事务正确结束，那么它增加的新数据，对于其他正在运行的事务是可见的。如果更新数据的事务正常结束，这个修改将对其他正在运行的事务，是不可见的。和其他事务的的隔离的不同，在新增数据，而不是已经存在的数据(修改的数据)。

比如，如果一个事务 A执行"select count(1) from TABEL_X"，在这个时候，事务B向TABEL_X中，增加了一条新的数据，并正确提交，如果事务A 重新执行count(1)，获得的值是不同的。
<br/>
这叫做**幻读**。

+  **读已提交**（Oracle，PostgreSQL，SQL Server的默认选择）：这是这个是重复读+一个新的不同的隔离。如果事务A 读数据D，而这个时候，数据D被事务B修改(或者删除)，并且已经正确提交。如果A重新读数据D，那么他就可以看到在这个数据上的修改(或者删除)。  
+  **读未提交**：这个事最低层次的隔离。这个就是读已提交+新的不同的隔离。如果事务A读了数据D，那么数据D被事务B（正在运行还未提交）修改。如果这个时候A重新读数据D，他将会看到这个修改后的结果。如果事务B回滚，这个时候A第二次读数据D，被事务B修改的数据就像没有被修改的一样，没有任何理由。(因为事务B回滚了)

这个叫做**脏读**。
<br/>
<br/>
大部分数据使用它自己独特的隔离级别(就像Post供热SQL， Oracle，SQL Server使用的快照隔离)。然后，更多数据通常并不是全部的SQL规范的四个隔离，尤其是读未提交。
<br/>
默认的隔离级别可以在数据库连接开始的时候(仅仅需要添加非常简单的代码)，被用户和开发者重新定义。
<br/>
##同步控制
<br/>
隔离，一致性和原子性的真正问题是**在相同数据上的写操作**（增加，修改和删除）：

+  如果所有的事务都是只读数据，在没有其他事务修改数据的情况下，他们都可以同时运行。
+  如果事务中的一个(起码)，正在修改一个数据，这个数据要被其他的事务读取，数据库需要一种方法，对其他的事务，隐藏这个修改。另外，也需要保证这个修改不会被其他的事务消除，因为其他事务看不到这份修改的数据。

<br/>
这个问题叫做**并行控制**
<br/>
解决这个问题最简单的方式是一个接一个执行事务(比如：串行)。但是它不能进行扩展，即使运行在多核多处理的服务器上也只能使用一个核。非常没有效率...  
解决这个问题的理想方式是：在每一时刻，事务都可以常见或者取消：

+  监控所有事务的的所有操作。
+  检测2个(或者多个)事务在读/修改相同数据的时候，是否冲突。
+  修改冲突的事务的操作的执行顺序，减少冲突的部分的范围。
+  记录可以被取消的事务。

<br/>
更加正式的说，这是一个冲突调度时候的再调度问题。更加具体的说，这个是一个困难的，CPU密集型的优化问题。企业型数据库肯定不能耗费数小时去给每个事务去找到到最好的调度方式。因此，他们使用次于理想方式的途径，这些方式导致处理冲突的事务更多的时间浪费。
<br/>
##锁管理器
<br/>
为了解决这个问题，大部分数据库使用**锁**并且/或者**数据版本**。这个是一个大的话题，我将关注于锁。之后我会讲解写数据版本。
<br/>
**悲观锁**
<br/>
锁机制的思路是：

+  如果事务需要数据
+  它锁定数据
+  如果其他事务也需要这个数据
+  它等待第一个事务释放这个数据的锁。


这就被称为**独占锁**
<br/>
但是事务在只需要读数据的时候，却使用独占锁就太浪费了。因为**它强制其他事务在读相同的数据的时候也必须等待**。这就是为什么需要另一种锁，**共享锁**。
<br/>
共享锁的思路：

+  如果事务只需要读数据A，
+  它对数据A加“共享锁”，然后读数据
+  如果第二个事务也只需要读数据A，
+  它对数据A加”共享锁“，然后读数据
+  如果第三个事务需要修改数据A，
+  它对数据加”独占锁“，但是他必须等待，一直等到2个其他事物释放他们的共享锁，才能实施它的独占锁。


另外，如果数据被施加独占锁，一个事务只需要读这个数据，也必须等待独占锁的结束，然后对数据加共享锁。
<br/>
![锁管理器](media/lock_manager.png)
<br/>
锁管理器是加锁和解锁的过程。从实现上来说，它在哈希表中存储着锁(键值是要锁的数据)，以及对应的数据。

+  那些事务正在锁定数据
+  那些事务正在等待数据

<br/>
**死锁**
<br/>
但是锁的使用会导致一个问题，两个事务在永远的等待对方锁定的数据。
<br/>
![死锁](media/deadlock.png)
<br/>
在这个例子中：

+  事务A在数据data1上加了独占锁，并等待数据data2
+  事务B在数据data2上加了独占锁，并等待数据data1

<br/>
这就是**死锁**
<br/>
在死锁中，锁管理器为了避免死锁，会选择取消(回滚)其中一个事务。这个选择不容易的：

+  取消修改最少数据的事务是不是更好（这样产生了最好性能的回滚）？
+  取消已经执行时间最少的事务是不是更好，因为其他事务已经等了更久？
+  取消总体执行时间最少的事务（能够避免可能的饥饿问题）。
+  当出现回滚的时候，多少个事务别这个回滚影响？

<br/>
在我们做出选择之前，必须确定是否存在死锁。
<br/>
这个哈希表，可以被看做是图(像之前的例子)。如果在图中产生了一个循环，就有一个死锁。因为确认环太浪费性能（因为有环的图一般都非常大），这里有一个常用小技术：使用**超时**。如果一个锁没有在超时的时间内结束，这个事务就进入了死锁状态。
<br/>
<br/>
锁管理器在加锁之前也会检测这个锁会不会产生死锁。但是重复一下，做这个检测是非常耗费性能的。因此，这些提前的检测是基本的规则。
<br/>
**两段锁**
<br/>
营造纯净的隔离最简单的方式是在事务开始的时候加锁，在事务结束的时候解锁。这意味着事务开始必须等待所有的锁，在事务结束的时候必须解除它拥有的锁。这是可以工作的但是**产生了巨大的时间浪费**。
<br/>
一个更快速的方法是**两段锁协议**（DB2，SQL Server使用这项技术），在这项技术中，事务被划分成两个极端：

+  **扩展阶段**，这个阶段，事务可以获取锁，但是不能解锁。
+  **收缩阶段**，这个阶段，事务可以解锁（锁定的数据已经处理完成，不能再继续处理），但是不能获取新锁。

<br/>
![两段锁](media/two-phase-locking.png)
<br/>
两段锁的思路有两个简单的规则：

+  解除不再使用的锁，减少需要这些锁其他事务等待时间。
+  避免在其他事务开始时候，修改数据，因此造成其他事务需要的数据不一致。

<br/>
这个协议工作的很好，除了这个情况，即一个事务修改数据，在释放关联的锁被取消（回滚）。这个情况结束时候，会导致其他事务读取修改后的值，而这个值就要回滚。为了解决这个问题，**所有的独占锁必须在事务完成的时候释放**。
<br/>
**一些话**
<br/>
 当然真正的数据库是更加复杂、精细的系统，使用了更多类型锁(比如意向锁)，更多的控制粒度（可以锁定一行，锁定一页，锁定一个分区，锁定一张表，锁定表分区）。但是基本思路是一样的。
<br/>
我只讲述了段春基于锁的方法。**数据版本是另一个处理这个问题的方式**
<br/>
版本处理的思路是：

+  每一个事务在同一时间修改同样的数据。
+  每一个事务有他自己的数据拷贝(或者说版本)。
+  如果两个事务修改同样的数据，只有一份修改会被接受，另一份修改将会被拒接，相应的实惠就会回滚（或者重新执行）。

<br/>
这个种方式提高了性能：

+  **读类型的事务不会阻塞写类型的事务**
+  **写类型的事务不会阻塞读类型的事务**
+  没有了”繁重的、缓慢的“锁管理器的影响

<br/>
万物皆美好啊，除了两个事务同时写一份数据。因此，你在结束时候，会有巨大磁盘空间浪费。
<br/>
数据版本和锁是两个不同的方式：**乐观锁和悲观锁**。他们都有正反两方面，根据你的使用情况（读的多还是写的多）。对于数据版本的介绍，我推荐这个关于Post供热SQL实现多版本的并发控制是![非常好的介绍](http://momjian.us/main/writings/pgsql/mvcc.pdf)
<br/>
一些数据库比如DB2（一直到DB2 9.7），SQL Server(除了快照隔离)都只是使用了锁。其他的像PostgreSQL，MySQL和Oracle是使用了混合的方式包括锁，数据版本。我还不知道有什么数据库只使用了数据版本（如果你知道那个数据库只是用了数据版本，请告诉我）。
<br/>
[更新在2015/08/20]，一个读者告诉我：
>  Firebird和 Interbase就是只使用了数据版本，没有使用记录锁，版本控制对于索引来说也是有非常有意思的影响：有时，一个唯一索引包含了副本，索引的数目比数据的行更多

<br/>
如果你读到关于不同的隔离层次的时候，你可以增加隔离层次，你增加锁的数量，因此事务在等待锁时间的浪费。这就是很多数据库默认不使用最高级别隔离（串行化）的原因。
<br/>
当然，你可以自己查看这些主流数据库的文档（像 Mysql，PostgreSQL，Oracle）。
<br/>
## 日志管理器
<br/>
我们已经看到为了增加性能，数据库在内存中存储数据。事务已经提交，但是一旦服务器崩溃，你可是可能丢失在内存中的数据，这就破坏了事务的持久性。
<br/>
如果服务器崩溃的时候，你正在向磁盘写入数据。你会得到一部分写入磁盘的结果，这样就破坏了事务的原子性。
<br/>
**事务的任何修改写入都是要不不做要么做完**
<br/>
为了解决这个问题，有两个方法：

+  **影子拷贝/影子页**：每一个事务创建自己的一份数据库拷贝（或者数据库的一部分拷贝）并在这个拷贝上进行修改。一旦出现错误，这份拷贝就被删除。如果成功，数据库就立即切换到数据到这份拷贝上，通过文件系统的技巧，他就删除了老的数据。
+  **事务日志**：事务日志是指就是存储空间。在每次写入磁盘之前，数据库已经写了一份信息在事务日志中，防止事务崩溃、取消的出现，数据库知道如何删除（或者完成）没有完成事务。


**WAL**
<br/>
当大型数据库上许多事务都在运行，影子拷贝/影子页有一个巨大的磁盘使用过量。这就是现代数据库使用**事务日志**。事务日志必须存储在**稳定的存储介质**上，我不能更加深入的挖掘存储技术但是使用(起码)RAID 磁盘是必须，避免磁盘损坏。
<br/>
大部分现代数据库（起码Oracle，SQL Server, DB2, PostgreSQL，Mysql和SQLite）处理事务日志使用了**Write-Ahead Logging protocol **(WAL),这个WAL协议是一组规则：

+  在数据库的每一个修改都产生一条日志记录，**这条日志记录必须在数据写入磁盘之前，写入事务日志。**
+  日志记录必须按顺序写，记录A先于记录B发生，那么必须在B之前写入。
+  当一个事务已经提交，在事务胜利完成之前，提交单据必须已经写入事务日志。


![日志管理器](media/log_manager.png)
<br/>
这个是日志管理器的工作。在缓存管理器和数据访问管理器（它将数据吸入磁盘）中间，就能找到它的身影，日志管理器把每个update/delete/create/commit/rollback，在写入磁盘之前，将相应的信息在事务日志上。很简单，不是吗？
<br/>
错误的答案！毕竟我们已经想过，我已经知道和数据库相关的一切事情都被”database effect“所诅咒。更加严重的问题是，找到具有很好性能的日志写入方法。如果吸入日志太慢，他们竟会拖慢所有的事情。
<br/>
**ARIES**
<br/>
在1992年，IBM的研究人员”发明“了一个WAL的增强版本叫做ARIES。ARIES差不多已经被所有的现代数据使用。虽然逻辑处理有一些差异，但是ARIES的思想都已经遍地开花了。跟着![MIT的课程](http://db.csail.mit.edu/6.830/lectures/lec15-notes.pdf)，我也引用了这项发明的思想，”没有比写一个好的事务恢复更好的做法“。在我5岁的时候，ARIES论文已经发表，我不了解那些苦逼的研究人民的传言。我打算在我们开始最后的技术章节之前，将一些有意思的东西，放松一下。我阅读了ARIES很大篇幅的研究论文，我发现这个很有意思。我想讲述一个ARIES的整体的形态，但是如果你有一些真材实料，强烈推荐去阅读那篇论文。
<br/>
ARIES全称是Algorithms for Recovery and Isolation Exploiting Semantics。
<br/>
这项技术的两个目的：

+  1）**写日志性能**
+  2）快速和**可靠的恢复**

数据库回滚事务有多种原因：

+  用户取消了事务
+  服务器运行失败或者网络失败
+  事务破坏了数据库的完整性（比如：你在一个column上有UNIQUE的约束，但是事务增加了一个相同的值）
+  因为死锁


有时候(比如，遇到网络失败)，数据库能恢复事务。
<br/>
但是这可能吗？为了回答这个问题，我们必须明白信息就在日志记录里面。
<br/>
**日志**
<br/>
每一个**事务的每一个操作(dadd/remove/modify)都要产生日志**。日志记录包括：

+   **LSN**: 唯一的日志序列号(Log Sequence Number)。LSN是按照时间如遇的。只意味着操作A比操作B发生的早，操作A的LSN比操作B的LSN要小。
+   **TransID**：操作的事务ID
+   **PageID**： 修改数据的磁盘位置。磁盘数据的最小单位是块，所以数据的位置也就是存放数据磁盘块的位置。
+   **PrevLSN**：相同事务，上一条日志记录的LSN。
+   **UNDO**：消除这个操作影响的方法。


比如，一个Update操作，这个UNDO就要存放update之前，**要update元素的元素值**（物理UNDO）或者可以回归之前状态的逆运算（物理UNDO）。

+   **REDO**：继续操作的方法
<br/>
相同的事情是，完成这个操作是这两种方法。要么存放修改之后值的元素，要么记录继续操作的运算。
<br/>
+  ....（ARIES还有两个字段：UndoNxtLSN和类型）

<br/>
注：原文是Page，但是磁盘单位但是我们更愿意用块。
<br/>
此外，磁盘的每个块(存储数据，不是日志)都有最有一个最后修改数据的操作日志记录的ID（LSN）
<br/>
\*这个方式LSN更复杂，因为因为他还要牵涉到日志存储。但是思路是一样的。
<br/>
\*\*ARIES只只用逻辑UNDO,因为处理物理UNDO才是个大麻烦。
<br/>
注：从我这点浅薄的见解，只有PostgreSQL没有使用UNDO。它使用一个垃圾收集服务，有这个服务来清除老版本的数据。这个实现跟PostgreSQL的数据版本时间有关。
<br/>
为了更清楚的理解，这个一个简化的图形，这个例子说明的是语句”UPDATE FROM PERSON SET AGE = 18;“产生的日志记录。这个语句是在ID18的事务中执行的。
<br/>
![简化ARIES日志](media/ARIES_logs.png)
<br/>
每一个日志都有唯一的LSN。这些日志通过相同的事务关联在一起，通过时间顺序进行逻辑管理。（这个执行链的最后一条日志，也是之后一个操作的日志）
<br/>
日志缓冲
<br/>
为了避免日志写入成为性能瓶颈，引入了**日志缓冲**。
<br/>
![日志写入过程](media/ARIES_log_writing.png)
<br/>
如果查询执行器要求这样的修改：

+  1）缓存管理器在它的缓冲区保存修改结果。
+  2）日志管理器在它的缓冲区保存相应的日志。
+  3）在这个步骤中，查询执行器关心这个操作是否完成（可以执行其他的修改）
+  4）这个时候（或者稍后）日志管理器将日志写入事务日志。决定什么时候写入日志由这个算法决策。
+  5）这个时候（或者稍后）缓存管理器将修改写入磁盘。决定什么时候写入磁盘由这个算法决策。

<br/>
**当一个事务已经提交，这意味着事务中的每一个操作的1，2，3，4，5个步骤都已经完成**。写入事务日志挺快的，因为它仅仅是”在事务日志中增加记录“，然而写入数据是非常复杂的，因为”要用方便、快速读的方式写入数据“。
<br/>
**STEAL和FORCE模式**
<br/>
性能原因，**5个步骤可能在提交之后才能完成**，因为在崩溃的情况下，可能需要REDO日志恢复事务执行。这是就是**NO-FORCE policy（非强制模式）**
<br/>
一个数据库可以选择强制模式（FORCE policy）（五个步骤必须在提交之前完成），这样可以减少恢复过程中的负载。
<br/>
另一个问题是选择**一步一步将数据写入磁盘（STEAL Policy）**，还是缓存管理器等待，直到提交指令，一次将所有的修改一次性写入磁盘（NO-STEAL）。在STEAL和NO STEAL中进行选择，要看你的需要。快速写入但是使用UNDO日志数据恢复慢，还是快速恢复。
<br/>
不同的模式对于数据恢复有不同的影响，请看如下总结：

+  **STEAL/NO-FORCE需要UNDO和REDO：最好的性能**但是更富在的日志和恢复过程（像ARIES）。**这是大部分数据库的选择**。注：我已经在大力那个的研究资料和课程中了解到这个事情，但是并没有在官方文档上找到这个描述（明确的）。
+  STEAL/FORCE只需要UNDO
+  NO-STEAL/NO-FORCE只需要REDO
+  NO-STEAL/FORCE不需要任何条件，**性能最差**和需要巨大的内存。

注： STEAL/NO STEAL描述对象是修改的数据  
    FORCE/NO FORCE说明的对象是写入日志的时间。
<br/>
**数据恢复**
<br/>
OK，我们已经有了非常好的日志，让我们来使用它。
<br/>
假设数据因为内部错误而崩溃，你重启数据库，这个恢复程序就开始了。
<br/>
ARIES从失败中通过三个方法进行恢复

+  1）**分析方式**：恢复程序读取真个事务日志，重新创建在崩溃的时候，正在执行的操作。这可确定了那些事务要被回滚（没有提交指令的事务都要被回滚），那些数据应该在崩溃的时候写入磁盘。
+  2）**redo方式**：这个方法从分析过程已经确定读取日志记录开始，使用REDO去更新数据库到在崩溃之前的状态。
<br/>
在redo过程中，REDO的日志是按照时间循序处理的（使用LSN）。
<br/>
对于每一条日志，恢复程序读取需要修改数据所属的磁盘块的LSN。
<br/>
如果LSN(page_on_disk) >= LSN(log_record)，这就是说数据在崩溃之前已经被写入磁盘（在崩溃之前，日志之后，LSN值已经被重写），所以已经完成。
<br/>
如果LSN(page_on_disk) < LSN(log_record)，磁盘块上的数据需要更新。
<br/>
REDO对所有的事务都回滚，是可以简化恢复过程的（但是我确定现代的数据不会这样做）
<br/>
+  3）UNDO方式：这个方式回滚所有在崩溃之前没有提交的事务。回滚从每个事务的最后一条日志开始，一直执行UNDO日志，直到出现非时间表的顺序（这个就是用到了日志的PrevLSN）。

<br/>
在恢复过程中，事务日志必须提醒恢复程序，他们要做出的行动，因为数据写入磁盘和写入事务日志是同步的。一个解决方案可以删除未完成的事务的条目，但是相当困难。相应的，ARIES在事务日志中写入综合性的日志，可以逻辑删除那些已经移除的事务的日志条目。
<br/>
当一个事务被”手动的“取消,或者被锁管理器(为了解决死锁)，或者仅仅因为网络失败，这个时候分析方法就是不需要的。事实上，那些需要REDO，那些需要UNDO的信息是在两个内存中的表里边：

+  **事务表**（存储所有当前的事务状态）
+  **脏页表**（存储所有需要被写入磁盘的数据信息）

<br/>
这些表被缓存管理器更新，在新事务创建时候，事务管理器更新。因为他们是在内存中的，当数据库崩溃，他们也要被销毁。
<br/>
分析阶段的工作就是运用事务日志的信息，重建崩溃后的两张表。*为了加快分析速度，ARIES提供了**检查点（checkpoint）**的概念。这个思路就是将事务表，脏页表一次一次的写入磁盘，在写入磁盘的时候，保存的最后一个LSN也写入磁盘。在分析阶段，之后LSN之后的日志才会被分析。
<br/>
##结束语
<br/>
在写这篇文章之前，我已经明白这是一个庞大的课题，需要花费大量时间才能写出深入的文章。事实证明，我太乐观了，我比预计多花费了两倍的时间，但是我学到了很多。
<br/>
如果你想对数据库有一个全面的了解，我推荐你阅读这个研究资料”![数据库结构](db.cs.berkeley.edu/papers/fntdb07-architecture.pdf)“。这个一个对于数据库一个非常好的入门介绍（有119页），同时对于非计算机学科的人也是非常易读的。这篇研究资料在本文的计划上帮了我很多。它想我的文章一样，没有太关注与数据结构和算法，更多的是，架构原理。
<br/>
如果你已经仔细的阅读了本文，你就应该知道一个数据库系统是多么的强大。因为这是一个长而又长的文章，让我来帮你回顾一下，我们看到了什么：

+   概述了B+树索引
+   全面描述了数据库系统
+   概述了重点在join操作上，基于成本的数据库优化方法。
+   概述了缓存池的管理
+   概述了事务管理


但是数据库有更多的智能。比如，我也不能讨论的高深的话题：

+  如何管理数据库集群和全局事务
+  如何在数据库系统正在运行中，进行快照
+  如何有效的存储（压缩）数据
+  如何管理内存

<br/>
请你再选择问题多多的NoSQL数据库和拥有坚实基础的关系型数据库时候多多考虑。当然别误导我，有一些NoSQL数据还是很棒的。但是他们仍然是关注与特定的问题的新人，也是吸引了一些应用使用。
<br/>
作为结束语，如果有问你一个数据库如何运行，除了临阵脱逃之外，你可以告诉他们
<br/>
![魔术](media/magic_low2.gif)
<br/>
或者给他们看看这篇文章。
<br/>
注：
data set 翻译为数据集，指的是table。
index anding 这个操作，没明白是个什么操作，已经在pg的官方文档上寻找，没有找到。
cache和buffer：对于本文来说，这两者并没有什么太大的不同，都是指的是内存。
而内存中一些区别，具体可以看操作系统drop cache的级别：
但是在本文中，并没有严格的区分。
