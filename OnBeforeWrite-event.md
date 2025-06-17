The OnBeforeWrite event is fired just before handling each file data upload request (i.e., each HTTP PATCH request containing a chunk of file data). It allows inspecting the upload and optionally rejecting the request before any data is written to storage.

If reading or inspecting the upload content (the HTTP request body) is needed, please keep in mind these points:

- Using `HttpRequest.EnableBuffering()` to rewind the body stream is not suggested. This causes the content to be written to disk twice (once by the buffering mechanism and again by tusdotnet configured store) and creates unnecessary disk I/O and performance issues.
- Recommended way is to read the request body with some `PipeReader` and buffering only what is needed. Ideal steps for this:
    - Wrap the original stream in a rewindable wrapper (like a `MemoryStream`) before handing it back to the tusdotnet pipeline
    - Ensure your replacement stream is compatible with `TusDiskStore` or your custom `ITusStore` implementation

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
            if (ctx.Store.GetUploadOffsetAsync(ctx.FileId, CancellationToken.None).Result != ctx.UploadOffset)
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