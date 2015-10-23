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




{% qnimg test/demo.png %}



---

**References**:

- [Isolating Code Under Test with Microsoft Fakes](https://msdn.microsoft.com/en-us/library/hh549175.aspx)
- [Hexo博客转移 从github到gitcafe](http://magicer.coding.io/2015/10/Hexo-github-gitcafe/)
- [List of DNS Record Types](https://en.wikipedia.org/wiki/List_of_DNS_record_types)
- [DNS解析过程详解](http://blog.chinaunix.net/uid-28216282-id-3757849.html)



And also:

- [Markdown Chech and cross character](http://stackoverflow.com/questions/712132/in-html-i-can-make-a-checkmark-with-x2713-is-there-a-corresponding-x-mark)
