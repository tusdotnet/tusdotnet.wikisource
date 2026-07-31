To allow a browser to upload files from a page on a different domain you will need to enable cross origin resource sharing (CORS).

`tusdotnet.Helpers.CorsHelper` provides helper methods for all relevant CORS values:

- `GetAllowedHeaders()` for `Access-Control-Allow-Headers`
- `GetAllowedMethods()` for `Access-Control-Allow-Methods`
- `GetExposedHeaders()` for `Access-Control-Expose-Headers`

# ASP.NET Core

```csharp
// Program.cs
builder.Services.AddCors();

app.UseCors(builder => builder
    // CorsHelper returns the headers and methods required by the tus protocol.
    // Modify these if your application needs additional headers or methods.
    .WithHeaders(tusdotnet.Helpers.CorsHelper.GetAllowedHeaders())
    .WithMethods(tusdotnet.Helpers.CorsHelper.GetAllowedMethods())
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
        AllowAnyHeader = false,
        AllowAnyMethod = false,
    };

    corsPolicy.Origins.Add("https://example.com"); // Replace with your actual origin(s)

    // CorsHelper returns the headers and methods required by the tus protocol.
    // Modify these if your application needs additional headers or methods.
    // Headers, Methods and ExposedHeaders have private setters; populate the existing list instances instead.
    foreach (var header in tusdotnet.Helpers.CorsHelper.GetAllowedHeaders())
        corsPolicy.Headers.Add(header);

    foreach (var method in tusdotnet.Helpers.CorsHelper.GetAllowedMethods())
        corsPolicy.Methods.Add(method);

    foreach (var header in tusdotnet.Helpers.CorsHelper.GetExposedHeaders())
        corsPolicy.ExposedHeaders.Add(header);

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
