The OnBeforeCreate event is fired just before a file is created. 

Calling `FailRequest` on the `BeforeCreateContext` passed to the callback will reject the request with a 400 Bad Request status code. Calling `FailRequest` multiple times will concatenate the error messages.

> :information_source: Note that this event only fires for client requests and not when manually calling the store's methods.

```csharp
app.UseTus(context => new DefaultTusConfiguration
{
	UrlPath = "/files",
	Store = new TusDiskStore(@"C:\tusfiles\"),
	Events = new Events
	{
		OnBeforeCreateAsync = ctx =>
		{
			if (!ctx.Metadata.ContainsKey("name"))
			{
				ctx.FailRequest("name metadata must be specified. ");
			}

			if (!ctx.Metadata.ContainsKey("contentType"))
			{
				ctx.FailRequest("contentType metadata must be specified. ");
			}

			return Task.CompletedTask;
		}
	}
});
```