title: awaiter in csharp TPL
tags:
---


<!--more-->

{% blockquote Lucian Wischik http://blogs.msdn.com/b/lucian/archive/2012/12/12/how-to-write-a-custom-awaiter.aspx "How to write a custom awaiter" %}
The normal behavior of the "await" operator on a task is to suspend execution of the method; then, when the task operand has finished, to resume execution on the same *SynchronizationContext* and with the same *ExecutionContext*.

You can replace the part in italics with your own logic by awaiting an operand of your own type (not a task) and writing a custom awaiter for it. It's not a common need. 
content.
{% endblockquote %}

## Title1



---


**sample code**



{% blockquote [author[, source]] [link] [source_link_title] %}
content
{% endblockquote %}


{% codeblock [title] [lang:language] [url] [link text] %}
code snippet
{% endcodeblock %}


{% post_path slug %}



---

**References**:

- [Async/Await FAQ](http://blogs.msdn.com/b/pfxteam/archive/2012/04/12/10293335.aspx)
- [Async CTP Refresh - design changes](http://blogs.msdn.com/b/lucian/archive/2011/04/15/async-ctp-refresh-design-changes.aspx)
- [Custom awaitables for dummies](http://stackoverflow.com/questions/12661348/custom-awaitables-for-dummies)
- [How to write a custom awaiter](http://blogs.msdn.com/b/lucian/archive/2012/12/12/how-to-write-a-custom-awaiter.aspx)
- [await anything;](http://blogs.msdn.com/b/pfxteam/archive/2011/01/13/10115642.aspx)
- [ExecutionContext vs SynchronizationContext](http://blogs.msdn.com/b/pfxteam/archive/2012/06/15/executioncontext-vs-synchronizationcontext.aspx)
- [Understanding C# async / await (2) The Awaitable-Awaiter Pattern](http://weblogs.asp.net/dixin/understanding-c-sharp-async-await-2-awaitable-awaiter-pattern)


And also:

- [Markdown Check and cross character](http://stackoverflow.com/questions/712132/in-html-i-can-make-a-checkmark-with-x2713-is-there-a-corresponding-x-mark)
