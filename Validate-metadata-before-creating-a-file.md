In tusdotnet 1.x there is no built in support for validating metadata when creating a new file. The feature has been proposed for tusdotnet 2.x but there is currently no ETA on this release.

In the meanwhile one can add this behavior by adding an extra middleware just before the call to `app.UseTus` as below.

```csharp
app.Use(async (context, next) =>
{
	// "/files" must be the same path as specified in DefaultTusConfiguration.
	if (context.Request.Path == "/files" && context.Request.Method == "POST")
	{
		var metaString = context.Request.Headers["Upload-Metadata"];
		var meta = tusdotnet.Models.Metadata.Parse(metaString);

		// Validate your metadata here...
		tusdotnet.Models.Metadata myMetadata;
		if (!meta.TryGetValue("MyMetaKey", out myMetadata))
		{
			// Write error message and return if anything is invalid
			context.Response.StatusCode = 400;
			await context.Response.WriteAsync("MyMetaKey is missing");
			return;
		}

		if (myMetadata.GetString(new System.Text.UTF8Encoding()) != "Hello world")
		{
			context.Response.StatusCode = 400;
			await context.Response.WriteAsync("MyMetaKey is invalid");
			return;
		}
	}

	// Call next middleware if everything is OK
	await next.Invoke();
});

app.UseTus(context => new DefaultTusConfiguration...);
```