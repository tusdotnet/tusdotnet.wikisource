This page describes the architecture of tusdotnet and the overall concepts of how everything fits together.

tusdotnet is a .NET server implementation of the [tus.io](https://tus.io) resumable upload protocol. It runs as middleware on both **ASP.NET Core** and **OWIN**, and supports .NET Framework 4.5.2, .NET Standard 1.3/2.0, .NET Core 3.1, and .NET 6+.

There are three main parts of tusdotnet:

* The middleware
* A configuration object
* A data store

## Middleware

When a request arrives, the middleware analyzes it to determine what the client is trying to do, the *intent*, based on the HTTP method, URL path, and headers. The resolved intent is dispatched to a dedicated handler for each tus operation. Requests that tusdotnet cannot handle, such as GET requests or requests missing the `Tus-Resumable` header, are forwarded down the pipeline unchanged so other handlers can still serve completed files or unrelated resources on the same host.

The recommended approach for ASP.NET Core (.NET Core 3.1+) is endpoint routing via `MapTus`, which integrates with ASP.NET Core's authorization and other endpoint conventions. `UseTus` is also available and is the right choice for hybrid upload solutions or older frameworks. See [Configure tusdotnet](Configure-tusdotnet#endpoint-routing-or-middleware) for a comparison and guidance on which to choose.

For OWIN, use `UseTus` on `IAppBuilder`.

## Configuration

tusdotnet is configured through a *configuration factory*, a delegate passed to `MapTus` or `UseTus` that returns a `DefaultTusConfiguration` instance. The factory runs on **every request**, which means it can inspect the incoming `HttpContext` and return a different configuration per user, tenant, or any other request property. Returning `null` disables tusdotnet for that request.

```csharp
app.MapTus("/files", async httpContext =>
{
    return new DefaultTusConfiguration
    {
        Store = new TusDiskStore("/uploads"),
        // ...
    };
});
```

The configuration object controls what store to use, size limits, expiration, which tus extensions are enabled, and more. See [Configure tusdotnet](Configure-tusdotnet) for the full reference.

## Event system

tusdotnet is event-driven. Rather than subclassing or overriding behaviour, you attach async callbacks to the `Events` property of `DefaultTusConfiguration`. The events fire in the following order for each request:

1. Configuration factory
2. `OnAuthorizeAsync`, which runs on every tus request regardless of intent. Use it for fine-grained authorization or general request logging.
3. `OnBeforeXAsync`, which runs before the operation is executed. Call `ctx.FailRequest(...)` here to reject the request.
4. `OnXCompleteAsync`, which runs after the operation has completed successfully.


`OnFileCompleteAsync` fires once the very last byte of the upload has been received and saved.

Each event receives a context object with relevant data for that phase. For example, `OnBeforeCreateAsync` exposes the parsed metadata so you can validate it before the file is created, while `OnAuthorizeAsync` exposes the intent so you can apply different authorization rules per operation.

Because event handlers are separate delegates with no built-in shared state, use `ctx.HttpContext.Items` to pass data between events within the same request. This is useful to avoid hitting the database twice across `OnBeforeCreateAsync` and `OnCreateCompleteAsync`, for example.

See [Common patterns](Common-patterns) for practical examples, and the individual event pages for full details:

* [OnAuthorize](OnAuthorizeAsync-event)
* [OnBeforeCreate](OnBeforeCreate-event) / [OnCreateComplete](OnCreateComplete-event)
* [OnBeforeWrite](OnBeforeWrite-event)
* [OnFileComplete](Processing-a-file-once-the-file-upload-is-complete)
* [OnBeforeDelete](OnBeforeDelete-event) / [OnDeleteComplete](OnDeleteComplete-event)

## Data store

The data store determines both where upload data is saved and which tus extensions the server exposes. Each extension maps to a corresponding interface on the store. If the store implements it, the extension is available. If not, it is silently disabled regardless of the `AllowedExtensions` setting.

tusdotnet ships with `TusDiskStore`, which saves files to a local directory and implements all extension interfaces. [Custom data stores](Custom-data-store) are also supported.

See [Configure TusDiskStore](Configure-TusDiskStore) for configuration options, and [Custom data store](Custom-data-store) for guidance on implementing your own.

## Further reading

For a deeper look at the internals, including intent handlers, the validation pipeline, locking, and performance optimisations, see [`tusdotnet/Documentation/Architecture.md`](https://github.com/tusdotnet/tusdotnet/blob/master/tusdotnet/Documentation/Architecture.md) in the source repository.
