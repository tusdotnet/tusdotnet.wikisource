To enable CORS, install the Nuget package Microsoft.Owin.Cors and modify your appbuilder code as below.

```
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

app.UseTus(() => ...)
```