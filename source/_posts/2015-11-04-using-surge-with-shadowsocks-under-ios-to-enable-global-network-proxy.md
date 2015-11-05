title: 在ios设备上使用surge配合shadowsocks开启网络全局代理
date: 2015-11-04 21:06:33
tags:
  - surge
  - ios
  - proxy
  - tun
toc_number: false
sticky: 0
---

ios上的网络代理一直是我诟病的话题之一。
1. 手机网络不能使用代理
2. 默认的wifi下代理只能支持http

随着ios9添加的新的`Network Extension`的特性，surge([App Store](https://itunes.apple.com/cn/app/surge-web-developer-tool-proxy/id1040100637))出现了，利用surge所提供给我们的功能，我们将给出一个近乎完美的全局代理方案。

{% qnimg "2015-11-04-using-surge-with-shadowsocks-under-ios-to-enable-global-network-proxy/surge.JPG" extend:-wm.h400 %}

<!--more-->

# Surge是什么

Surge是一个ios网络包抓取分析工具，它是基于ios9中`Network Extension`和`VPN`的新特性而来。
Surge是一个基于`规则(Rule)`的可配置型工具，我们可以利用Surge在ios网络层中添加代理，这样可以将ios设备的很多网络数据进行截获，从而实现对网络数据的分析和采集。

如果大家在windows上经常发开http程序的话，想必著名的`fiddler`大家一定不会陌生，在利用代理进行网络数据包分析方面，Surge可以说与`fiddler`的做法是很相似的。

如果说surge的初衷，还是一个网络包抓取和分析工具的话，它的基于`规则`的数据截获和转发能力，给我们提供了非常好的网络代理功能。
如果Surge可以长久的存活下去的话，想必一定是将来一个非常有效的ios设备网络代理工具。

如果想了解surge的具体细节，可以参考官方的主页介绍 [surge.run](http://surge.run)。


# Surge的配置
Surge并不是一个打开即可以使用的工具，它是一个高可配置的工具。
打开Surge之后，我们只有一个默认的配置文件。


{% qnimg "2015-11-04-using-surge-with-shadowsocks-under-ios-to-enable-global-network-proxy/surge-first-open.jpg" extend:-wm.black.h600 %}



## Title2



# sample code

- qiniu picture
{% qnimg test/demo.png title:title alt:description %}
{% qnimg test/demo.png title:title alt:description extend:-watermark.black %}

- quote
{% blockquote [author[, source]] [link] [source_link_title] %}
content
{% endblockquote %}

- Code Block
{% codeblock [title] [lang:language] [url] [link text] %}
code snippet
{% endcodeblock %}

- jsFiddle
{% jsfiddle shorttag [tabs] [skin] [width] [height] %}

- Gist
{% gist gist_id [filename] %}

- Link
{% link text url [external] [title] %}

- Include Code
{% include_code [title] [lang:language] path/to/file %}


- YouTube
{% youtube video_id %}

- Vimeo
{% vimeo video_id %}

- Include Posts
{% post_path slug %}
{% post_link slug [title] %}


- Include Assets
{% asset_path slug %}
{% asset_img slug [title] %}
{% asset_link slug [title] %}


- Raw
{% raw %}
content
{% endraw %}





---

**References**:

- [Isolating Code Under Test with Microsoft Fakes](https://msdn.microsoft.com/en-us/library/hh549175.aspx)


And also:

- [Markdown Chech and cross character](http://stackoverflow.com/questions/712132/in-html-i-can-make-a-checkmark-with-x2713-is-there-a-corresponding-x-mark)

---

{% raw %}
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.
Please refer to the auther's name `winterTTr` when you remix, transform, and build upon this material. 
{% endraw %}
本作品由`winterTTr`创作，采用[知识共享署名-非商业性使用 4.0 国际许可协议](http://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可。修改，参照或者转载请注明作者`winterTTr`
