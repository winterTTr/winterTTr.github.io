title: {{ title }}
tags:
sticky: 0
toc_number: false
---


<!--more-->



## Title1



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

- [Markdown Check and cross character](http://stackoverflow.com/questions/712132/in-html-i-can-make-a-checkmark-with-x2713-is-there-a-corresponding-x-mark)
