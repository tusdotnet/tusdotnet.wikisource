tusdotnet supports processing of a file once it has been completed using the `OnFileCompleteAsync` callback. The tus protocol, and by extension tusdotnet, separates the upload of the file from the processing of the file content. Post-processing can be done either synchronously during the final upload request or asynchronously after the request has completed. Both approaches have their advantages and disadvantages.

> :information_source: Note that this event only fires once when the upload completes and will not be fired on additional checks for upload status.

## Sync processing

* **Immediate Response**: Returns the response directly to the client in the final upload request.
* **No Retry Mechanism**: Cannot be retried. If an error occurs during processing, the tus client will see that the file is already completely uploaded and won't retry the operation.
* **Limited Response Content**: tus does not allow for content to be returned for successful write operations, as the response must be `204 No Content`. However, headers can be used to provide small amounts of data.
* **Resource Availability**: The server might not have the resources to process the file immediately.

### Example

```csharp
app.UseTus(httpContext => new DefaultTusConfiguration
{
	Store = new TusDiskStore(@"C:\tusfiles\"),
	UrlPath = "/files",
	Events = new Events
	{
		OnFileCompleteAsync = async eventContext =>
		{
			ITusFile file = await eventContext.GetFileAsync();
			
			var result = await DoSomeProcessing(file, eventContext.CancellationToken);
			eventContext.HttpContext.Headers.Append("Result", result);
		}
	}
});
```

```javascript
// Using tus-js-client
const upload = new tus.Upload(file,
{
	endpoint: 'files/',
	onSuccess: (payload) => {
		const { lastResponse } = payload;
		console.log(lastResponse.getHeader('Result'));
	},
});

upload.start();
``` 

## Async processing

* **Polling URL**: Returns a URL to the client, which can be used to poll for information about the processing status.
* **Retry Capability**: Processing can be retried since the upload is separate from the processing.
* **Flexible Scheduling**: The server can choose when to process the uploaded file, for example, by using a queue.
* **Additional Endpoint**: Requires a new endpoint to be implemented for processing status.

### Example

```csharp
app.MapGet(
    "/status/{id}",
    async (string id) =>
    {
        var status = await FetchStatus(id);
        return Results.Ok(new { status, retryUrl = $"/retry/{id}" });

        static async Task<string> FetchStatus(string id) => "ok";
    }
);

app.MapPost(
    "/retry/{id}",
    async (string id) =>
    {
        await Process(id);
        return Results.Ok();

        static async Task Process(string id) { }
    }
);

app.UseTus(httpContext => new DefaultTusConfiguration
{
	Store = new TusDiskStore(@"C:\tusfiles\"),
	UrlPath = "/files",
	Events = new Events
	{
		OnFileCompleteAsync = async eventContext =>
		{
			ITusFile file = await eventContext.GetFileAsync();
			
			await QueueForProcessing(file);
			
			// One could also use the OnAuthorizeAsync event to add the header to all responses depending on the use case.
			eventContext.HttpContext.Headers.Append("Content-Location", $"/status/{file.Id}");
		}
	}
});

```

```javascript
// Using tus-js-client

const onContentLocationUrl = (url) => {
	// Omitted: Use url and poll status until finished
	console.log(url);
}

const upload = new tus.Upload(file,
{
	endpoint: 'files/',
	onSuccess: (payload) => {
		const { lastResponse } = payload;
		onContentLocationUrl(lastResponse.getHeader('Content-Location'));
	},
});

upload.start();
``` 