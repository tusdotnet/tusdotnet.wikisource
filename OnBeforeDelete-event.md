The OnBeforeDelete event is fired just before a file is deleted.

Calling FailRequest on the BeforeDeleteContext passed to the callback will reject the request with a 400 Bad Request status code. Calling FailRequest multiple times will concatenate the error messages.

> :information_source: Note that this event only fires for client requests and not when manually calling the store's methods.

```csharp
app.UseTus(context => new DefaultTusConfiguration
{
    UrlPath = "/files",
    Store = new TusDiskStore(@"C:\tusfiles\"),
    Events = new Events
    {
        OnBeforeDeleteAsync = ctx =>
        {
            if(!SomeBusinessLogic())
            {
                ctx.FailRequest($"Cannot delete {ctx.FileId} due to business logic");
            }
            
            return Task.CompletedTask;
        }
    }
});
```