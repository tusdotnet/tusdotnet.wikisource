The OnBeforeWrite event is fired just before handling each file data upload request (i.e., each HTTP PATCH request containing a chunk of file data). It allows inspecting the upload and optionally rejecting the request before any data is written to storage.

If you need to read or inspect the uploaded content (the HTTP request body), it is *highly* recommended to use pipelines, if available and supported by the `ITusStore` in use. The PipeReader implementation (`HttpContext.Request.BodyReader`) includes built-in buffering, whereas the Stream implementation (`HttpContext.Request.Body`) does not.

If you must use the Stream implementation, please consider the following points:

- The Stream can only be read once because it is not seekable. Buffering must be enabled for it to function correctly.

- Using `HttpRequest.EnableBuffering()` to rewind the body stream is a quick solution, but it has downsides when using `TusDiskStore`. It causes the content to be written to disk twice — first by the buffering mechanism and again by the tusdotnet configured store, resulting in unnecessary disk I/O and performance issues.

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
			if (!SomeBusinessLogic())
			{
				ctx.FailRequest("Failing request due to some business logic")
			}

			return Task.CompletedTask;
		}
    }
});
```

```csharp
app.UseTus(context => new DefaultTusConfiguration
{
	UrlPath = "/files",
	Store = new TusDiskStore(@"C:\tusfiles\"),
	Events = new Events
	{
		OnBeforeWriteAsync = ctx =>
		{
			if (ctx.UploadOffset is not 0)
				return;

			var read = await ctx.HttpContext.Request.BodyReader.ReadAtLeastAsync(
				10,
				ctx.CancellationToken
			);

			var isValid = IsValidFileSignature(read.Buffer);

			if (isValid)
			{
				logger.LogInformation("File contains the data needed");
			}
			else
			{
				ctx.FailRequest("File is invalid");
			}

			ctx.HttpContext.Request.BodyReader.AdvanceTo(read.Buffer.Start, read.Buffer.Start);
		},
    }
});
```

> :information_source: Note that the content may not be complete in the request as the client could be sending 1 byte or 10 GB of data.