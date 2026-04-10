To allow a browser to upload files from a page on a different domain you will need to enable cross origin resource sharing (CORS).

# ASP.NET Core

```csharp
// Program.cs
builder.Services.AddCors();

app.UseCors(builder => builder
    .AllowAnyHeader()
    .AllowAnyMethod()
    .WithOrigins("https://example.com") // Replace with your actual origin(s)
    .WithExposedHeaders(tusdotnet.Helpers.CorsHelper.GetExposedHeaders())
);

app.MapTus("/files", httpContext => ...);
```

# ASP.NET 4.x (OWIN)

Install the `Microsoft.Owin.Cors` package and modify your Startup class as below.

```csharp
public void Configuration(IAppBuilder app)
{
    var corsPolicy = new System.Web.Cors.CorsPolicy
    {
        AllowAnyHeader = true,
        AllowAnyMethod = true,
    };

    corsPolicy.Origins.Add("https://example.com"); // Replace with your actual origin(s)

    // ExposedHeaders has a private setter so reflection is needed to set it.
    corsPolicy.GetType()
        .GetProperty(nameof(corsPolicy.ExposedHeaders))
        .SetValue(corsPolicy, tusdotnet.Helpers.CorsHelper.GetExposedHeaders());

    app.UseCors(new CorsOptions
    {
        PolicyProvider = new CorsPolicyProvider
        {
            PolicyResolver = context => Task.FromResult(corsPolicy)
        }
    });

    app.UseTus(...);
}
```
