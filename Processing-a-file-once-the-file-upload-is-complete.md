tusdotnet supports processing of a file once it has been completed using the `OnFileCompleteAsync` callback.

```csharp
app.UseTus(request => new DefaultTusConfiguration
{
	Store = new TusDiskStore(@"C:\tusfiles\"),
	UrlPath = "/files",
	Events = new Events
	{
		OnFileCompleteAsync = async ctx =>
		{
			// ctx.FileId is the id of the file that was uploaded.
			// ctx.Store is the data store that was used (in this case an instance of the TusDiskStore)

			// A normal use case here would be to read the file and do some processing on it.
			var file = await ((ITusReadableStore)ctx.Store).GetFileAsync(ctx.FileId, ctx.CancellationToken);
			var result = await DoSomeProcessing(file, ctx.CancellationToken);

			if (!result.Success)
			{
				throw new MyProcessingException("Something went wrong during processing");
			}
		}
	}
});
``