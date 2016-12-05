tusdotnet ships with a single store, the TusDiskStore, which saves files in a directory on disk. You can implement your own store by implementing one or more of the following interfaces. 

tusdotnet will automatically handle requests depending on what interfaces are implemented.

* [ITusStore](#itusstore)- Support for the core protocol
* [ITusCreationStore](#ituscreationstore) - Support for the Creation extension
* [ITusReadableStore](#itusreadablestore) - Support for reading files from the store (e.g. for downloads or processing)

## ITusStore
Required: yes | Tus-Extension: <none>

This is the interface for the core protocol. It must be implemented for the store to work. 

```csharp
public interface ITusStore
{
	/// <summary>
	/// Write data to the file using the provided stream.
	/// The implementation must throw <exception cref="TusStoreException"></exception> 
	/// if the streams length exceeds the upload length of the file.
	/// </summary>
	/// <param name="fileId">The id of the file to write.</param>
	/// <param name="stream">The request input stream from the client</param>
	/// <param name="cancellationToken">Cancellation token to use when cancelling.</param>
	/// <returns>The number of bytes written</returns>
	Task<long> AppendDataAsync(string fileId, Stream stream, CancellationToken cancellationToken);

	/// <summary>
	/// Check if a file exist.
	/// </summary>
	/// <param name="fileId">The id of the file to check.</param>
	/// <param name="cancellationToken">Cancellation token to use when cancelling.</param>
	/// <returns></returns>
	Task<bool> FileExistAsync(string fileId, CancellationToken cancellationToken);

	/// <summary>
	/// Returns the upload length specified when the file was created or null if Defer-Upload-Lenght was used.
	/// </summary>
	/// <param name="fileId">The id of the file to check.</param>
	/// <param name="cancellationToken">Cancellation token to use when cancelling.</param>
	/// <returns>The upload length of the file</returns>
	Task<long?> GetUploadLengthAsync(string fileId, CancellationToken cancellationToken);

	/// <summary>
	/// Returns the current size of the file a.k.a. the upload offset.
	/// </summary>
	/// <param name="fileId">The id of the file to check.</param>
	/// <param name="cancellationToken">Cancellation token to use when cancelling.</param>
	/// <returns>The size of the current file</returns>
	Task<long> GetUploadOffsetAsync(string fileId, CancellationToken cancellationToken);
}
```

## ITusCreationStore
Required: yes | Tus-Extension: creation

This interface handles the creation extension of the protocol and is used for creating file references that one can later upload data to using the core protocol.

```csharp
public interface ITusCreationStore
{
	/// <summary>
	/// Create a file upload reference that can later be used to upload data.
	/// </summary>
	/// <param name="uploadLength">The length of the upload in bytes</param>
	/// <param name="metadata">The Upload-Metadata request header or null if no header was provided</param>
	/// <param name="cancellationToken">Cancellation token to use when cancelling</param>
	/// <returns></returns>
	Task<string> CreateFileAsync(long uploadLength, string metadata, CancellationToken cancellationToken);

	/// <summary>
	/// Get the Upload-Metadata header as it was provided to <code>CreateFileAsync</code>.
	/// </summary>
	/// <param name="fileId">The id of the file to get the header for</param>
	/// <param name="cancellationToken">Cancellation token to use when cancelling</param>
	/// <returns>The Upload-Metadata header</returns>
	Task<string> GetUploadMetadataAsync(string fileId, CancellationToken cancellationToken);
}
```