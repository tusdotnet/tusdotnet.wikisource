tusdotnet is simply configured by running

```csharp
app.UseTus(context => new DefaultTusConfiguration {... });
```

The configuration consists of a single ITusConfiguration instance which contains the following properties:

```csharp
/// <summary>
/// Represents the configuration used by tusdotnet.
/// </summary>
public interface ITusConfiguration
{
	/// <summary>
	/// The url path to listen for uploads on (e.g. "/files").
	/// If the site is located in a subpath (e.g. https://example.org/mysite) it must also be included (e.g. /mysite/files) 
	/// </summary>
	string UrlPath { get; }

	/// <summary>
	/// The store to use when storing files
	/// </summary>
	ITusStore Store { get; }

	/// <summary>
	/// Callback ran when a file is completely uploaded. 
	/// This callback is called only once after the last bytes have been written to the store.
	/// It will not be called for any subsequent upload requests for already completed files.
	/// </summary>
	Func<string, ITusStore, CancellationToken, Task> OnUploadCompleteAsync { get; }

	/// <summary>
	/// The maximum upload size to allow. Exceeding this limit will return a "413 Request Entity Too Large" error to the client.
	/// Set to null to allow any size. The size might still be restricted by the web server or operating system.
	/// </summary>
	int? MaxAllowedUploadSizeInBytes { get; }

	/// <summary>
	/// Set an expiration time where incomplete files can no longer be updated.
	/// This value can either be <code>AbsoluteExpiration</code> or <code>SlidingExpiration</code>.
	/// Absolute expiration will be saved per file when the file is created.
	/// Sliding expiration will be saved per file when the file is created and updated on each time the file is updated.
	/// Setting this property to null will disable file expiration.
	/// </summary>
	ExpirationBase Expiration { get; }
}
```

Depending on what store is used some configuration might also be needed for the store. The disk store that ships with tusdotnet requires a directory path and whether or not it should delete "partial" files on concatenation. 

```csharp
Store = new TusDiskStore(@"C:\tusfiles\", true)
```

In the above case `C:\tusfiles\` is where all files will be saved and true indicates that partial files should be deleted. The default value is false so that no files are unexpectedly deleted. If unsure, or not using the concatenation extension, leave it as false. See [Custom data store -> ITusConcatenationStore](https://github.com/smatsson/tusdotnet/wiki/Custom-data-store#itusconcatenationstore) for more details.
