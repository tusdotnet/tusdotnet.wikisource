The OnCreateComplete event is fired once a file has been created.

```csharp
app.UseTus(context => new DefaultTusConfiguration
{
	UrlPath = "/files",
	Store = new TusDiskStore(@"C:\tusfiles\"),
	Events = new Events
	{
		OnCreateCompleteAsync = ctx =>
		{
			logger.LogInformation($"Created file {ctx.FileId} using {ctx.Store.GetType().FullName}");
			return Task.CompletedTask;
		}
	}
});
```