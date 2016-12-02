Since the tus spec does not contain downloading files tusdotnet will automatically forward all GET requests to the next middleware so that the developer can chose to allow file downloads.

The following example requires that the data store implements `ITusReadableStore` (`TusDiskStore` does). If it does not one would have to figure out where the files are stored and read them in some other way.

```
app.Use(async (context, next) =>
			{
				// /files/ is where we store files
				if (context.Request.Uri.LocalPath.StartsWith("/files/", StringComparison.Ordinal))
				{
					// Try to get a file id e.g. /files/<fileId>
					var fileId = context.Request.Uri.LocalPath.Replace("/files/", "").Trim();
					if (!string.IsNullOrEmpty(fileId))
					{
						var store = new TusDiskStore(@"C:\tusfiles\");
						var file = await store.GetFileAsync(fileId, context.Request.CallCancelled);

						if (file == null)
						{
							context.Response.StatusCode = 404;
							await context.Response.WriteAsync($"File with id {fileId} was not found.", context.Request.CallCancelled);
							return;
						}

						var fileStream = await file.GetContentAsync(context.Request.CallCancelled);
						var metadata = await file.GetMetadataAsync(context.Request.CallCancelled);

						// The tus protocol does not specify any required metadata.
						// "contentType" is metadata that is specific to this domain and is not required.
						context.Response.ContentType = metadata.ContainsKey("contentType")
							? metadata["contentType"].GetString(Encoding.UTF8)
							: "application/octet-stream";

						if (metadata.ContainsKey("name"))
						{
							var name = metadata["name"].GetString(Encoding.UTF8);
							context.Response.Headers.Add("Content-Disposition", new[] { $"attachment; filename=\"{name}\"" });
						}

						await fileStream.CopyToAsync(context.Response.Body, 81920, context.Request.CallCancelled);
						return;
					}
				}
```