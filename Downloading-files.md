The tus protocol does not cover downloading files. If you need to support downloads you will have to implement that yourself. The following example requires that the data store implements `ITusReadableStore` (`TusDiskStore` does). If it does not, you will need to locate the files and read them in some other way.

```csharp
app.MapGet("/files/{fileId}", async httpContext =>
{
    var fileId = (string)httpContext.GetRouteValue("fileId")!;
    var store = new TusDiskStore(@"C:\tusfiles");

    var file = await store.GetFileAsync(fileId, httpContext.RequestAborted);

    if (file == null)
    {
        httpContext.Response.StatusCode = 404;
        return;
    }

    var metadata = await file.GetMetadataAsync(httpContext.RequestAborted);

    // The tus protocol does not specify any required metadata.
    // "contentType" and "name" are metadata specific to this domain and are not required.
    httpContext.Response.ContentType = metadata.ContainsKey("contentType")
        ? metadata["contentType"].GetString(Encoding.UTF8)
        : "application/octet-stream";

    if (metadata.ContainsKey("name"))
    {
        var name = metadata["name"].GetString(Encoding.UTF8);
        httpContext.Response.Headers.Append("Content-Disposition", $"attachment; filename=\"{name}\"");
    }

    using var fileStream = await file.GetContentAsync(httpContext.RequestAborted);
    await fileStream.CopyToAsync(httpContext.Response.Body, httpContext.RequestAborted);
});
```


<details>
<summary><h3>On older frameworks that do not support endpoint routing</h3></summary>

```csharp
app.Use(async (context, next) =>
{
    if (!context.Request.Path.StartsWithSegments(new PathString("/files"), StringComparison.Ordinal,
            out PathString remaining))
    {
        await next();
        return;
    }

    // Try to get a file id e.g. /files/<fileId>
    string fileId = remaining.Value.TrimStart('/');
    if (string.IsNullOrEmpty(fileId))
    {
        await next();
        return;
    }

    var store = new TusDiskStore(@"C:\tusfiles\");
    var file = await store.GetFileAsync(fileId, context.RequestAborted);

    if (file == null)
    {
        context.Response.StatusCode = 404;
        await context.Response.WriteAsync($"File with id {fileId} was not found.", context.RequestAborted);
        return;
    }

    var metadata = await file.GetMetadataAsync(context.RequestAborted);

    // The tus protocol does not specify any required metadata.
    // "contentType" and "name" are metadata specific to this domain and are not required.
    context.Response.ContentType = metadata.ContainsKey("contentType")
        ? metadata["contentType"].GetString(Encoding.UTF8)
        : "application/octet-stream";

    if (metadata.ContainsKey("name"))
    {
        var name = metadata["name"].GetString(Encoding.UTF8);
        context.Response.Headers.Append("Content-Disposition", $"attachment; filename=\"{name}\"");
    }

    using var fileStream = await file.GetContentAsync(context.RequestAborted);
    await fileStream.CopyToAsync(context.Response.Body, context.RequestAborted);
});
```

</details>
