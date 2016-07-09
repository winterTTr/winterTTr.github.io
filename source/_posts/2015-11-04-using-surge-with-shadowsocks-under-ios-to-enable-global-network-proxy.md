title: 在ios设备上使用surge配合shadowsocks开启网络全局代理
date: 2015-11-04 21:06:33
tags:
  - surge
  - ios
  - proxy
  - tun
toc_number: false
sticky: 0
categories:
  - internet
---

ios上的网络代理一直是我诟病的话题之一。
1. 手机网络不能使用代理
2. 默认的wifi下代理只能支持http

随着ios9添加的新的`Network Extension`的特性，surge([App Store](https://itunes.apple.com/cn/app/surge-web-developer-tool-proxy/id1040100637))出现了，利用surge所提供给我们的功能，我们将给出一个近乎完美的全局代理方案。

{% qnimg "2015-11-04-using-surge-with-shadowsocks-under-ios-to-enable-global-network-proxy/surge.jpg" %}

<!--more-->

# Surge是什么

Surge是一个ios网络包抓取分析工具，它是基于ios9中`Network Extension`和`VPN`的新特性而来。
Surge是一个基于`规则(Rule)`的可配置型工具，我们可以利用Surge在ios网络层中添加代理，这样可以将ios设备的很多网络数据进行截获，从而实现对网络数据的分析和采集。

如果大家在windows上经常发开http程序的话，想必著名的`fiddler`大家一定不会陌生，在利用代理进行网络数据包分析方面，Surge可以说与`fiddler`的做法是很相似的。

如果说surge的初衷，还是一个网络包抓取和分析工具的话，它的基于`规则`的数据截获和转发能力，给我们提供了非常好的网络代理功能。
如果Surge可以长久的存活下去的话，想必一定是将来一个非常有效的ios设备网络代理工具。

如果想了解surge的具体细节，可以参考官方的主页介绍 [surge.run](http://surge.run)。


# Surge配置界面
Surge并不是一个打开即可以使用的工具，它是一个高可配置的工具。
打开Surge之后，我们只有一个默认的配置文件。
我们可以看到，默认的配置文件是所有的网络都是直接进行访问(all to Direct)。

{% qnimg "2015-11-04-using-surge-with-shadowsocks-under-ios-to-enable-global-network-proxy/surge-first-open.jpg" %}

## 编辑已有配置
点击`Edit`之后，我们可以进行更多的设置
{% qnimg "2015-11-04-using-surge-with-shadowsocks-under-ios-to-enable-global-network-proxy/surge_edit.jpg" title:点击编辑 %}

对于已有的文件，我们可以更改名称，添加更多的代理，添加规则
{% qnimg "2015-11-04-using-surge-with-shadowsocks-under-ios-to-enable-global-network-proxy/surge_edit_existing config.jpg" title:编辑已有配置 %}


## 代理设置
这个是代理设置界面，在这里，可以添加三种类型的代理，`http`|`https`|`socks5`。
{% qnimg "2015-11-04-using-surge-with-shadowsocks-under-ios-to-enable-global-network-proxy/surge_proxy.jpg" title:代理配置界面 %}

我们发现，这里并没有我们所需要的`shadowsocks`。是的，`shadowsocks`在界面似乎不可以直接添加，只能通过直接编辑配置文件，添加custom类型的代理。我们随后会说到。


## 添加规则
{% qnimg "2015-11-04-using-surge-with-shadowsocks-under-ios-to-enable-global-network-proxy/surge_rule.jpg" title:规则配置界面 %}
在规则配置界面中，可以添加三种很多种不同的预定义类型的规则
- domain: 域名完全匹配
- suffix: 匹配域名的结尾部分
- keyword: 域名中包含这个关键字
- ip: 如果请求的ip地址在范围中， 使用ip加mask的形式，例如：`192.168.1.1/24`
- GeoIP: 基于ip的地理位置，例如`CN(中国)`，`US(美国)`等等

我们添加了规则之后，可以选择当数据包满足规则的时候如何处理：
- DIRECT: 也就是我们的直连，必经过任何转发，用于访问国内网络
- REJECT: 拒接链接，比较多的是拦截特定的包，例如广告请求
- [代理名称]: 默认是没有这一项的，当我们添加了不同代理之后，没一个添加的代理也会出现在这里，所以，我们可以将特定的网络请求转发到一个我们预定义的代理上面，这也是surge中本文最关心的功能。


## 网络下载配置文件
我们知道，很多时候，大家对于手工配置上文提到的这些东西，是很闲麻烦的。
所以，网络下载配置文件也是surge非常人性化的feature，我们可以通过URL直接下载surge的配置文件，让他人的东西劳动成果很快的为我们所用。
这里分享一个很不错的配置文件，源自`surge.pm`网站。

在`编辑已有配置`中，选择`Download configuration from URL`，然后添加
> http://surge.pm/main.conf

{% qnimg "2015-11-04-using-surge-with-shadowsocks-under-ios-to-enable-global-network-proxy/surge_download.jpg" title:下载配置 %}


## 更改shadowsocks配置
我们上文提到的配置文件中，已经为我们添加了shadowsocks的代理，不过，添加的代理只是例子，不能使用，我们需要利用上面提到的更改配置文件的办法，更改这个代理到我们自己的shadowsocks服务器的配置。

{% qnimg "2015-11-04-using-surge-with-shadowsocks-under-ios-to-enable-global-network-proxy/surge_shadowsocks.jpg" title:更改shadowsocks配置 %}

- server: 更改为自己的ss服务器的地址
- port: 为端口号
- Username: 直接被当成了ss的加密方式来使用了，不再作为用户名
- password: ss的登录密码

这样就可以直接使用这个配置文件进行shadowsocks的网络代理了。

# Surge的网络监控功能
我们上面提到了很多的surge的配置，当然，最原始的初衷，还是为了使用surge进行网络监控。我们可以在`Analytics`选卡中看到


## 请求解析
{% qnimg "2015-11-04-using-surge-with-shadowsocks-under-ios-to-enable-global-network-proxy/surge_request analytics.jpg" title:请求解析 %}
在请求界面中，我们可以看到具体的每个请求的header和一些相关的参数。据说作者之后会加入请求的body的显示。

## 规则分析
在`Rule`界面中，我们也可以看到规则的相应情况，而且，最人性化的，我们在这里可以动态添加规则项
{% qnimg "2015-11-04-using-surge-with-shadowsocks-under-ios-to-enable-global-network-proxy/surge_change_rule.jpg" title:动态添加规则 %}

---
经过上面的介绍，相比大家已经对surge的功能有了比较详细的了解，打开配置文件中，大部分的配置行都和我们之前介绍的界面相关，语法很容易理解。也希望大家多多分享自己的配置文件，服务于大家。

---

{% raw %}
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.
Please refer to the author's name `winterTTr` when you remix, transform, and build upon this material. 
{% endraw %}
本作品由`winterTTr`创作，采用[知识共享署名-非商业性使用 4.0 国际许可协议](http://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可。修改，参照或者转载请注明作者`winterTTr`
