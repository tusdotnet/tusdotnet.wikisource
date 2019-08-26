tusdotnet supports processing of a file once it has been completed using the `OnFileCompleteAsync` callback.

```csharp
app.UseTus(httpContext => new DefaultTusConfiguration
{
	Store = new TusDiskStore(@"C:\tusfiles\"),
	UrlPath = "/files",
	Events = new Events
	{
		OnFileCompleteAsync = async eventContext =>
		{
			// eventContext.FileId is the id of the file that was uploaded.
			// eventContext.Store is the data store that was used (in this case an instance of the TusDiskStore)

			// A normal use case here would be to read the file and do some processing on it.
			ITusFile file = await eventContext.GetFileAsync();
			var result = await DoSomeProcessing(file, eventContext.CancellationToken);

			if (!result.Success)
			{
				throw new MyProcessingException("Something went wrong during processing");
			}
		}
	}
});
``