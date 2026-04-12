The OnDeleteComplete event is fired once a file has been deleted.

> :information_source: Note that this event only fires for client requests and not when manually calling the store's methods.

```csharp
app.MapTus("/files", context => new DefaultTusConfiguration
{
    Store = new TusDiskStore(@"C:\tusfiles\"),
    Events = new Events
    {
        OnDeleteCompleteAsync = ctx =>
        {
            var logger = ctx.HttpContext.RequestServices.GetRequiredService<ILogger<Program>>();
            logger.LogInformation($"Deleted file {ctx.FileId} using {ctx.Store.GetType().FullName}");
            return Task.CompletedTask;
        }
    }
});
```