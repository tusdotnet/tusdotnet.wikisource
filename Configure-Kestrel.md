> :warning: As of tusdotnet 2.8 this configuration is no longer needed. The page is kept for backward compatibility. For tusdotnet 2.8 or later use [`DefaultTusConfiguration.ClientReadTimeout`](Configure-tusdotnet) instead.

This page describes how to configure Kestrel if not running behind a reverse proxy such as IIS or Nginx.

Kestrel does not by default have a request timeout <sup>[[1](https://github.com/aspnet/KestrelHttpServer/pull/485#discussion_r55264843), [2](https://github.com/dotnet/aspnetcore/issues/10079#issuecomment-490519795)]</sup> and thus does not cancel the request cancellation token if the client disconnects abruptly. As tusdotnet relies on this information to handle client disconnects it might end up in locked files, unreleased resources etc.

To setup a request timeout middleware one can use the following code.

> :information_source: The request timeout middleware is only needed when running directly on Kestrel or when running behind some specific reverse proxies. In most cases the reverse proxy will handle the request timeout and notify tusdotnet that the client has disconnected.

```csharp
app.Use(async (httpContext, next) =>
{
    // Specify timeout, in this case 60 seconds. 
    // If running behind a reverse proxy make sure this value matches the request timeout in the proxy.
    var requestTimeout = TimeSpan.FromSeconds(60);

    // Add timeout to the current request cancellation token. 
    // If the client does a clean disconnect the cancellation token will also be flagged as cancelled.
    using var timeoutCts = CancellationTokenSource.CreateLinkedTokenSource(httpContext.RequestAborted);

    // Make sure to cancel the cancellation token after the timeout. 
    // Once this timeout has been reached, tusdotnet will cancel all pending reads 
    // from the client and save the parts of the file has been received so far.
    timeoutCts.CancelAfter(requestTimeout);

    // Replace the request cancellation token with our token that supports timeouts.
    httpContext.RequestAborted = timeoutCts.Token;

    // Continue the execution chain.
    await next();
});

// The usual setup of tusdotnet.
app.MapTus("/files", httpContext => ...);
```
