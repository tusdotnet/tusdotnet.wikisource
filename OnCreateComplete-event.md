The OnCreateComplete event is fired once a file has been created.

> :information_source: Note that this event only fires for client requests and not when manually calling the store's methods.

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