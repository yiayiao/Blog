---
title: Python学习总结01——基础数据类型
date: 2019-05-03 22:59:48
mathjax: true
tags:
 - Code
categories:
 - Python
 - Basic
---

### 写在前面的

Python学习总结这一系列博客，记录了自己在学习Python3的过程中的一些知识总结与思考。写博客出发点不为求多求全，主要记录下面几类知识：

1. 容易忽略但是又很重要的知识，比如Python3的字符编码
2. 理解起来比较困难的知识，比如metaclass
3. 记忆起来比较困难的知识，这一点和上面一点又重合，但又不完全相同，比如Socket，理解起来很容易，但是写起来还是费劲的。记录这部分知识，自然是为了以后查找复制之用。

相应的，显而易见的，容易理解，容易记忆的知识，博客中不做记录。如果你有缘看到了这些博客，它们对你学习入门可能起不到太大的帮助，但是或许可以帮你查漏补缺，让我们一起思考与交流。

### 数值运算

Python相比与Java和C++，在加减乘除之外，多了两个运算符：**//** 和 **\*\***。抛开几个比较简单的运算符，这里关注下表中的几个运算符：

| 运算符 | 描述     | 实例                                          |
| :----- | :------- | :------------------------------------------------ |
| /      | 除法运算 | 4 / 2得到2.0， 结果为float类型                                   |
| //     | 整数运算 | 4 // 2得到2， 结果为int类型； 4.0 // 2得到2.0， 结果为float类型  |
| \*\*     | 幂运算   | 4 \*\* 2得到16， 结果为int类型； 4.0 \*\* 2得到16.0, 结果为float类型 |

注意Python的'//'作用于int类型时可以相当于Java和C++的'/'，而当参数之一为float类型时却与Java和C++的效果不同。简言之Python严格区分了除法和整除运算符，而Java和C++没有区分，Java和C++的'/'作用于两个int时为整除，参数之一为浮点类型是为除法。

Python新增幂运算符\*\*，那么问题来了, $\sqrt{4}$怎么写, 你可能已经猜到了：4 \*\* 0.5；相似的， $\sqrt[3]{4}$可以写作：4 \*\* (1 / 3)。

### 数值类型

Python3分别用int类型和float类型表示整数与浮点数，问题来了，Python3有没有long和double类型？答案是没有，接着往下问，int类型和float的表示范围分别是什么？

尽量不卖关子，Python3的int类型取代了旧的long类型，换句话说int类型没有范围限制，网上有些博客写到在Python3里面int的最大值是sys.maxsize, 这其实是误解，从名字就可以看出端倪，sys.maxsize表示的是list和str最大的size，并不是int类型的最大值。以下为官网的部分摘要：

> PEP 0237: Essentially, long renamed to int. That is, there is only one built-in integral type, named int; but it behaves mostly like the old long type.

> The sys.maxint constant was removed, since there is no longer a limit to the value of integers. However, sys.maxsize can be used as an integer larger than any practical list or string index. It conforms to the implementation’s “natural” integer size and is typically the same as sys.maxint in previous releases on the same platform (assuming the same build options).

与int不同，float类型确实有最大值的，可以通过sys.float_info.max和sys.float_info.min分别获取float的最大值和最小值。它们分别为：1.7976931348623157e+308 和 2.2250738585072014e-308，在Ubuntu环境上测试，Python的float类型的范围与C++的Double类型的范围相同，C++的Double类型的范围可以通过执行以下代码查看：

``` C++
#include <iostream>
#include <limits>
using namespace std;
int main() {
    cout << "double maxvalue" << numeric_limits<double>::max() << endl;
    cout << "double minvalue" << numeric_limits<double>::min() << endl;
    return 0;
}
```

Python3的float还包含了正无穷和负无穷的表示，float("inf")可以表示正无穷，float("-inf")可以表示负无穷。既然float能够表示的数据范围有限，Python怎么进行浮点型的大数计算呢，可以通过Decimal实现，需要import引入decimal模块，不再这里赘述。

float不光有数据范围的限制，float之间的计算还需要考虑精度的损失，两个浮点数据之间不能直接比较。sys.float_info.epsilon可以表示机器能够区分出的两个浮点数的最小差别，判断两个函数是否相等，可以定义函数如下：

``` Python
def is_float_equal(a,  b):
    return abs(a - b) <= sys.float_info.epsilon
```

此外，还可以通过import math模块，通过math.isclose(a, b, *, rel_tol=1e-09, abs_tol=0.0)函数判断两个浮点数是否相等，按照官网描述，该函数实现逻辑如下：

``` Python
return abs(a-b) <= max(rel_tol * max(abs(a), abs(b)), abs_tol)
```

稍加留心，我们可以注意到math.isclose函数默认比较参数间的相对误差，可以使用match.isclose重写上文的is_float_equal函数，比较两个浮点数据的绝对误差：

``` Python
math.isclose(a, b, rel_tol=0, abs_tol=sys.float_info.epsilon)
```

### 列表 - list

Python用列表list表示了Java和C++中的Array，List和Vector等类型，这确实省去了很多的麻烦，接下来从“增删改查”看看列表的操作。

1.“增”，下表分为从列表的末尾或者中间，添加列表或者单个元素：

|操作|函数实现|切片实现|
|:----|:---|:---|
|在my_list末尾插入列表|my_list.extent(another_list)|my_list[len(my_list):]=another_list|
|在my_list末尾插入元素|my_list.append(element)|my_list[len(my_list):]=[element,]|
|在my_list的index下标位置插入列表|---|my_list[index:index] = another_list|
|在my_list的index下标位置插入元素|my_list.insert(index, element)|my_list[index:index]=[element,]|

不由得感叹，切片真是一个无比神奇的东西！注意通过切片或者extent函数插入字符串str时，会将str作为列表处理，我们看下面一段代码：

``` Python
my_list = [1, 2, 3]
my_list.extent('123')
print(my_list)
```

它的执行结果是：1,2,3,'1','2','3'，而不是：1,2,3,'1,2,3'。

2.“删”，下表分别从列表的末尾或者中间删除元素

|操作|函数实现|切片实现|
|:----|:---|:---|
|删除my_list末尾的元素|my_list.pop()|my_list[-1:0]=[]|
|删除my_list下标为index的元素|del my_list[index]|my_list[index:index+1]=[]|

一般不会遇到从列表中删除列表这种需求，如果遇到了，也可以使用切片实现，比如保留my_list中前三个元素，其他全部删除：my_list[3:] = []

3.“改”

更改列表中的元素，最自然而然的，就是使用下标操作，或者切片，不再赘述。

4.“查”

说到查，最自然而然的，是想到find关键字，然而很遗憾，Python并没有find函数，因为它有另一个关键字：in，或者我们可以转变一下思维，从列表my_list中查找element元素，可以转而判断my_list中element元素的个数count是否大于0，或者求element元素在my_list中的下标index。

``` Python
my_list = [1,2,3]
element = 1
#判断element是否在my_list中
if element in my_list:
    print("exist")
#或者
if my_list.count(element) > 0:
    print("exist")
#或者下面这样
try:
    my_list.index(element)
except ValueError:
    print('not exist')
else:
    print('exist')
```

列表的index函数在这里略显诡异，如果element在my_list中不存在，它并没有返回-1或者其他的无效的值，而是直接抛出异常。我觉得这又是Python思想的一个体现，解决一个问题一种实现就足够了，第二种第三种实现都是多余的，用index函数判断元素是否存在就是多余的，不被推荐的实现。

### 元组 - tuple

“元组（tuple）”可以被看做不能被修改的列表，因为不能被修改，元组可以作为字典的key，而列表不行。当一个函数包含了多个返回值时，使用type命令查看该函数的返回值，可以看到其类型是一个元组。

### 字符串 - str



### 一些思考

以下内容仅代表个人观点和思考，可能有失偏颇。

先讨论一点，int 和 float 类型，即整类型和浮点数类型，是不是类，如果你和我一样，是从C++和Java走来的，那我想我们应该有相似的认识，int 和 float 属于基本数据类型，所谓基本数据类型，就是它们不是类。但是毫无疑问，在Python的世界里面，int 和 float 都是类，不信你用 type 函数测试一下，Python会清晰的告诉你，它们的类型分别是<class 'int'\>和<class 'float'\>。

那么问题来了，你说它是类，它就是类了吗？这个问题我自己思考了比较久，我自己的看法是，还是别把 int 和 float 基本数据类型当类。看到这里可能有人已经在心里骂我了，我们先想想什么是类，面向对象的忠实信徒可能会说，万事万物皆可为类，可是回想一下类的定义：类是对现实生活中一类具有共同特征的事物的抽象。你是否接受有些东西不可抽象，数字是什么的抽象，既然是抽象，意味着有些特征可以忽略，当我们说数字是类，我们是需要使用它什么特征，又需要忽略它什么特性。我觉得这个问题说不清楚，所以我宁愿接受 int 不是类，当然对这个问题你可能有自己的观点。

为什么我要花时间扯这些问题呢，因为我觉得我是个笨人，我这个笨人有这样一些特点，如果一些东西分类不够明确，我就可能用混；如果有些东西说不清楚它是什么，除非每天都接触，否则我就会忘。

当我从数字类型本身，而不是类的角度看 int 和 float 时，就多了一个看它的维度，首先是它具备了原子性，不能对它内部进行修改，从这一点出发，Python世界里的 str 和 tuple 类型有相似的特点，而C++世界里的std::string类型则没有；其次它们再次变得简单，我可以用小学所学的加减乘除作用于它，不需要关注更加复杂的东西。


### 3. Python3的列表拷贝

引用赋值，浅拷贝，深拷贝


实现类的成员拷贝函数？留到后面，我还没有学到
