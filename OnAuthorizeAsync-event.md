The OnAuthorize event is the first event to fire once a request has been determined to be a tus request. This event allows the request to be authorized for the given intent.

Calling FailRequest on the OnAuthorizeContext passed to the callback will reject the request with the provided http status code and status message.

```csharp
app.UseTus(httpContext => new DefaultTusConfiguration
{
    UrlPath = "/files",
    Store = new TusDiskStore(@"C:\tusfiles\"),
    Events = new Events
    {
        OnAuthorizeAsync = eventContext => 
        {
            if (!eventContext.HttpContext.User.Identity.IsAuthenticated) 
            {
                eventContext.FailRequest(HttpStatusCode.Unauthorized);
                return Task.CompletedTask;
            }

            // Do other verification on the user; claims, roles, etc. In this case, check the username.
            if (eventContext.HttpContext.User.Identity.Name != "test") 
            {
                eventContext.FailRequest(HttpStatusCode.Forbidden, "'test' is the only allowed user");
                return Task.CompletedTask;
            }

            // Verify different things depending on the intent of the request.
            // E.g.:
            //   Does the file about to be written belong to this user?
            //   Is the current user allowed to create new files or have they reached their quota?
            //   etc etc
            switch (ctx.Intent) {
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
                default:
                    break;
            }

            return Task.CompletedTask;
        }
    }
});

```