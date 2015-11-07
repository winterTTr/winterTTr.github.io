title: Enable ASP.NET Web API detail error message in release
date: 2015-09-07 16:24:40
tags:
  - C#
  - ASP.NET
---

After deploy the asp.net web api application, I found that when a error (exception) occurs, the return message would only be

```json
{
    "Message": "An error has occurred."
}
```

But when in local debug, the message would be more detail:

```json
{
    "Message": "An error has occurred.",
    "ExceptionMessage": "The remote server returned an error: (409).....",
    "ExceptionType": "System.Net.WebException",
    "StackTrace": "   at System.Net.HttpWebRequest.EndGetResponse(IAsyncResult asyncResult)...."
}
```

Theoretically, you should not show those detail information to your final user, no matter for detail code security, or user experience.
But when debugging our cloud service after deploying, we may need to figure out what happened in backend.

In fact, ASP.NET web api has a separate configuration for how the error detail is shown in different environments.

In your `HttpConfiguration`, there is a property called `IncludeErrorDetailPolicy`. Let's check its possible value.

```csharp
// Summary:
//     Specifies whether error details, such as exception messages and stack traces,
//     should be included in error messages.
public enum IncludeErrorDetailPolicy
{
    // Summary:
    //     Use the default behavior for the host environment. For ASP.NET hosting, use
    //     the value from the customErrors element in the Web.config file. For self-hosting,
    //     use the value System.Web.Http.IncludeErrorDetailPolicy.LocalOnly.
    Default = 0,
    //
    // Summary:
    //     Only include error details when responding to a local request.
    LocalOnly = 1,
    //
    // Summary:
    //     Always include error details.
    Always = 2,
    //
    // Summary:
    //     Never include error details.
    Never = 3,
}
```

Its comments has already show the detail, so for my case:
```csharp
public class Startup
{
    public void Configuration(IAppBuilder app)
    {
        app.UseCloudServiceGateway();

        var config = new HttpConfiguration
        {
            IncludeErrorDetailPolicy = IncludeErrorDetailPolicy.Always // Add this line to enable detail mode in release
        };
        WebApiConfig.Register(config);
        app.UseWebApi(config);
    }
}
```

Done~~

---
{% raw %}
<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.
Please refer to the author's name `winterTTr` when you remix, transform, and build upon this material. 
{% endraw %}
本作品由`winterTTr`创作，采用[知识共享署名-非商业性使用 4.0 国际许可协议](http://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可。修改，参照或者转载请注明作者`winterTTr`
