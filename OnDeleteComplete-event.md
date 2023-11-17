The OnDeleteComplete event is fired once a file has been deleted.

> :information_source: Note that this event only fires for client requests and not when manually calling the store's methods.

```csharp
app.UseTus(context => new DefaultTusConfiguration
{
    UrlPath = "/files",
    Store = new TusDiskStore(@"C:\tusfiles\"),
    Events = new Events
    {
        OnDeleteCompleteAsync = ctx =>
        {
            logger.LogInformation($"Deleted file {ctx.FileId} using {ctx.Store.GetType().FullName}");
            return Task.CompletedTask;
        }
    }
});
```