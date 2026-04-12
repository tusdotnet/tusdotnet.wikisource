tusdotnet is built around a configuration factory that runs on every request and an event system that hooks into different phases of the upload lifecycle. Together these two give you full control over authorization, per-request logic, and post-processing without needing to step outside the standard ASP.NET Core patterns you already know.

## Accessing the current user and scoped services

The configuration factory receives the current `HttpContext` on every request, which means you have access to the authenticated user, route values, and any scoped service registered in the DI container.

```csharp
builder.Services.AddScoped<IQuotaService, QuotaService>();

app.MapTus("/files", async httpContext =>
{
    var currentUser = httpContext.User;
    var quota = httpContext.RequestServices.GetRequiredService<IQuotaService>();

    return new DefaultTusConfiguration
    {
        Store = new TusDiskStore($"/uploads/{currentUser.Identity.Name}"),
        Events = new Events
        {
            OnBeforeCreateAsync = async ctx =>
            {
                if (!await quota.UserHasRemainingQuotaAsync(currentUser.Identity.Name))
                    ctx.FailRequest(HttpStatusCode.Forbidden, "Upload quota exceeded");
            }
        }
    };
});
```

## Authorization

Because `MapTus` integrates with standard endpoint routing, authorization policies can be applied the same way as any other endpoint:

```csharp
app.MapTus("/files", async httpContext => new DefaultTusConfiguration
{
    Store = new TusDiskStore("/uploads"),
})
.RequireAuthorization("UploadPolicy");
```

For more fine-grained control, such as verifying that the file being written or deleted belongs to the current user, use the `OnAuthorizeAsync` event. It runs on every tus request and exposes the intent (create, write, delete etc.):

```csharp
app.MapTus("/files", async httpContext =>
{
    var myDb = httpContext.RequestServices.GetRequiredService<MyDbContext>();

    return new DefaultTusConfiguration
    {
        Store = new TusDiskStore("/uploads"),
        Events = new Events
        {
            OnAuthorizeAsync = async ctx =>
            {
                if (ctx.Intent == IntentType.WriteFile || ctx.Intent == IntentType.DeleteFile)
                {
                    var fileOwner = await myDb.GetFileOwnerAsync(ctx.FileId);
                    if (fileOwner != ctx.HttpContext.User.Identity.Name)
                        ctx.FailRequest(HttpStatusCode.Forbidden);
                }
            }
        }
    };
});
```

See [OnAuthorize](OnAuthorizeAsync-event) for more details.

## Per-tenant configuration

Since the factory runs per request, it can return a completely different configuration depending on the tenant. Returning `null` disables tusdotnet for that request.

```csharp
app.MapTus("/files", async httpContext =>
{
    var tenant = httpContext.RequestServices
                            .GetRequiredService<ITenantResolver>()
                            .Resolve(httpContext);

    if (tenant == null)
        return null;

    return new DefaultTusConfiguration
    {
        Store = new TusDiskStore($"/uploads/{tenant.Id}"),
        MaxAllowedUploadSizeInBytesLong = tenant.MaxUploadBytes,
    };
});
```

## Running logic when an upload completes

Use `OnFileCompleteAsync` to react when the last chunk of a file has been received. This is the right place to kick off post-processing, move the file, or notify another service.

```csharp
app.MapTus("/files", async httpContext =>
{
    var processingQueue = httpContext.RequestServices.GetRequiredService<IProcessingQueue>();

    return new DefaultTusConfiguration
    {
        Store = new TusDiskStore("/uploads"),
        Events = new Events
        {
            OnFileCompleteAsync = async ctx =>
            {
                await processingQueue.EnqueueAsync(ctx.FileId);

                ctx.HttpContext.Response.Headers.Append(
                    "Content-Location", $"/status/{ctx.FileId}"
                );
            }
        }
    };
});
```

See [Processing a completed upload](Processing-a-file-once-the-file-upload-is-complete) for a deeper discussion of sync vs async post-processing.

## Passing data between events

Events are separate callbacks with no built-in shared state. `HttpContext.Items` is a per-request dictionary available in all events via `ctx.HttpContext.Items` and works well for passing data from an early event to a later one.

A common example is looking up a database record in `OnBeforeCreateAsync` and reusing it in `OnCreateCompleteAsync` without hitting the database twice:

```csharp
app.MapTus("/files", async httpContext =>
{
    var myDb = httpContext.RequestServices.GetRequiredService<MyDbContext>();

    return new DefaultTusConfiguration
    {
        Store = new TusDiskStore("/uploads"),
        Events = new Events
        {
            OnBeforeCreateAsync = async ctx =>
            {
                var metadata = ctx.Metadata;

                if (!metadata.ContainsKey("projectId"))
                {
                    ctx.FailRequest("projectId metadata is required");
                    return;
                }

                var projectId = metadata["projectId"].GetString(Encoding.UTF8);
                var project = await myDb.GetProjectAsync(projectId);

                if (project == null)
                {
                    ctx.FailRequest(HttpStatusCode.NotFound, "Project not found");
                    return;
                }

                // Store the project for use in OnCreateCompleteAsync
                ctx.HttpContext.Items["project"] = project;
            },

            OnCreateCompleteAsync = async ctx =>
            {
                var project = (Project)ctx.HttpContext.Items["project"];

                await myDb.RegisterUploadAsync(project.Id, ctx.FileId);
            }
        }
    };
});
