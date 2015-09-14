title: 'C# Diagnostics with ETW'
tags:
---


<!--more-->


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




---

**References**:

- [EventSource Class](https://msdn.microsoft.com/en-us/library/system.diagnostics.tracing.eventsource(v=VS.110).aspx)
- [Introduction Tutorial: Logging ETW events in C#: System.Diagnostics.Tracing.EventSource](http://blogs.msdn.com/b/vancem/archive/2012/07/09/logging-your-own-etw-events-in-c-system-diagnostics-tracing-eventsource.aspx)
- [The specification for the System.Diagnostics.Tracing.EventSource class](http://blogs.msdn.com/b/vancem/archive/2012/07/09/more-details-on-eventsource-the-class-specification.aspx)
- [Windows high speed logging: ETW in C#/.NET using System.Diagnostics.Tracing.EventSource](http://blogs.msdn.com/b/vancem/archive/2012/08/13/windows-high-speed-logging-etw-in-c-net-using-system-diagnostics-tracing-eventsource.aspx)
- [ETW in C#: Controlling which events get logged in an System.Diagnostics.Tracing.EventSource](http://blogs.msdn.com/b/vancem/archive/2012/08/14/etw-in-c-controlling-which-events-get-logged-in-an-system-diagnostics-tracing-eventsource.aspx)
- [Cloud Diagnostics - Take Control of Logging and Tracing in Windows Azure](https://msdn.microsoft.com/magazine/ff714589.aspx)
- [Vance Morrison's Weblog](http://blogs.msdn.com/b/vancem/)


And also:

- [Markdown Chech and cross character](http://stackoverflow.com/questions/712132/in-html-i-can-make-a-checkmark-with-x2713-is-there-a-corresponding-x-mark)
