tusdotnet is simply configured by running either `UseTus` or `MapTus` on your application builder, depending on if you like to use endpoint routing or not (see [differences further down](#endpoint-routing-or-middleware))

```csharp
app.UseTus(context => new DefaultTusConfiguration {... });

// OR use endpoint routing (only available for .NET Core 3.1 and later)

app.MapTus("/files", context => new DefaultTusConfiguration {... });

```

The "configuration factory" (`context => new ...`) will run on each request. Different configurations can be returned for different clients by examining the incoming `HttpContext` or `IOwinRequest`.

The return value of the factory is a single DefaultTusConfiguration instance which contains the following properties. Return null from the factory to disable tusdotnet for the current request.

```csharp
public class DefaultTusConfiguration
{
    /// <summary>
    /// The url path to listen for uploads on (e.g. "/files").
    /// If the site is located in a subpath (e.g. https://example.org/mysite) it must also be included (e.g. /mysite/files)
    /// This property is not used when using endpoint routing.
    /// </summary>
    public virtual string UrlPath { get; set; }

    /// <summary>
    /// The store to use when storing files.
    /// </summary>
    public virtual ITusStore Store { get; set; }

    /// <summary>
    /// Lock provider to use when locking to prevent files from being accessed while the file is still in use.
    /// Defaults to using in-memory locks.
    /// </summary>
    public virtual ITusFileLockProvider FileLockProvider { get; set; }

    /// <summary>
    /// Callbacks to run during different stages of the tusdotnet pipeline.
    /// </summary>
    public virtual Events Events { get; set; }

    /// <summary>
    /// The maximum upload size to allow. Exceeding this limit will return a "413 Request Entity Too Large" error to the client.
    /// Set to null to allow any size. The size might still be restricted by the web server or operating system.
    /// This property will be preceded by <see cref="MaxAllowedUploadSizeInBytesLong" />.
    /// </summary>
    public virtual int? MaxAllowedUploadSizeInBytes { get; set; }

    /// <summary>
    /// The maximum upload size to allow. Exceeding this limit will return a "413 Request Entity Too Large" error to the client.
    /// Set to null to allow any size. The size might still be restricted by the web server or operating system.
    /// This property will take precedence over <see cref="MaxAllowedUploadSizeInBytes" />.
    /// </summary>
    public virtual long? MaxAllowedUploadSizeInBytesLong { get; set; }

    /// <summary>
    /// Use the incoming request's PipeReader instead of the stream to read data from the client.
    /// This is only available on .NET Core 3.1 or later and if the store supports it through the ITusPipelineStore interface.
    /// </summary>
    public virtual bool UsePipelinesIfAvailable { get; set; }

    /// <summary>
    /// Set an expiration time where incomplete files can no longer be updated.
    /// This value can either be <c>AbsoluteExpiration</c> or <c>SlidingExpiration</c>.
    /// Absolute expiration will be saved per file when the file is created.
    /// Sliding expiration will be saved per file when the file is created and updated on each time the file is updated.
    /// Setting this property to null will disable file expiration.
    /// </summary>
    public virtual ExpirationBase Expiration { get; set; }

    /// <summary>
    /// Set the strategy to use when parsing metadata. Defaults to <see cref="MetadataParsingStrategy.AllowEmptyValues"/> for better compatibility with tus clients.
    /// Change to <see cref="MetadataParsingStrategy.Original"/> to use the old format.
    /// </summary>
    public virtual MetadataParsingStrategy MetadataParsingStrategy { get; set; }

    /// <summary>
    /// Tus extensions allowed to use by the client. Defaults to <see cref="TusExtensions.All" />.
    /// In addition to being in this list the extension must also be supported by the store provided in <see cref="DefaultTusConfiguration.Store"/> to be accessible for the client.
    /// </summary>
    public TusExtensions AllowedExtensions { get; set; }

    /// <summary>
    /// Timeout to wait for data from the client. 
    /// The timeout is applied from the moment the store starts reading from the client until it has filled its internal read buffer.
    /// Once the buffer is filled, the timeout is reset and restarted on the next read.
    /// When <see cref="UsePipelinesIfAvailable" /> is enabled, the internal read buffer is always 4 KiB. When false, it is determined by the store.
    /// A higher value will make tusdotnet wait longer for data, but will also result in locks not being released as fast which can be an issue if the client abrubtly disconnects due to network loss or similar.
    /// The default value is 60 seconds.
    /// </summary>
    public TimeSpan ClientReadTimeout { get; set; }
}
```

# Events

tusdotnet is event based which means that the developer will set up event handlers that will fire during different phases of the upload. These are all set using the `Events` property of the `DefaultTusConfiguration` instance.

For each request, the events will be called in the following order:
1. `<configuration factory>`
2. `OnAuthorize`
3. `OnBeforeX`
4. `OnXComplete`

Once the file is completely uploaded the `OnFileComplete` event will fire.

Each event is described below:
* [OnAuthorize](OnAuthorizeAsync-event)
* [OnFileComplete](Processing-a-file-once-the-file-upload-is-complete)
* [OnBeforeCreate](OnBeforeCreate-event)
* [OnCreateComplete](OnCreateComplete-event)
* [OnBeforeDelete](OnBeforeDelete-event)
* [OnDeleteComplete](OnDeleteComplete-event)

# Store

The store is the heart of how the data is stored by tusdotnet. Please refer to the store's documentation to find options and how to use it.

* [TusDiskStore](Configure-tusdiskstore)

# Allowed extensions

The tus protocol consists of a core protocol and multiple extensions that add support for various things such as deleting a file, using checksum verification and others.

tusdotnet will support extensions based on what the store supports and what extensions have been enabled using the `AllowedExtensions` property on the configuration object. By default all extensions are available if the store supports them.

To enable all extensions except for termination (deletion of a file):

```csharp
AllowedExtensions = TusExtensions.All.Except(TusExtensions.Termination)
```

To only allow creation and termination (a.k.a creating files and deleting files):
```csharp
AllowedExtensions = new TusExtensions(TusExtensions.Creation, TusExtensions.Termination)
```

To only enable a single extension:
```csharp
AllowedExtensions = TusExtensions.Creation
```

# Endpoint routing or middleware?

tusdotnet supports running both as an endpoint and as a middleware. Which one to chose depends on the use case. 
For most use cases it is recommended to use endpoint routing (`app.MapTus`). This requires that the runtime is .NET Core 3.1 or later.

Advantages to using endpoint routing (`app.MapTus`):
* Integration with the template engine in ASP.NET Core
* Integration with authorization and other endpoint conventions in ASP.NET Core

Advantages to using the middleware (`app.UseTus`):
* Hybrid uploads solutions can be created, i.e. the server can handle non tus requests on the same endpoint.
* Works on older frameworks than .NET Core 3.1