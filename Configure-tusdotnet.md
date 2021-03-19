tusdotnet is simply configured by running

```csharp
app.UseTus(context => new DefaultTusConfiguration {... });
```

The provided factory (`context => new ...`) will run on each request. Different configurations can be returned for different clients by examining the incoming `HttpContext` or `IOwinRequest`.

The return value of the factory is a single DefaultTusConfiguration instance which contains the following properties:

```csharp
/// <summary>
/// The default tusdotnet configuration.
/// </summary>
public class DefaultTusConfiguration
{
	/// <summary>
	/// The url path to listen for uploads on (e.g. "/files").
	/// If the site is located in a subpath (e.g. https://example.org/mysite) it must also be included (e.g. /mysite/files) 
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
	/// Set an expiration time where incomplete files can no longer be updated.
	/// This value can either be <code>AbsoluteExpiration</code> or <code>SlidingExpiration</code>.
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
```

Depending on what store is used some configuration might also be needed for the store.
