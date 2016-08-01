title: 'C# Unit Test with Microsoft Fakes'
tags:
  - 'C#'
  - Unit Test
  - Fake
date: 2015-09-05 10:43:17
categories:
  - C#
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
![stub-shim-concept-diagram.jpg](http://7xljtv.com1.z0.glb.clouddn.com/images/2015-09-05-CSharp-Unit-Test-with-Microsoft-Fakes/stub-shim-concept-diagram.jpg)



| Comparison     | Shim          | Stub        |
| -------------- | ------------- | ----------- |
| Performance | Slow ( rewrite code ) | As it is ( just interface implementation ) |
| Static method, seal type | &#10004; | &#x2717; |
| Internal Type with InternalsVisibleToAttribute| &#10004; | &#10004; |
| Private methods | &#10004; if all the types on the method signature are visible | &#x2717; |
| Interfaces and abstract methods | &#x2717; | &#10004; |


## Using Shim
Here let we say that we want to test a function in a `DemoClassLibrary`.

We have a class that implements something depends on `System` library.

```csharp
using System;

namespace DemoClassLibrary
{
    public class Foo
    {
        public long UtcNowTick()
        {
            return DateTime.UtcNow.Ticks;
        }
    }
}

```

So this `UtcNowTick` is depends on the `DateTime`, and right now we want to test this function.

This is a typical scenario for us to use `shim` to test because we cannot control the behaviour of the `system` library.

First, we add the fake to the library to we want to fake.
![add-fakes-assembly.jpg](http://7xljtv.com1.z0.glb.clouddn.com/images/2015-09-05-CSharp-Unit-Test-with-Microsoft-Fakes/add-fakes-assembly.jpg)

After this operation, we can find that we have a fake assembly created:
![after-add-fake.jpg](http://7xljtv.com1.z0.glb.clouddn.com/images/2015-09-05-CSharp-Unit-Test-with-Microsoft-Fakes/after-add-fake.jpg)

So right now, we have the fake assembly for system now, and we can control the behaviour of system namespace api as below:

```csharp
using System;
using DemoClassLibrary;
using Microsoft.QualityTools.Testing.Fakes;
using Microsoft.VisualStudio.TestTools.UnitTesting;

namespace ClassLibaryTest
{
    [TestClass]
    public class TestFoo
    {
        [TestMethod]
        public void TestUtcNowTick()
        {
            using (ShimsContext.Create())
            {
                System.Fakes.ShimDateTime.UtcNowGet = () =>
                {
                    return DateTime.MinValue;
                };


                var foo = new Foo();
                var tick = foo.UtcNowTick();

                Assert.AreEqual(tick,0);
            }
        }
    }
}
```

As the code show, the fake framework will add an extra code for your faked namespace (such as `system` in our demo), and all the extra code will under `Fakes` namespace.
We can replace function calls under `Fakes` namespace to our test implementation expected. Every type will have a `Shim+Typename` under `Fakes` for us to replace.


** There is some kinds of different scenarios which we need to test using Shim, here I list some:**

Let's say that we have a 3rd party library like this:

```csharp
namespace DemoDependence
{
    public class DependenceClass
    {
        private readonly string _data;

        public DependenceClass(string data)
        {
            _data = data;
        }


        public static string StaticMethod()
        {
            return "DependenceClass::StaticMethod ";
        }

        public string InstanceMethod()
        {
            return "DependenceClass::InstanceMethod " + _data;
        }
    }
}
```

And we have our application that use this library:
```csharp
using System;
using System.Globalization;
using DemoDependence;

namespace DemoClassLibrary
{
    public class Foo
    {
        public string UseDependencyInstanceMethod()
        {
            var instance = new DependenceClass(
                DateTime.UtcNow.ToString(CultureInfo.CurrentCulture));
            return instance.InstanceMethod();
        }

        public string UseDependencyStaticMethod()
        {
            return DependenceClass.StaticMethod();
        }

        public string UseOneDependencyInstanceMethod(DependenceClass d)
        {
            return d.InstanceMethod();
        }


        public string UseConstructor(string d)
        {
            var instance = new DependenceClass(d);
            return instance.InstanceMethod();
        }

    }
}
```

we start our test.


### shim static function

```csharp
[TestMethod]
public void ShimStaticMethod()
{
    using (ShimsContext.Create())
    {
        ShimDependenceClass.StaticMethod =
            () =>
            {
                return "my string";
            };

        var foo = new Foo();

        Assert.AreEqual(foo.UseDependencyStaticMethod() , "my string");

    }
}
```




### shim instance function
```csharp
[TestMethod]
public void ShimInstanceMethod()
{
    using (ShimsContext.Create())
    {
        ShimDependenceClass.AllInstances.InstanceMethod =
            @class =>
            {
                return "my string";
            };

        var foo = new Foo();

        Assert.AreEqual(foo.UseDependencyInstanceMethod(), "my string");
    }
}

[TestMethod]
public void ShimOneInstanceMethod()
{
    using (ShimsContext.Create())
    {
        var d = new ShimDependenceClass()
        {
            InstanceMethod = () => { return "my string"; }
        };


        var foo = new Foo();

        Assert.AreEqual(foo.UseOneDependencyInstanceMethod(d), "my string");

    }
}
```



### shim constructor
```csharp
[TestMethod]
public void ShimConstructor()
{
    using (ShimsContext.Create())
    {
        ShimDependenceClass.ConstructorString =
            (@this, s) =>
            {
                var shim = new ShimDependenceClass(@this)
                {
                    InstanceMethod = () => { return "my string"; }
                };
            };

        var foo = new Foo();

        Assert.AreEqual(
            foo.UseConstructor(string.Empty),
            "my string");

    }
}
```


## Using Stub
Stum is typically using to implement the interface, let say we have a interface:
```csharp
public interface IDependenceInterface
{
    string InterfaceMethod();
}
```

And our Foo has one more function:
```csharp
public class Foo
{
    // Hide others

    public string UseInterface(IDependenceInterface i)
    {
        return i.InterfaceMethod();
    }
}
```

The stub would be like this:
```csharp
[TestMethod]
public void StubInterface()
{
    var i = new StubIDependenceInterface()
    {
        InterfaceMethod = () => { return "my string"; }
    };


    var foo = new Foo();

    Assert.AreEqual(
        foo.UseInterface(i),
        "my string");
}
```


## Compile warning issue
When you use fake the `System` library, you may see the compilation warning, that is because some type cannot be fakes, you can change the fake configuration file to fixit.
The warning is like below:
> Warning 20 Some fakes could not be generated. For complete details, set Diagnostic attribute of the Fakes element in this file to 'true' and rebuild the project.

Just shim or stub the things you really need, here is one of my example in real project:
```xml
<Fakes xmlns="http://schemas.microsoft.com/fakes/2011/" Diagnostic="true">
  <Assembly Name="Microsoft.WindowsAzure.Configuration" Version="3.0.0.0"/>
  <StubGeneration>
    <Clear/>
  </StubGeneration>
  <ShimGeneration>
    <Clear/>
    <Add FullName="Microsoft.Azure.CloudConfigurationManager"/>
  </ShimGeneration>
  <Compilation>
    <Property Name="PlatformTarget">x64</Property>
  </Compilation>
</Fakes>
```



## Other Options

- [Moq](https://github.com/Moq/moq4) which is my favourite test framework





---

**References**:

- [Isolating Code Under Test with Microsoft Fakes](https://msdn.microsoft.com/en-us/library/hh549175.aspx)
- [Using shims to isolate your application from other assemblies for unit testing](https://msdn.microsoft.com/en-us/library/hh549176.aspx)
- [Using stubs to isolate parts of your application from each other for unit testing](https://msdn.microsoft.com/en-us/library/hh549174.aspx)


And about markdown:

- [Markdown Check and cross character](http://stackoverflow.com/questions/712132/in-html-i-can-make-a-checkmark-with-x2713-is-there-a-corresponding-x-mark)

---
{% raw %}
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.
Please refer to the author's name `winterTTr` when you remix, transform, and build upon this material. 
{% endraw %}
本作品由`winterTTr`创作，采用[知识共享署名-非商业性使用 4.0 国际许可协议](http://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可。修改，参照或者转载请注明作者`winterTTr`


