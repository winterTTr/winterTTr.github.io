title: Claims Identity in ASP.NET authentication Middleware
sticky: 0
toc_number: false
date: 2015-12-12 21:10:14
tags:
  - Claims
  - ClaimsIdentity
  - WIF
  - Authentication
  - AuthenticationMiddleware
  - ASP.NET
categories:
  - ASP.NET
---

The original motivation for this article is about following issue:
- how does the claim identity information persist after login in ASP.NET
- why does updating of claim identity in owin context not persist in further request

The `WIF(Windows Identity Foundation)` provides a Claims-Based Identity Model. And in `ASP.NET`, we can already build a Claims-Aware ASP.NET Web Application. Especially, when using with different kind of authentication middleware, `WIF` provides the same abstract layer to access the identity information across the whole asp.net pipeline context.

In this article we will talk about some detail about asp.net authentication middleware based on the `CookieAuthenticationMiddleware`. At the end, let's discuss more about persist claim in cookie across request.

<!--more-->

# Owin Middleware
Simple speaking, Owin pipeline is a link of middleware, and the request will dive into this link to the end(or short-circuited before end), and the response will pop up back though the pipeline middleware. When coming through the middleware, this gives opportunity to middleware to process and even short-circuit the whole process line.


    Request
           /=====\        /=====\        /=====\
    =====> |     | =====> |     | =====> |     |
           | MW1 |        | MW2 |        | MW3 |
    <===== |     | <===== |     | <===== |     | <== Response
           \=====/        \=====/        \=====/
    
This diagram is a very simple and straightforward explanation about the Owin Middleware.
And authentication middleware if one kind of middleware that will give their effect to the request and response process.

{% codeblock OwinMiddleware lang:csharp https://github.com/yreynhout/katana-clone/blob/ac4f4f48a3c56221faa554995d8b8c1940b5f838/src/Microsoft.Owin/OwinMiddleware.cs OwinMiddleware.cs %}
using System.Threading.Tasks;
namespace Microsoft.Owin
{
    public abstract class OwinMiddleware
    {
        protected OwinMiddleware(OwinMiddleware next)
        {
            Next = next;
        }
        protected OwinMiddleware Next { get; set; }
        public abstract Task Invoke(IOwinContext context);
    }
}
{% endcodeblock %}

Here is the definition the `OwinMiddleware`, there is two key pieces in code:
1. next: reference to the next middleware in pipeline 
2. `Invoke` abstract function: this is the how this middleware could perform on the owin conext. The usually pattern would be "Before action" + "next.Invoke()" + "after action".



# AuthenticationMiddleware
`AuthenticationMiddleware` is the base class for all authentication middleware. And it implement the basic pattern for the `Invoke` function for a concrete authentication middleware. Here is the flow:

![cookie_authentication_middleware_invoke.jpg](https://winterttrgithubio.blob.core.windows.net/images/2015-12-12-Claim-Identity-In-AspNet-Authentication-Middleware/cookie_authentication_middleware_invoke.jpg)

And more detail code is as below:
{% codeblock AuthenticationMiddleware lang:csharp https://github.com/yreynhout/katana-clone/blob/ac4f4f48a3c56221faa554995d8b8c1940b5f838/src/Microsoft.Owin.Security/Infrastructure/AuthenticationMiddleware.cs AuthenticationMiddleware.cs %}
public override async Task Invoke(IOwinContext context)
{
    AuthenticationHandler<TOptions> handler = CreateHandler();
    await handler.Initialize(Options, context);
    if (!await handler.InvokeAsync())
    {
        await Next.Invoke(context);
    }
    await handler.TeardownAsync();
}
{% endcodeblock %}

So we can see that, the concrete authentication middleware will create an authentication handler and use that handler for further processing. The main logic for a authentication middleware is mainly about how the handler is implemented.
So let's see the peice of code for `CreateHandler` in `CookieAuthenticationMiddleware`:
{% codeblock CookieAuthenticationMiddleware lang:csharp https://github.com/yreynhout/katana-clone/blob/ac4f4f48a3c56221faa554995d8b8c1940b5f838/src/Microsoft.Owin.Security.Cookies/CookieAuthenticationMiddleware.cs CookieAuthenticationMiddleware.cs %}
protected override AuthenticationHandler<CookieAuthenticationOptions> CreateHandler()
{
    return new CookieAuthenticationHandler(_logger);
}
{% endcodeblock %}



# AuthenticationHandler
So the main logic of authentication middleware will be focus on authentication handler. Let see a deeper graph when we expand the `AuthenticationHandler` level:

![authentication_handler.jpg](https://winterttrgithubio.blob.core.windows.net/images/2015-12-12-Claim-Identity-In-AspNet-Authentication-Middleware/authentication_handler.jpg)

The blue part will be the main part of how `AuthenticationHandler` works. And the red part will be how a specific authentication handler, here is `CookieAuthenticationHandler` implemented.


# CookieAuthenticatonHandler
We do not have the plan to dive into every detail of how CookieAuthenticationHandler implemented. So we only focus to original issue, how claim identity does the persistence underlying.

In fact, originally, i thought the claim identity which we can access via `User.Identity` will be persist in cookie. So if I update the claims in `User` context, everything will be persist ( i mean update ) into the cookie later. So I can access the updated value in later request.

However, the thing is not as I wish.

Here is the expanded version for `CookieAuthenticationHandler`:
![teardown.jpg](https://winterttrgithubio.blob.core.windows.net/images/2015-12-12-Claim-Identity-In-AspNet-Authentication-Middleware/teardown.jpg)

The `TeardownAsync` will perform the persistence logic from the identity, but not like we wish. From the diagram, we found that findally, the `CookieAuthenticationHandler` use the `ApplyResponseGrantAsync` to save information. And what is `Grant`?

We can refer to more detail from the implementation:
{% codeblock CookieAuthenticationHandler lang:csharp https://github.com/yreynhout/katana-clone/blob/ac4f4f48a3c56221faa554995d8b8c1940b5f838/src/Microsoft.Owin.Security.Cookies/CookieAuthenticationHandler.cs#L81 CookieAuthenticationHandler.cs %}
protected override async Task ApplyResponseGrantAsync()
{
    AuthenticationResponseGrant signin = Helper.LookupSignIn(Options.AuthenticationType);
    bool shouldSignin = signin != null;

    // <<< ignore some code >>>

    if (shouldSignin)
    {
        var context = new CookieResponseSignInContext(
            Context,
            Options,
            Options.AuthenticationType,
            signin.Identity,
            signin.Properties);

        // <<< ignore some code >>>

        Response.Cookies.Append(
            Options.CookieName,
            cookieValue,
            cookieOptions);
    }

    // <<< ignore some code >>>
}
{% endcodeblock %}
From the code, two things need to pay attention to:
1. the handler will save the `grant` into cookie
2. the action in step 1 will only happen when we can have this `grant`

Ok, let's find the final piece about this `grant`

{% codeblock SecurityHelper lang:csharp https://github.com/yreynhout/katana-clone/blob/ac4f4f48a3c56221faa554995d8b8c1940b5f838/src/Microsoft.Owin.Security/Infrastructure/SecurityHelper.cs#L102 SecurityHelper.cs %}
public AuthenticationResponseGrant LookupSignIn(string authenticationType)
{
    if (authenticationType == null)
    {
        throw new ArgumentNullException("authenticationType");
    }

    AuthenticationResponseGrant grant = _context.Authentication.AuthenticationResponseGrant;
    if (grant == null)
    {
        return null;
    }

    foreach (var claimsIdentity in grant.Principal.Identities)
    {
        if (string.Equals(authenticationType, claimsIdentity.AuthenticationType, StringComparison.Ordinal))
        {
            return new AuthenticationResponseGrant(claimsIdentity, grant.Properties ?? new AuthenticationProperties());
        }
    }

    return null;
}
{% endcodeblock %}

So we finally found where the data comes from - **Authentication.AuthenticationResponseGrant**.


# AuthenticationResponseGrant
`Grant` is the concept of the information that we retreive after we do the authentication. If the cookie handler save this **grant** into the cookie, we need to find where is grant come from. 
{% codeblock AuthenticationManager lang:csharp https://github.com/yreynhout/katana-clone/blob/ac4f4f48a3c56221faa554995d8b8c1940b5f838/src/Microsoft.Owin/Security/AuthenticationManager.cs#L200 AuthenticationManager.cs %}
public void SignIn(AuthenticationProperties properties, params ClaimsIdentity[] identities)
{
    AuthenticationResponseRevoke priorRevoke = AuthenticationResponseRevoke;
    if (priorRevoke != null)
    {
        // Scan the sign-outs's and remove any with a matching auth type.
        string[] filteredSignOuts = priorRevoke.AuthenticationTypes
            .Where(authType => !identities.Any(identity => identity.AuthenticationType.Equals(authType, StringComparison.Ordinal)))
            .ToArray();
        if (filteredSignOuts.Length < priorRevoke.AuthenticationTypes.Length)
        {
            if (filteredSignOuts.Length == 0)
            {
                AuthenticationResponseRevoke = null;
            }
            else
            {
                AuthenticationResponseRevoke = new AuthenticationResponseRevoke(filteredSignOuts);
            }
        }
    }

    AuthenticationResponseGrant priorGrant = AuthenticationResponseGrant;
    if (priorGrant == null)
    {
        AuthenticationResponseGrant = new AuthenticationResponseGrant(new ClaimsPrincipal(identities), properties);
    }
    else
    {
        ClaimsIdentity[] mergedIdentities = priorGrant.Principal.Identities.Concat(identities).ToArray();

        if (properties != null)
        {
            // Update prior properties
            foreach (var propertiesPair in properties.Dictionary)
            {
                priorGrant.Properties.Dictionary[propertiesPair.Key] = propertiesPair.Value;
            }
        }

        AuthenticationResponseGrant = new AuthenticationResponseGrant(new ClaimsPrincipal(mergedIdentities), priorGrant.Properties);
    }
}
{% endcodeblock %}

So in fact, this `grant` information is set when we call the `SignIn` function of the authenticaton manager. And if we do not call `SignIn`, the **AuthenticationResponseGrant** will be null. In that case, the cookie will not be updated.

And the `User.Identity` will **NOT** set to this authentication grant if `SignIn` is not called. That is why even update the `User.Identity` but those updated information will not take effect for any further request in asp.net.

This is also explain every detail about this stackoverflow question [Persisting claims across requests](http://stackoverflow.com/questions/25292137/persisting-claims-across-requests)

# How to Persist Claim
After dive into such more detail from code level, let's discuss about the original topic.

I found a solution here from the stackoverflow [How to update a claim in ASP.NET Identity?](http://stackoverflow.com/questions/24587414/how-to-update-a-claim-in-asp-net-identity), this solution will set the `AuthenticationResponseGrant` explicitly when try to update the claim items, this will trigger the sign in logic when `Teardown` is called.

In fact, this is one of the solution that indeed take effect. But it has a potential issue, that is, in fact, such kind of code will trigger the **sign in** logic underlying, and the **sign in** logic will update the cookie expiration time. This behavior works as you sigin in again when you explicitly set the `AuthenticationResponseGrant`. Of course, if the cookie expiration time is not a problem for your business logic, espeically when you use the [SlidingExpiration](https://github.com/yreynhout/katana-clone/blob/ac4f4f48a3c56221faa554995d8b8c1940b5f838/src/Microsoft.Owin.Security.Cookies/CookieAuthenticationOptions.cs#L85), everyting should be fine.

But for my business which use the cookie authentication expiration as the web application login expiration. This will be a problem, and i should not use this way to update claim information in cookie.

So based on the implementation of Katana( i do not read the latest code, so correct me if i miss something), the identity saved in cookie is not designed for dynamically update. Those claim identity in the cookie is represented as a unique identity information that **binding to a specific login** and it should be immutable. And, updating the claim data in cookie with `AuthenticationResponseGrant` means to a totally new login instead of updating the current login context. So all the claim data that is persisted in the cookie should not be updated within a specific login session. So that is why we can found that most of the authenticaton middleware save the claim identity information belong to the login itself to the cookie.

Yes, you may say that, how about those that do not belong to so-called "login information", where should I save it? In fact, within asp.net project template the microsoft already provide us the way when you select "Individual Use Account".

![individual_user_account.jpg](https://winterttrgithubio.blob.core.windows.net/images/2015-12-12-Claim-Identity-In-AspNet-Authentication-Middleware/individual_user_account.jpg)

I do not want to dive more code here, the magic is `ApplicationUserManager`. This manager is using `UserStore` binding to a entityframework `DbContext` object. We can use the user manager's `AddClaimAsync`|`GetClaimAsync`|`RemoveClaimAsync` to manipulate the claim data we want to save. The user manager will sync those saved claim everytime when the next query is comming, like what we wish. And yes, i think you already guess the conclusion, user manager is using the database to save the claim identity information underlying.

And one more option, if you just want to save some data between request, and those information is not a sensitive data, such as access code, you can use the `Request.Cookie` directly, and you can see those data from chrome debug panel under `Resource` tab directly. More simple, right?


Ok, claim identity in asp.net is so a big topic so i would talk more aspect on other article, let's end here.
Any question, please just reply and discuss with me :-)


---

**References**:
- [Persisting claims across requests](http://stackoverflow.com/questions/25292137/persisting-claims-across-requests)
- [Server side claims caching with Owin Authentication](http://stackoverflow.com/questions/19192428/server-side-claims-caching-with-owin-authentication)
- [How to update a claim in ASP.NET Identity?](http://stackoverflow.com/questions/24587414/how-to-update-a-claim-in-asp-net-identity)




---
