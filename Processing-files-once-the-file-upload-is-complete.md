tusdotnet supports processing of a file once it has been completely uploaded using the `OnUploadCompleteAsync` callback on the `ITusConfiguration` object. 

Example

```
app.UseTus(request => new DefaultTusConfiguration
			{
				Store = new TusDiskStore(@"C:\tusfiles\"),
				UrlPath = "/files",
				OnUploadCompleteAsync = (fileId, store, cancellationToken) =>
				{
					// fileId is the id of the file that was uploaded.
					// store is the data store that was used (in this case an instance of the TusDiskStore)

					// A normal use case here would be to read the file and do some processing on it.
					var file = await (store as ITusReadableStore).GetFileAsync(fileId, cancellationToken);
					var result = await DoSomeProcessing(file, cancellationToken);

					if(!result.Success) {
						throw new MyProcessingException("Something went wrong during processing");
					}
					
					return Task.FromResult(true);
				}
			});
``