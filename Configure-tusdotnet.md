tusdotnet is simply configured by running

```csharp
app.UseTus(context => new DefaultTusConfiguration {... });
```

The configuration consists of a single DefaultTusConfiguration instance which contains the following properties:

```csharp
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
	/// Callbacks to run during different stages of the tusdotnet pipeline.
	/// </summary>
	public virtual Events Events { get; set; }

	/// <summary>
	/// The maximum upload size to allow. Exceeding this limit will return a "413 Request Entity Too Large" error to the client.
	/// Set to null to allow any size. The size might still be restricted by the web server or operating system.
	/// </summary>
	public virtual int? MaxAllowedUploadSizeInBytes { get; set; }

	/// <summary>
	/// Set an expiration time where incomplete files can no longer be updated.
	/// This value can either be <code>AbsoluteExpiration</code> or <code>SlidingExpiration</code>.
	/// Absolute expiration will be saved per file when the file is created.
	/// Sliding expiration will be saved per file when the file is created and updated on each time the file is updated.
	/// Setting this property to null will disable file expiration.
	/// </summary>
	public virtual ExpirationBase Expiration { get; set; }
}
```

Depending on what store is used some configuration might also be needed for the store. The disk store that ships with tusdotnet requires a directory path and whether or not it should delete "partial" files on concatenation. 

```csharp
Store = new TusDiskStore(@"C:\tusfiles\", deletePartialFilesOnConcat: true)
```

In the above case `C:\tusfiles\` is where all files will be saved and `deletePartialFilesOnConcat: true` indicates that partial files should be deleted once a final file has been created (only used by the concatenation extension). The default value is false so that no files are unexpectedly deleted. If unsure, or not using the concatenation extension, leave it as false. See [Custom data store -> ITusConcatenationStore](https://github.com/smatsson/tusdotnet/wiki/Custom-data-store#itusconcatenationstore) for more details.
