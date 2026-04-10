The `OnAuthorizeAsync` event is the first event to fire once a request has been identified as a tus request. It is useful for fine-grained authorization that goes beyond what ASP.NET Core's built-in auth handles, such as verifying that the file being accessed belongs to the current user.

Because it runs on every tus request regardless of intent, it can also be used as a general "request begin" hook for things like logging or setting up trace context.

Calling `FailRequest` on the context will reject the request with the provided HTTP status code and message.

```csharp
app.MapTus("/files", httpContext => new DefaultTusConfiguration
{
    Store = new TusDiskStore(@"C:\tusfiles\"),
    Events = new Events
    {
        OnAuthorizeAsync = eventContext =>
        {
            if (!eventContext.HttpContext.User.Identity.IsAuthenticated)
            {
                // Note: ASP.NET Core will automatically authenticate the user using the default authentication scheme.
                // If this is not the scheme you wish to use, call AuthenticationHttpContextExtensions.AuthenticateAsync
                // here to authenticate the current request using your preferred scheme.
                eventContext.FailRequest(HttpStatusCode.Unauthorized);
                return Task.CompletedTask;
            }

            if (eventContext.HttpContext.User.Identity.Name != "test")
            {
                eventContext.FailRequest(HttpStatusCode.Forbidden, "'test' is the only allowed user");
                return Task.CompletedTask;
            }

            // The intent tells you what the client is trying to do, allowing you to apply
            // different authorization rules per operation.
            switch (eventContext.Intent)
            {
                case IntentType.CreateFile:
                    break;
                case IntentType.ConcatenateFiles:
                    break;
                case IntentType.WriteFile:
                    break;
                case IntentType.DeleteFile:
                    break;
                case IntentType.GetFileInfo:
                    break;
                case IntentType.GetOptions:
                    break;
            }

            return Task.CompletedTask;
        }
    }
});
```
