title: Python的函数参数传递：传值？引用？
date: 2015-10-24 19:58:47
tags:
  - python
  - arguments
  - 参数传递
sticky: 10
---


我想，这个标题或许是很多初学者的问题。尤其是像我这样的对C/C++比较熟悉，刚刚进入python殿堂的朋友们
。C/C++的函数参数的传递方式根深蒂固的影响这我们的思维--引用？传值？究竟是那种呢。
语言的特性决定了是使用的方法，那么，现在我们来探究一下python的函数参数传递方式。

<!--more-->

# 对象vs变量

在python中，`类型`属于`对象`，`变量`是没有类型的，这正是python的语言特性，也是吸引着很多pythoner的一点。所有的变量都可以理解是内存中一个对象的“引用”，或者，也可以看似c中void*的感觉。所以，希望大家在看到一个python变量的时候，把`变量`和真正的`内存对象`分开。

> 类型是属于对象的，而不是变量。

这样，很多问题就容易思考了。

例如：
{% codeblock 对象vs变量 lang:python %}
nfoo = 1   #一个指向int数据类型的nfoo（再次提醒，nfoo没有类型）
lstFoo = [1]   #一个指向list类型的lstFoo，这个list中包含一个整数1
{% endcodeblock %}


# 可更改(mutable)与不可更改(immutable)对象

对应于上一个概念，就必须引出另了另一概念，这就是`可更改（mutable）对象`与`不可更改（immutable）对象`。
对于python比较熟悉的人们都应该了解这个事实，在python中，strings, tuples, 和numbers是不可更改的对象，而list,dict等则是可以修改的对象。那么，这些所谓的可改变和不可改变影响着什么呢？

{% codeblock 可更改vs不可更改 lang:python %}
nfoo = 1
nfoo = 2

lstFoo = [1]
lstFoo[0] = 2
{% endcodeblock %}

代码第2行中，内存中原始的1对象因为不能改变，于是被“抛弃”，另nfoo指向一个新的int对象，其值为2

代码第5行中，更改list中第一个元素的值，因为list是可改变的，所以，第一个元素变更为2。其实应该说，lstFoo指向一个`包含一个对象的数组`。赋值所发生的事情，是有一个新int对象被指定给lstFoo所指向的数组对象的第一个元素，但是对于lstFoo本身来说，所指向的数组对象并没有变化，只是数组对象的内容发生变化了。这个看似void*的变量所指向的对象仍旧是刚刚的那个有一个int对象的list。

如下图所示：

{% qnimg "2015-10-24-python-passing-arguments-as-value-or-reference/mutable-immutable-object.png" extend:-watermark.black" %}


# Python的函数参数传递：传值？引用？
对于变量（与对象相对的概念），其实，python函数参数传递可以理解为就是变量传值操作，用C++的方式理解，就是对void*赋值。如果这个变量的值不变，我们看似就是引用，如果这个变量的值改变，我们看着像是在赋值。有点晕是吧，我们仍旧据个例子。


{% codeblock 不可变对象参数调用 lang:python %}
def ChangeInt( a ):
    a = 10
nfoo = 2 
ChangeInt(nfoo)
print nfoo #结果是2
{% endcodeblock %}

这时发生了什么，有一个int对象2，和指向它的变量nfoo，当传递给ChangeInt的时候，按照传值的方式，复制了变量nfoo的值，这样，a就是nfoo指向同一个Int对象了，函数中a=10的时候，发生什么？（还记得我上面讲到的那些概念么），int是不能更改的对象，于是，做了一个新的int对象，另a指向它（但是此时，被变量nfoo指向的对象，没有发生变化），于是在外面的感觉就是函数没有改变nfoo的值，看起来像C++中的传值方式。


{% codeblock 可变对象参数调用 lang:python %}
def ChangeList( a ):
    a[0] = 10
lstFoo = [2]
ChangeList(lstFoo )
print nfoo #结果是[10]
{% endcodeblock %}

当传递给ChangeList的时候，变量仍旧按照“传值”的方式，复制了变量lstFoo 的值，于是a和lstFoo 指向同一个对象，但是，list是可以改变的对象，对a[0]的操作，就是对lstFoo指向的对象的内容的操作，于是，这时的a[0] = 10，就是更改了lstFoo 指向的对象的第一个元素，所以，再次输出lstFoo 时，显示[10]，内容被改变了，看起来，像C++中的按引用传递。

恩，现在是不是对python中的变量和对象的概念有了更深入的理解了呢？
通过我上面的解释，我想大家也可以自己搞定其他类型对象的传递问题了吧。

---

原文写于2008年，发表在CSDN，发现文章反馈比较多，所以适当更新后重新发表在个人博客。

---

{% raw %}
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.
Please refer to the auther's name `winterTTr` when you remix, transform, and build upon this material. 
{% endraw %}
本作品由`winterTTr`创作，采用[知识共享署名-非商业性使用 4.0 国际许可协议](http://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可。修改，参照或者转载请注明作者`winterTTr`
