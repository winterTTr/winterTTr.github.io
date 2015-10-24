title: 从DNS到github pages自定义域名 -- 漫谈域名那些事
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

---

# 域名
域名就是一段文字，更具体地说，是一段人类容易识别的文字，它的作用很简单，就是用来给一个ip起一个人们能够记得住的名字。
域名是一种资源，很多时候甚至是地位财富和身份的象征，所以越来越多的geek们都趋之若鹜的申请自己的域名，我也不例外。


## 根域
我们通常知道的域名，例如[winterttr.me](http://winterttr.me)，其实是完整域名的缩写。 真正的全称为[winterttr.me.](http://winterttr.me)，请注意最后的`.`，这个就是根域。 它的现实体现为全球13台固定ip的根域服务器。 从`a.root-servers.net`到`m.root.servers.net`。

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

---

# 有关DNS的那些事
咱们谈完了域名，那么就不得不说到DNS（Domain Name System)，DNS所承担的主要任务，就是所谓的域名解析。这些由DNS系统中的DNS服务器负责。那么，DNS服务器解析一个域名得到了什么？ -- **IP地址**

所以，域名解析的过程，说白了就是把一个人类记得住的域名变成ip网络中机器认识的ip地址。

那么，DNS服务器上都存了些啥？最主要的就是能够完成域名解析的一些记录

## A记录（A record)
A记录在DNS中的意义就是，域名到ip地址的转换。
所以，当我们在DNS服务器中添加一个A记录时，是告诉服务器，将某个特定的域名映射到一个ip地址。这个算是最简单直白的转换规则了。

## CNAME记录（CNAME record)
CNAME的意义，简单说就是别名，即将一个域名射到另一个域名（区别于A记录的ip）。所以，CNAME通常有两种用法：
- 不同顶级域名之间的跳转
例如：我的域名是 `winterTTr.me`(顶级域名为`me`)。如果我希望，当我访问这个域名的时候，实际上是访问我的`winterTTr.github.io`（顶级域名为`io`）的主页时，虽然他们在不同的顶级域名，但是我可以用CNAME记录映射。
- 将一个子域名映射到域名
例如，你想当访问者输入`www.winterTTr.me`（一个`WWW`子域名）的时候，仍旧访问`winterTTr.me`这个域名所指向的内容时，可以将`www.winterTTr.me`利用CNAME记录映射到`winterTTr.me`。

## NS记录（Name Server）
简单来说，就是声明谁来负责解析我这个域名，指定了负责解析我这个域名的服务器的地址。这条记录赋予我们一个特殊的能力，就是，我可以让自己指定的一个DNS解析服务器，而不一定是域名提供商自带的域名解析服务器。简单来说，就是在godaddy买的域名，默认是使用godaddy的域名服务器来进行域名解析的，但是如果我想让别的server解析（例如NDSPod），而不受godaddy服务器的限制呢？那就是更改这个NS记录的内容。一般来讲，是两条记录，一条主服务器，一条副服务器。

## 现实中的一些例子
这个就是我的域名在`DNSPod`中的设置。

{% qnimg "2015-10-23-from-dns-to-github-custom-domain/dnspod-setting-of-winterttr.png" extend:-watermark.black %}

可以看到，NS记录为dnspod的服务器域名，`dnspod`提供了一种非常方便的服务。就是可以根据不同的线路类型，进行不同的解析。图中可以看到，国内的使用gitcafe，国外的使用github，同时，baidu抓取国内线路gitcafe的内容，躲避了github封闭baidu spider的问题。

---

# 在godaddy花钱的时候买到了什么

这不是废话么，买到了域名！
当然，除了这一串字符串之外，是什么让我们真正拥有了这些字符串使用权呢？

## 在godaddy中配置域名的能力
话说花了钱，买了域名，为啥这个域名就是你的呢？
这是因为，godaddy将这个域名中的各种记录的配置权力分配了给你，于是，你可以定义域名的ip（A记录），或者将这个域名指向另一个别名（CNAME记录）。

## 除了这些我还能做啥
默认的域名，是在godaddy自带的域名解析服务器中进行的。godaddy是提供更改NS记录的权利的，所以，我们可以将这个域名解析的能力交给godaddy之外的人，这就是我如何做到使用`DNSPod`来进行域名解析的。

---
# github中的自定义域名

那么，经过上面一堆讲解，最终还是回到最实际的问题，github中的自定义域名。


## github中的域名支持
github的域名是支持A记录的，这个意思就是，github的服务器域名是个固定ip。所以，当我们需要将申请的域名给予一个自己的github.io的地址的时候，我们可以在DNS服务器的配置中添加一条A记录，指向github的服务器地址。
现在github的服务器地址为：
- 192.30.252.153
- 192.30.252.154

如果你做了上面的操作，那意思就是，我希望把我的主域名`winterTTr.me`完全指向github.io的主页。

不过，如果你的只是想将一个子域名，例如`www.winterTTr.me`，而不是主域名`winterTTr.me`分配给你的github主页，那么，A记录会完全绑定你的主域名，所以这个场景A记录不适合。你需要在DNS服务器中添加一条CNAME记录，将子域名指向`winterTTr.github.io`，这样，当用户访问`WWW`子域名的时候，会跳转到github.io的主页上。


## 告诉github你的域名
在项目下建立一个CNAME文件，在其中写上给你的主页分配的域名地址。
这个操作的作用在哪里？

1. 当直接访问github.io主页的时候github知道redirect到哪里
也就是说，当你指定了CNAME之后，我们再次访问一个`github.io`的网站时，我们会发现，域名自动变成了我们制定的自定义域名。这是因为CNAME中指出了自定义域名是什么，所以，当我们访问`github.io`的时候，会触发`http 301`。

  > azureuser@ubuntu-jpe:~$ curl -I winterttr.github.io
  > HTTP/1.1 301 Moved Permanently
  > Server: GitHub.com
  > Content-Type: text/html
  > Location: http://winterttr.me/
  > X-GitHub-Request-Id: 2BF9481E:370C:8930CF:562B48F6
  > Content-Length: 178
  > Accept-Ranges: bytes
  > Date: Sat, 24 Oct 2015 09:02:36 GMT
  > Via: 1.1 varnish
  > Age: 54
  > Connection: keep-alive
  > X-Served-By: cache-nrt6130-NRT
  > X-Cache: HIT
  > X-Cache-Hits: 1
  > X-Timer: S1445677356.846925,VS0,VE0
  > Vary: Accept-Encoding

  我们可以看到，github的服务器知道，我们需要访问的io网站已经有了别的域名，并且返回301让浏览器跳转到自定义域名。

2. 当用你的域名访问的时候github知道我去那个io里面找
当然，当我们直接使用自定义域名访问的时候，由于DNS服务器的配置，最终我们会访问github.io的主机，当主机收到我们的请求的时候，会拿我们http请求中的host和repository中的CNAME文件比较，从而知道，当前的域名应该访问那个具体的`xxx.github.io`的内容。


---

这基本上就是从一个从申请域名到到github.io配置后的完整故事

---

**References**:

- [How DNS Works](https://technet.microsoft.com/en-us/library/cc772774(v=ws.10).aspx)
- [Do CNAME records result in a second DNS lookup?](https://serverfault.com/questions/419617/do-cname-records-result-in-a-second-dns-lookup?newreg=ca603855951147c98c90169583e43a68)
- [Subdomain vs Domain](http://halfelf.org/2012/subdomain-vs-domain/)
- [Domain Name System](https://en.wikipedia.org/wiki/Domain_Name_System)
- [What happens if multiple GitHub Pages repositories all point to the same domain name?](https://www.quora.com/What-happens-if-multiple-GitHub-Pages-repositories-all-point-to-the-same-domain-name)
- [Setting up a custom domain with GitHub Pages](https://help.github.com/articles/setting-up-a-custom-domain-with-github-pages/)


---
{% raw %}
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.
Please refer to the auther's name `winterTTr` when you remix, transform, and build upon this material. 
{% endraw %}
本作品由`winterTTr`创作，采用[知识共享署名-非商业性使用 4.0 国际许可协议](http://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可。修改，参照或者转载请注明作者`winterTTr`
