title: 使用DNSPod支持基于github和gitcafe双线路的hexo博客
date: 2015-10-24 23:39:32
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
categories:
  - internet
---

为什么要支持双线路的博客？
- github国内访问速度缓慢，偶尔甚至无法访问
- baidu爬虫被github禁用([外媒一撇](https://news.ycombinator.com/item?id=9275041))
- 不想放弃github

这样做可以获得什么优势：
- github对应国外访问，gitcafe对应国内访问，保持速度
- baidu爬虫抓取gitcafe数据，不会受到github的禁用影响
- 双线路同一域名，在DNS服务器上实现分流，不会因为主机不同影响最终用户的访问域名

在开始本文之前，强烈建议读一下我的另一篇文章[从DNS到github pages自定义域名 -- 漫谈域名那些事](http://winterTTr.me/2015/10/23/from-dns-to-github-custom-domain/)，可以帮助你熟悉过程中的很多原理。

<!--more-->

# 更改hexo的配置文件
实现github和gitcafe部署
```
deploy:
  type: git
  repo:
    github: https://github.com/winterTTr/winterTTr.github.io.git
    gitcafe: https://gitcafe.com/winterTTr/winterTTr.git
```
这样就可以使用`hexo delpy`直接部署github和gitcafe了。

这里要说一句，github使用master分支作为静态页展示，gitcafe使用gitcafe-pages分支作为静态页展示。这点hexo的deploy里面已经很好的处理了，我们不用特别指定。


# 申请DNSPod添加域名管理
NDSPod的域名解析服务中，提供根据线路进行不同的路由的功能是免费的，这也是为什么选择DNSPod的原因

添加了域名后，DNSPod会自动导入当前的域名下的各种记录
{% qnimg "2015-10-24-use-dnspod-to-support-hexo-blog-on-github-and-gitcafe/dnspod-add-domain-name.jpg" %}

在这里我们可设置针对不同线路的CNAME记录

{% qnimg "2015-10-24-use-dnspod-to-support-hexo-blog-on-github-and-gitcafe/dnspod-setting-of-winterttr.jpg" %}



# 在域名提供商更改NS记录
我是用的是godaddy申请的域名，godaddy中NS记录不是在`Zone File`的配置下，而是点击域名旁边的下拉列表更改的
{% qnimg "2015-10-24-use-dnspod-to-support-hexo-blog-on-github-and-gitcafe/godaddy-ns-record.jpg" %}

将其设置成DNSPod的服务器地址

{% qnimg "2015-10-24-use-dnspod-to-support-hexo-blog-on-github-and-gitcafe/godaddy-set-dnspod.jpg" %}


# 等待
经过上面设置后，还需要等待全球DNS服务器的递归更新。

然后可以测试从不同地区的主机效果：

这里是从家里的机器：

> MacBook-Pro:~ winterTTr$ dig winterttr.me
> ; <<>> DiG 9.8.3-P1 <<>> winterttr.me
> ;; global options: +cmd
> ;; Got answer:
> ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 7737
> ;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 2, ADDITIONAL: 10
> 
> ;; QUESTION SECTION:
> ;winterttr.me.			IN	A
> 
> ;; ANSWER SECTION:
> winterttr.me.		600	IN	CNAME	winterttr.gitcafe.io.
> winterttr.gitcafe.io.	600	IN	A	103.56.54.5
> 
> ;; AUTHORITY SECTION:
> gitcafe.io.		600	IN	NS	f1g1ns1.dnspod.net.
> gitcafe.io.		600	IN	NS	f1g1ns2.dnspod.net.
> 
> ;; ADDITIONAL SECTION:
> f1g1ns1.dnspod.net.	171513	IN	A	180.153.9.189
> f1g1ns1.dnspod.net.	171513	IN	A	182.140.167.166
> f1g1ns1.dnspod.net.	171513	IN	A	111.30.132.180
> f1g1ns1.dnspod.net.	171513	IN	A	113.108.80.138
> f1g1ns1.dnspod.net.	171513	IN	A	125.39.208.193
> f1g1ns2.dnspod.net.	55977	IN	A	101.226.30.224
> f1g1ns2.dnspod.net.	55977	IN	A	112.90.82.194
> f1g1ns2.dnspod.net.	55977	IN	A	115.236.137.40
> f1g1ns2.dnspod.net.	55977	IN	A	115.236.151.191
> f1g1ns2.dnspod.net.	55977	IN	A	182.140.167.188
> 
> ;; Query time: 1649 msec
> ;; SERVER: 192.168.0.1#53(192.168.0.1)
> ;; WHEN: Sat Oct 24 17:45:10 2015
> ;; MSG SIZE  rcvd: 294

然后登陆一台国外的主机：
> ~$ dig winterttr.me
> ; <<>> DiG 9.9.5-3ubuntu0.4-Ubuntu <<>> winterttr.me
> ;; global options: +cmd
> ;; Got answer:
> ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 37875
> ;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1
> 
> ;; OPT PSEUDOSECTION:
> ; EDNS: version: 0, flags:; udp: 1280
> ;; QUESTION SECTION:
> ;winterttr.me.			IN	A
> 
> ;; ANSWER SECTION:
> winterttr.me.		600	IN	CNAME	winterttr.github.io.
> winterttr.github.io.	854	IN	CNAME	github.map.fastly.net.
> github.map.fastly.net.	30	IN	A	103.245.222.133
> 
> ;; Query time: 173 msec
> ;; SERVER: 100.72.124.100#53(100.72.124.100)
> ;; WHEN: Sat Oct 24 09:46:29 UTC 2015
> ;; MSG SIZE  rcvd: 125


完成！~

---


{% raw %}
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.
Please refer to the author's name `winterTTr` when you remix, transform, and build upon this material. 
{% endraw %}
本作品由`winterTTr`创作，采用[知识共享署名-非商业性使用 4.0 国际许可协议](http://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可。修改，参照或者转载请注明作者`winterTTr`
