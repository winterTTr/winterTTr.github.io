title: 'C# Unit Test with Microsoft Fakes'
tags:
  - 'C#'
  - Unit Test
  - Fake
date: 2015-09-05 10:43:17
---

In common C# Unit test, we always meet the problem that our final application is depending on many 3rd party or system references. But for unit test, we need to separate and control the dependency libraries behaviour. One of the solution will be `Microsoft Fake`.

Here let me talk about Microsoft Fake , and how to unit test your application which depends on many system or other references.

<!-- more -->


## Microsoft Fake

**Microsoft Fakes** is a test framework help you isolate the code you are testing by replacing other parts of the application.
**Microsoft Fake** is highly integrated with Visual Studio and you can easily start to use it with very several clicks.

Fakes come in two flavours:

- A `shim` modifies the compiled code of your application at run time so that instead of making a specified method call, it runs the shim code that your test provides. Shims can be used to replace calls to assemblies that you cannot modify, such .NET assemblies. 

- A `stub` replaces a class with a small substitute that implements the same interface. To use stubs, you have to design your application so that each component depends only on interfaces, and not on other components. 

A good diagram to distinguish those two:
![Shim and Stub](http://7xljtv.com1.z0.glb.clouddn.com/stub%20shim%20concept%20diagram.png)


| Comparison     | Shim          | Stub        |
| -------------- | ------------- | ----------- |
| Performance | Slow ( rewrite code ) | As it is ( just interface implementation ) |
| Static method, seal type | &#10004; | &#x2717; |
| Internal Type with InternalsVisibleToAttribute| &#10004; | &#10004; |
| Private methods | &#10004; if all the types on the method signature are visible | &#x2717; |
| Interfaces and abstract methods | &#x2717; | &#10004; |


## Using Shim
Coming soon...


## Using Stub
Coming soon...


## Compile warning issue
Coming soon...


## Other Options

- [Moq](https://github.com/Moq/moq4) which is my favorate test framework





---

**References**:

- [Isolating Code Under Test with Microsoft Fakes](https://msdn.microsoft.com/en-us/library/hh549175.aspx)
- [Using shims to isolate your application from other assemblies for unit testing](https://msdn.microsoft.com/en-us/library/hh549176.aspx)
- [Using stubs to isolate parts of your application from each other for unit testing](https://msdn.microsoft.com/en-us/library/hh549174.aspx)


And about markdown:

- [Markdown Chech and cross character](http://stackoverflow.com/questions/712132/in-html-i-can-make-a-checkmark-with-x2713-is-there-a-corresponding-x-mark)
