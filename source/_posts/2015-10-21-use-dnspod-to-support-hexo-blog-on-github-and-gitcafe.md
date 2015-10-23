title: 使用DNSPod支持基于github和gitcafe双线路的hexo博客
date: 2015-10-21 23:39:32
tags:
  - hexo
  - github
  - gitcafe
  - github page
  - DNSPod
  - DNS
  - baidu
  - baidu爬虫
  - baidu spider
---

为什么要支持双线路的博客？
- github国内访问速度缓慢，偶尔甚至无法访问
- baidu爬虫被github禁用([外媒一撇](https://news.ycombinator.com/item?id=9275041))
- 不想放弃github

这样做可以获得什么优势：
- github对应国外访问，gitcafe对应国内访问，保持速度
- baidu爬虫抓取gitcafe数据，不会受到github的禁用影响
- 双线路同一域名，在DNS服务器上实现分流，不会因为主机不同影响最终用户的访问域名

本文希望从原理上将比较详细的描述整个配置的过程。

<!--more-->

## DNS那些事
在我们开始之前，让我们先熟悉一下DNS相关的一些知识。

### 根域名vs顶级域名vs二级域名
我们可以将一个普通的网络域名看成逆序的域名排列而成。
据个例子： [www.winterTTr.me](http://www.winterTTr.me)

- www.winterTTr.me为二级域名
- winterTTr.me为顶级域名
- me为根域名

以此类推

### A记录（A record)
A记录在DNS中的意义就是，域名到ip地址的转换。
所以，当我们在DNS服务器中添加一个A记录时，是告诉服务器，将某个特定的域名映射到一个ip地址。

### CNAME记录（CNAME record)
CNAME的意义，简单说就是别名，即将一个域名射到另一个域名（区别于A记录的ip）。所以，CNAME通常有两种用法：
- 不同根域名之间的跳转
例如：我的顶级域名是 `winterTTr.me`(根域名为`me`)。如果我希望，当我访问这个域名的时候，实际上是访问我的`winterTTr.github.io`（根域名为`io`）的主页时，虽然他们在不同的根域名，但是我可以用CNAME记录映射。
- 将一个二级域名映射到顶级域名
例如，你想当访问者输入`www.winterTTr.me`（一个二级域名）的时候，仍旧访问`winterTTr.me`这个域名所指向的内容时，可以将`www.winterTTr.me`利用CNAME记录映射到`winterTTr.me`。

### github中的CNAME文件做了啥
具体的细节我不是很清楚，也没有深刻研究，就不误导大家了。
从结果上讲，如果配置过github主页的域名绑定，那么会做如下几件事：
1. 在dns服务器上建立A记录，将顶级域名指向github的服务器ip
这个行为的目的是告诉dns服务器，如果有人访问我的域名，请将github.io服务器的ip告诉他。我们肯定会问到，github.io是个顶级域名，而我们的pages是在一个二级域名，github.io如何知道，刚刚请求的那个域名是对应的哪个二级域名呢？这就是第二步的原因。
2. 在github page中建立CNAME文件，并在文件中指定自定义域名
这个行为的目的，如果github服务器收到一个域名请求，请使用CNAME文件中指定的这个域名进行域名匹配查到，这样github就知道应该返回那个具体的github page的二级域名的内容了。

### gitcafe的自定义域名
gitcafe在这点上比github简单一些，可以在项目的配置页面直接添加自定义域名，原理和github是一样的，用来帮助gitcafe服务器查到应该返回哪个具体的地址。
不过，最新的gitcafe已经不支持A记录，只能使用CNAME记录实现域名跳转。如果你登记的自定义域名和你请求的自定义域名不能匹配，也是无法找到目标地址的。


### 在Godaddy买域名的时候买到了什么
当然是买到了域名！！这是废话！！
同时，你还买到了
- 配置解析这个域名时所使用的DNS服务器的权利（默认配置为godaddy的服务器）
- 配置godaddy的DNS服务器中这个域名的各种记录的权利

我们常用的，基本上是第二条给我们的权利。但是，第一条给与我们更大的配置空间，这样就意味着，我们可以不使用godaddy自带的DNS服务器，而是将域名解析的能力交给其他的域名服务器，例如DNSPod，这样我们就可以使用其他域名服务器所带给我们的godaddy中没有的能力（例如：DNSPod所具备的，根据不同访问线路动态返回DNS结果，这也是双线路的根本来源）

## 如何设置DNSPod中的记录



{% qnimg test/demo.png %}



---

**References**:

- [Isolating Code Under Test with Microsoft Fakes](https://msdn.microsoft.com/en-us/library/hh549175.aspx)
- [Hexo博客转移 从github到gitcafe](http://magicer.coding.io/2015/10/Hexo-github-gitcafe/)
- [List of DNS Record Types](https://en.wikipedia.org/wiki/List_of_DNS_record_types)
- [DNS解析过程详解](http://blog.chinaunix.net/uid-28216282-id-3757849.html)



And also:

- [Markdown Chech and cross character](http://stackoverflow.com/questions/712132/in-html-i-can-make-a-checkmark-with-x2713-is-there-a-corresponding-x-mark)
