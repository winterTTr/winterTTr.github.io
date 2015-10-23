title: 从DNS到github pages自定义域名 -- 谈谈域名那些事
date: 2015-10-23 10:59:17
tags:
  - github
  - gitcafe
  - github page
  - Domain Name
  - DNS
  - 域名
  - 域名服务器
---

当我们在github申请一个pages之后，很多人会选择自定义域名，给自己的github pages一个属于自己的名字。
那么，从申请到域名到最后的github自定义域名设置，中间究竟发生了什么，本文就来说说域名那些事。

<!--more-->

# 域名
域名就是一段文字，更具体地说，是一段人类容易识别的文字，它的作用很简单，就是用来给一个ip起一个人们能够记得住的名字。
域名是一种资源，很多时候甚至是地位财富和身份的象征，所以越来越多的geek们都趋之若鹜的申请自己的域名，我也不例外。


## 根域
我们通常知道的域名，例如[winterttr.me](http://winterttr.me)，其实是完整域名的缩写。 真正的全称为[winterttr.me.](http://winterttr.me)，请注意最后的`.`，这个就是根域。 它的显示体现为全球13台固定ip的根域服务器。 从`a.root-servers.net`到`m.root.servers.net`。

当我们在进行dns查询的时候，如果一个全新的域名从来没有进行查询，那么，最终会向这13台根域服务器进行请求。 不过，现在的浏览器已经基本上默认不再添加这个"多余"的点了，已然成为一种默认习惯。

当然，事实上，并不是真正的13台，而是13组，每一台在全球都有很多的镜像节点，所以你不用担心其中一台挂了会引起全球混乱~

## 顶级域名(Top Level Domain - TLD)
常用的顶级域名分为几种：
1 国家顶级域名，例如`.cn`,`.jp`
2 机构顶级域名，例如`.com`,`.edu`
3 还有其他分类

## 二级域名
这个就是我们常常能够申请到的域名，在顶级域名的左侧加上的一个自定义的文字段。据个例子： [winterTTr.me](http://winterTTr.me)。所以，我们通常所说的域名，往往指的是这个二级域名。


## 子域名(sbudomain name)
相对于上文所提到的“我们通常所说的域名”（二级域名）的基础上，又加入了子域名的概念，就是在一个域名的前面，加上新的字段，代表这个域名下的某个特定的主机或者协议。最常用的就是`WWW`协议，所以，我的子域名`www.winterttr.me`就是`winterttr.me`的`WWW`子域名。

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


{% qnimg test/demo.png title:title alt:description %}


{% qnimg test/demo.png title:title alt:description extend:?watermark/2/text/aHR0cDovL3dpbnRlclRUci5tZQ==/font/Y29uc29sYXM=/fontsize/520/fill/YmxhY2s= %}


---

**References**:

- [How DNS Works](https://technet.microsoft.com/en-us/library/cc772774(v=ws.10).aspx)
- [Do CNAME records result in a second DNS lookup?](https://serverfault.com/questions/419617/do-cname-records-result-in-a-second-dns-lookup?newreg=ca603855951147c98c90169583e43a68)
- [Subdomain vs Domain](http://halfelf.org/2012/subdomain-vs-domain/)


And also:

- [Markdown Chech and cross character](http://stackoverflow.com/questions/712132/in-html-i-can-make-a-checkmark-with-x2713-is-there-a-corresponding-x-mark)
