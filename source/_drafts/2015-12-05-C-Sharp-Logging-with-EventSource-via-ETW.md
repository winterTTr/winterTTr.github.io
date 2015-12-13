title: 'C# Logging with EventSource via ETW'
tags:
  - ETW
  - Logging
  - EventSource
  - 'C#'
sticky: 0
toc_number: false
categories:
  - 'C#'
date: 2015-12-05 22:36:07
---


Logging is the always a fundamental diagnostics capacity that we could leverage to debug our program. Comparing to the traditional `printf` style logging, C# have already built in a great logging capacity which we could use as more modern and structured logging way. That is `EventSource`, which use `ETW` (Event Tracing for Windows) as underlying technical. And we will talk about logging with `EventSource` in this article.


<!--more-->

# What is ETW?

Before we have any discusion about `EventSource`, let's have a glance about what is `ETW`.
Here is some defination from MSDN about `ETW`:

{% blockquote "MSDN" "https://msdn.microsoft.com/en-us/library/windows/hardware/ff545699(v=vs.85).aspx" "Link"  %}
Event Tracing for Windows (ETW) provides a mechanism to trace and log events that are raised by user-mode applications and kernel-mode drivers. ETW is implemented in the Windows operating system and provides developers a fast, reliable, and versatile set of event tracing features.
{% endblockquote %}

Another one:

{% blockquote "MSDN Blog" "http://blogs.msdn.com/b/ntdebugging/archive/2009/08/27/etw-introduction-and-overview.aspx" "Link"  %}
Event Tracing for Windows (ETW) is a system and software diagnostic, troubleshooting and performance monitoring component of Windows that has been around since Windows 2000. However, it wasn't until Windows Vista that major components of the OS were updated to heavily use ETW tracing; making it much more practical and useful.
ETW is useful from a variety of scenarios, including:
 -User & Admin: Being in control of your system and knowing what is going on. 
 -User & Admin: Troubleshooting performance, hardware and OS issues.
 -Enthusiast: Learning further about the OS and the low level guts of the OS.
 -Software Developer/ISV/OEM: Investigating issues with your software's interaction with Microsoft OS & technologies
 -Hardware Developer/IHV/OEM: Investigating issues with hardware interaction with the OS, including kernel, driver subsystems, up to the user stack.
ETW is a set of technologies and tools that can absolutely complement existing tools while providing a look into the guts of the OS at a very low level.
{% endblockquote %}

Here is a good summary:

{% blockquote "MSDN Blog" "http://blogs.msdn.com/b/oanapl/archive/2009/08/05/etw-event-tracing-for-windows-what-it-is-and-useful-tools.aspx" "Link"  %}
A general-purpose, high-speed tracing facility provided by the operating system. Using a buffering and logging mechanism implemented in the kernel, ETW provides a tracing mechanism for events raised by both user-mode applications and kernel-mode device drivers. Additionally, ETW gives you the ability to enable and disable logging dynamically, making it easy to perform detailed tracing in production environments without requiring reboots or application restarts. The logging mechanism uses per-processor buffers that are written to disk by an asynchronous writer thread. This allows large-scale server applications to write events with minimum disturbance.
{% endblockquote %}


So based on those definiation from msdn, `ETW` is a Windows operation system build-in feature. It provides a logging ability with good performance both for user application and kernel-mode device drivers. 


# What is EventSource?

Also from MSDN:

{% blockquote "MSDN - EventSource Class" "https://msdn.microsoft.com/en-us/library/system.diagnostics.tracing.eventsource(v=vs.110).aspx" "Link"  %}
This class is intended to be inherited by a user class that provides specific events to be used for ETW.
{% endblockquote %}

Yes, simple and direct, `EventSource` is the entry point from where we can start to use the ablitity that provided by underlying ETW. And in this article, we will talk about more some detail about how we use `EventSource` as well as potential point we need to pay attention to.


# Why ETW?
Before we talk about what `EventSource` can provide to us, let's simple discuss why we prefer `EventSource` comparing traditional `printf` style logging.

In fact, we have two categories of logging system:

- **weakly typed logging**
This logging is the traditonal `printf` style logging. The logging system only knows the how to log string and thus all the data is formatted into a final string before pass to the logging system. It is simple and direct, also most of the logging framework support this kind of logging system.

- **strong typed Logging (also know as `Semantic Logging`)**
In such kind of logging system, every logging item is not a simple string. Instead, it is a data set with schema defined. So we can fetch every field of the logging item back when we need. Here, we call those logging items as `event`. And usually, every event will have some common fields, such as `event name`, `event id`, `timestamp` and etc, plus some extra user-defined fields, which we called `payload`. So every event will have a `common fields` + `payload fields`.

The most important difference between `weakly typed logging` and `strong typed logging` is how the consumer process the logging data. `weakly typed logging` need to format the pieces of information into string and when we need those information, we need to write another parser to parse those information back to pieces. This is no effecient at all. And especially, if the author of the parser is not the author of original logging string, the parser usually need to *guess* what those information is, so this makes the parser every fragile. And usually the `weak typed logging` is comsumed by human as we can easily understand it. But that is totally not true for for any automation process tools. And all of those issues can be properly handled by `string typed logging`. 

`ETW` built into the windows system is just an good example of `strong typed logging` system.

# How to use EventSource


> typeperf -q -o "C:\Temp\counters.txt"


The **Event** attribute includes the event identifier as its first parameter. Every other parameter in this attribute is optional; the only required parameter is the event identifier

You can use the **EventSource** attribute on the class itself to provide a more user-friendly name for this event source, which will be used when you save log messages. If you omit the EventSource attribute, the class name is used as the name of the event source.


Note that there is a recommended naming convention for event source classes. The name should start with your company name and, if you expect to have more than one event source for your company, the name should include categories separated by a hyphen. Microsoft follows this convention; for example, there is an event source called `Microsoft-Windows-DotNetRuntime`. You should carefully consider your own naming strategy because, if you change it in the future, users consuming the events will no longer receive them.



Each event is defined using a method that wraps a call to the `WriteEvent` method in the `EventSource` class. The first parameter of the `WriteEvent` method is an event identifier that must be unique within the event source. Different overloaded versions of this method, which must use different event identifiers, enable you to write additional information to the log.


Use the `Message` attribute to give the message format and call `WriteEvent` to match that format:

```chsarp
[Event(3,
       Message = "loading page {1} activityID={0}",
       Opcode = EventOpcode.Start,
       Task = Tasks.Page, Keywords = Keywords.Page,
       Level = EventLevel.Informational)]
internal void PageStart(int ID, string url)
{
  if (this.IsEnabled()) this.WriteEvent(3, ID, url);
}
```


## Log Level
You should use log levels with values lower than Informational (such as Warning, Error and Critical) for relatively rare warnings or errors. When in doubt, use the default of Informational level. Use the Verbose level for events that can happen more than 1,000 times per second. Typically, users filter by severity level first and then, if necessary, refine the filter by using keywords.


## Log Group
You can use the optional `keywords` capability to define different groups of events so that, when you enable an event source, you can specify which groups of events to enable.


Each keyword value is a 64-bit integer, which is treated as a bit array (a few of the highest bits are reserved, but it still allows you to define a sufficient number of different keywords). You must ensure you assign powers of two as the value for each constant. 


## Checking an event source for errors
`Semantic Logging Application Block - Event Source Analyzer`




## Event vs Log



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







# Q&A

## Sometimes the ETW log do not transfer to the local storage table
MonAgent.exe cannot be started by MonAgentHost.exe.



---

**References**:

- [EventSource Class](https://msdn.microsoft.com/en-us/library/system.diagnostics.tracing.eventsource(v=VS.110).aspx)
- [Introduction Tutorial: Logging ETW events in C#: System.Diagnostics.Tracing.EventSource](http://blogs.msdn.com/b/vancem/archive/2012/07/09/logging-your-own-etw-events-in-c-system-diagnostics-tracing-eventsource.aspx)
- [The specification for the System.Diagnostics.Tracing.EventSource class](http://blogs.msdn.com/b/vancem/archive/2012/07/09/more-details-on-eventsource-the-class-specification.aspx)
- [Windows high speed logging: ETW in C#/.NET using System.Diagnostics.Tracing.EventSource](http://blogs.msdn.com/b/vancem/archive/2012/08/13/windows-high-speed-logging-etw-in-c-net-using-system-diagnostics-tracing-eventsource.aspx)
- [ETW in C#: Controlling which events get logged in an System.Diagnostics.Tracing.EventSource](http://blogs.msdn.com/b/vancem/archive/2012/08/14/etw-in-c-controlling-which-events-get-logged-in-an-system-diagnostics-tracing-eventsource.aspx)
- [Cloud Diagnostics - Take Control of Logging and Tracing in Windows Azure](https://msdn.microsoft.com/magazine/ff714589.aspx)
- [Vance Morrison's Weblog](http://blogs.msdn.com/b/vancem/)
- [HOW ARE EVENT PARAMETERS BEST USED TO CREATE AN INTUITIVE CUSTOM EVENTSOURCETRACE](http://blogs.msmvps.com/kathleen/2014/01/24/how-are-event-parameters-best-used-to-create-an-intuitive-custom-evnetsourcetrace/)
- [Using .NET 4.5.1, how do I use some of the non-intuitive properties provided by ETW?](http://stackoverflow.com/questions/21292547/using-net-4-5-1-how-do-i-use-some-of-the-non-intuitive-properties-provided-by)

Read more:
- [About Event Tracing](https://msdn.microsoft.com/en-us/library/windows/desktop/aa363668(v=vs.85).aspx)
- [Vance Morrison's Weblog](http://blogs.msdn.com/b/vancem/)
- [Semantic Logging 2.0](https://msdn.microsoft.com/en-us/library/dn775006.aspx)
- [Developing event sources using the .NET EventSource class](https://msdn.microsoft.com/en-us/library/dn774985(v=pandp.20).aspx)
- [Azure Table Storage event sink](https://msdn.microsoft.com/en-us/library/dn775010(v=pandp.20).aspx)
- [Microsoft Azure Diagnostics](http://justazure.com/microsoft-azure-diagnostics-part-4-custom-logging-components-azure-diagnostics-1-3-changes/)

