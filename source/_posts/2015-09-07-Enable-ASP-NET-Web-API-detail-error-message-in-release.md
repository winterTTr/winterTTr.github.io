title: Enable ASP.NET Web API detail error message in release
date: 2015-09-07 16:24:40
tags:
  - C#
  - ASP.NET
---

After delploy the asp.net web api application, I found that when a error (exception) occurs, the return message would only be

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
