Since the tus spec does not contain downloading files tusdotnet will automatically forward all GET requests to the next middleware so that the developer can chose to allow file downloads.

The following example requires that the data store implements `ITusReadableStore` (`TusDiskStore` does). If it does not one would have to figure out where the files are stored and read them in some other way.

```csharp
app.Use(async (context, next) =>
{
        if (context.Request.Path.StartsWithSegments(new PathString("/files"), StringComparison.Ordinal, 
                out PathString remaining))
        {
                // Try to get a file id e.g. /files/<fileId>
                string fileId = remaining.Value.TrimStart('/');
                if (!string.IsNullOrEmpty(fileId))
                {
                        var store = new TusDiskStore(@"C:\tusfiles\");
                        var file = await store.GetFileAsync(fileId, context.RequestAborted);
                
                        if (file == null)
                        {
                                context.Response.StatusCode = 404;
                                await context.Response.WriteAsync($"File with id {fileId} was not found.", context.RequestAborted);
                                return;
                        }
                
                        var fileStream = await file.GetContentAsync(context.RequestAborted);
                        var metadata = await file.GetMetadataAsync(context.RequestAborted);
                
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
                
                        using (var fileStream = await file.GetContentAsync(context.RequestAborted))
                        {
                                await fileStream.CopyToAsync(context.Response.Body, context.RequestAborted);
                        }
                }
                else
                {
                        // Call next handler in pipeline if it's something else
                        await next();
                }
        }
        else
        {
                await _next(context);
        }
});
```
