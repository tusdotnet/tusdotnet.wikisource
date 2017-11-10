The OnDeleteComplete event is fired once a file has been deleted.

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