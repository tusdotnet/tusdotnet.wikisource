*NOTE* This issue will be fixed in the upcoming 2.1 release (no ETA yet).

# Error message in Visual Studio

```
Severity	Code	Description	Project	File	Line	Suppression State
Error	CS1929	'IApplicationBuilder' does not contain a definition for 'UseTus' and the best extension method overload 'TusAppBuilderExtensions.UseTus(IAppBuilder, Func<IOwinRequest, ITusConfiguration>)' requires a receiver of type 'IAppBuilder'	
```

TL;DR: Look at the bottom of this post for [solutions](#solutions).

# Background of the problem

When I started developing tusdotnet .NET Core was still in beta and was not ready for production so I based the code on the OWIN pipeline. Once .NET Core was released for real I looked into different ways of determining what pipeline to use. At the time all ASP.NET Core apps would run on .NET Standard, which is a standard set of APIs that all .NET implementations should implement. tusdotnet uses this information to determine, at compile time, what pipeline to expose (either OWIN or ASP.NET Core). Since we still need to support OWIN for some unforeseeable future we still have this limitation. Good thing is that .NET Framework is compatible with .NET Standard. tusdotnet requires .NET Standard 1.3 which corresponds to .NET 4.6. The downside is that not all Nuget packages have been updated to support .NET standard (most have though). This is basically adding "netstandard1.3" to your supported list of frameworks. This whole thing with .NET Core, .NET Standard and .NET Framework is really messy and a kind-of-working overview can be found at https://docs.microsoft.com/en-us/dotnet/articles/standard/library#net-platforms-support

# Solutions

There are basically two solutions you can use:

1. Change platform to .NET Standard.

.NET Standard 1.3 is fully compatible with .NET Framework 4.6. The downside is that some Nuget packages might not be supported (most are though). To change platform, right click on your project in Visual Studio and select "Edit media.csproj". Change <TargetFramework>net452</TargetFramework> to <TargetFramework>netstandard1.3</TargetFramework>. Save the file and reload the project.

2. Continue using .NET Framework and use the OWIN pipeline alongside the ASP.NET Core pipeline.

This will require you to run two pipelines in your app which decreases performance somewhat. The upside is that all Nuget packages are supported. To use OWIN you need to install three packages from Nuget and update your setup code for tusdotnet.

Install packages: Microsoft.Owin, Microsoft.Owin.Cors and Microsoft.AspNetCore.Owin

Change your setup to use:
```csharp

app.UseOwin(setup => setup(next =>
{
	var owinAppBuilder = new AppBuilder();

	var aspNetCoreLifetime =
		(IApplicationLifetime)app.ApplicationServices.GetService(typeof(IApplicationLifetime));

	new AppProperties(owinAppBuilder.Properties)
	{
		OnAppDisposing = aspNetCoreLifetime?.ApplicationStopping ?? CancellationToken.None,
		DefaultApp = next
	};
	
	// Only required if CORS is used, configure it as you wish
	var corsPolicy = new System.Web.Cors.CorsPolicy
	{
		AllowAnyHeader = true,
		AllowAnyMethod = true,
		AllowAnyOrigin = true,
		SupportsCredentials = true
	};

	corsPolicy.GetType()
          .GetProperty(nameof(corsPolicy.ExposedHeaders))
          .SetValue(corsPolicy, tusdotnet.Helpers.CorsHelper.GetExposedHeaders());

	owinAppBuilder.UseCors(new Microsoft.Owin.Cors.CorsOptions
	{
		PolicyProvider = new CorsPolicyProvider
		{
			PolicyResolver = context => Task.FromResult(corsPolicy)
		}
	});

	owinAppBuilder.UseTus(context => new DefaultTusConfiguration
	{
	   // Excluded for brevity, use the same configuration as you would normally do
	});

	return owinAppBuilder.Build<Func<IDictionary<string, object>, Task>>();
}));

```
