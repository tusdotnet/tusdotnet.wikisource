The OnBeforeWrite event is fired just before each file data upload request (PATCH).

Calling `FailRequest` on the `BeforeWriteContext` passed to the callback will reject the request with a 400 Bad Request status code. Calling `FailRequest` multiple times will concatenate the error messages.

> :information_source: Note that this event only fires for client requests and not when manually calling the store's methods.

```csharp
app.UseTus(context => new DefaultTusConfiguration
{
	UrlPath = "/files",
	Store = new TusDiskStore(@"C:\tusfiles\"),
	Events = new Events
	{
		OnBeforeWriteAsync = ctx =>
		{
			if (ctx.UploadLength == 0)
            {
                ctx.FailRequest(HttpStatusCode.BadRequest);
            }
            if (!SomeBusinessLogic())
            {
                ctx.FailRequest("Failing request due to some business logic")
            }

			return Task.CompletedTask;
		}
    }
});
```