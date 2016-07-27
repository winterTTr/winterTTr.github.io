title: {{ title }}
tags:
sticky: 0
toc_number: false
categories:
---


<!--more-->



## Title1



## Title2


# sample code

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


- Center align text (Next theme plugin)
<blockquote class="blockquote-center">blah blah blah</blockquote>
{% centerquote %}blah blah blah{% endcenterquote %}
{% cq %} blah blah blah {% endcq %}

- Image exceed container (Next theme plugin)
`img 添加属性 class="full-image"即可`

<!-- HTML方式: 直接在 Markdown 文件中编写 HTML 来调用 -->
<!-- 其中 class="full-image" 是必须的 -->
<img src="/image-url" class="full-image" />

    <!-- 标签 方式，要求版本在0.4.5或以上 -->
{% fullimage /image-url, alt, title %}

<!-- 别名 -->
{% fi /image-url, alt, title %}





---

**References**:

- [Isolating Code Under Test with Microsoft Fakes](https://msdn.microsoft.com/en-us/library/hh549175.aspx)


And also:

- [Markdown Check and cross character](http://stackoverflow.com/questions/712132/in-html-i-can-make-a-checkmark-with-x2713-is-there-a-corresponding-x-mark)
