When a file resource is created, the client will receive an upload URL that is used to upload data to the resource.
By default tusdotnet will return a URL in the form of `<path>/<fileId>` where `<path>` is the request path for the creation request. E.g when using `app.MapTus("/files")` the upload URL will be `/files/<fileId>`.

This value can be modified from inside the [`OnCreateComplete`](OnCreateComplete-event) event by calling `SetUploadUrl` on the provided context. The URL can be either relative or absolute. Note that tusdotnet is path based which means that the last part of the path (`/files/{ctx.FileId}`) must be the file ID.

```csharp
OnCreateCompleteAsync = ctx =>
{
    ctx.SetUploadUrl(new Uri($"https://example.org/files/{ctx.FileId}?queryParam=1234"));
    return Task.CompletedTask;
}
```