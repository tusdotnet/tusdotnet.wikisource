To allow a browser to upload files from a page on a different domain you will need to enable cross origin resource sharing (CORS).

# ASP.NET 4.x

Install package Microsoft.Owin.Cors and modify your Startup class as below.

```
public void Configuration(IAppBuilder app)
{
	var corsPolicy = new CorsPolicy
	{
		AllowAnyHeader = true,
		AllowAnyMethod = true,
		AllowAnyOrigin = true,
		ExposedHeaders = { "Location" },
		SupportsCredentials = true
	};
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

# ASP.NET Core

Install package Microsoft.AspNetCore.Cors and modify your Startup class as below.

```
public void ConfigureServices(IServiceCollection services)
{
	services.AddCors();
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
	app.UseCors(builder => builder.AllowAnyHeader().AllowAnyMethod().AllowAnyOrigin());
	app.UseTus(...);
}

```