tusdotnet ships with a single store, the TusDiskStore, which saves files in a directory on disk. You can implement your own store by implementing one or more of the following interfaces. tusdotnet will automatically handle requests and add information to the Tus-Extension header depending on what interfaces are implemented by the store used for the request.

Please note that some methods might be called multiple times during request execution. It is up to the store to properly cache data.

The most common interfaces to implement are [ITusStore](#itusstore), [ITusCreationStore](#ituscreationstore) and [ITusReadableStore](#itusreadablestore). This will allow the store to create and upload files and to read the files back for processing or downloading.

# Interfaces
* [ITusStore](#itusstore) - Support for the core protocol
* [ITusPipelineStore](#ituspipelinestore) - Support for the core protocol when using System.IO.Pipelines instead of System.IO.Stream for reading data
* [ITusChecksumStore](#ituschecksumstore) - Support for the Checksum extension (checksum verification of files)
* [ITusConcatenationStore](#itusconcatenationstore) - Support for the Concatenation extension (merging multiple files together with a single command)
* [ITusCreationStore](#ituscreationstore) - Support for the Creation extension (creating new files)
* [ITusCreationDeferLength](#ituscreationdeferlengthstore) - Support for Upload-Defer-Length (sub extension of Creation)
* [ITusReadableStore](#itusreadablestore) - Support for reading files from the store (e.g. for downloads or processing)
* [ITusTerminationStore](#itusterminationstore) - Support for the Termination extension (deleting files)
* [ITusExpirationStore](#itusexpirationstore) - Support for the Expiration extensions (files expire after a period of time)

## ITusStore
Required: yes | Tus-Extension: \<none\>

This is the interface for the core protocol. It must be implemented for the store to work.

NOTE: It is recommended to implement [ITusPipelineStore](#ituspipelinestore) instead of this interface if running on modern platforms (.NET Core 3.1 or later).

Read more: http://tus.io/protocols/resumable-upload.html#core-protocol

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

## ITusPipelineStore
Required: no | Tus-Extension: \<none\>

This is the interface for the core protocol when using System.IO.Pipelines for reading. Using System.IO.Pipelines increases performance in regard to CPU usage, memory consumption and throughput. This interface inherits from `ITusStore`. It is recommended to implement this interface instead of `ITusStore` if running on modern platforms (.NET Core 3.1 or later).

Read more: http://tus.io/protocols/resumable-upload.html#core-protocol

```csharp
public interface ITusPipelineStore : ITusStore
{
	/// <summary>
	/// Write data to the file using the provided pipe reader.
	/// The implementation must throw <exception cref="TusStoreException"></exception> 
	/// if the pipe readers length exceeds the upload length of the file.
	/// </summary>
	/// <param name="fileId">The id of the file to write</param>
	/// <param name="pipeReader">The request input pipe reader from the client</param>
	/// <param name="cancellationToken">Cancellation token to use when cancelling</param>
	/// <returns>The number of bytes written</returns>
	Task<long> AppendDataAsync(string fileId, PipeReader pipeReader, CancellationToken cancellationToken);
}

```

## ITusChecksumStore
Required: no | Tus-Extension: checksum

This interface adds support for checksum verification of files. 

Read more: http://tus.io/protocols/resumable-upload.html#checksum

```csharp
public interface ITusChecksumStore
	{
		/// <summary>
		/// Returns a collection of hash algorithms that the store supports (e.g. sha1).
		/// </summary>
		/// <param name="cancellationToken">Cancellation token to use when cancelling</param>
		/// <returns>The collection of hash algorithms</returns>
		Task<IEnumerable<string>> GetSupportedAlgorithmsAsync(CancellationToken cancellationToken);

		/// <summary>
		/// Verify that the provided checksum matches the file checksum.
		/// </summary>
		/// <param name="fileId">The id of the file to check</param>
		/// <param name="algorithm">The checksum algorithm to use when checking. This algorithm must be supported by the store.</param>
		/// <param name="checksum">The checksom to use for verification</param>
		/// <param name="cancellationToken">Cancellation token to use when cancelling</param>
		/// <returns>True if the checksum matches otherwise false</returns>
		Task<bool> VerifyChecksumAsync(string fileId, string algorithm, byte[] checksum, CancellationToken cancellationToken);
	}
```

## ITusConcatenationStore
Required: no | Tus-Extension: concatenation

*Note*: This extension requires that [ITusCreationStore](#ituscreationstore) is also implemented.

This interface adds support for the concatenation extension which allows a client to concatenate multiple files into a final file with a single POST request. 

Read more: http://tus.io/protocols/resumable-upload.html#concatenation

```csharp
public interface ITusConcatenationStore
	{
		/// <summary>
		/// Returns the type of Upload-Concat header that was used when creating the file.
		/// Returns null if no Upload-Concat was used.
		/// </summary>
		/// <param name="fileId">The file to check</param>
		/// <param name="cancellationToken">Cancellation token to use when cancelling</param>
		/// <returns>FileConcatPartial, FileConcatFinal or null</returns>
		Task<FileConcat> GetUploadConcatAsync(string fileId, CancellationToken cancellationToken);

		/// <summary>
		/// Create a partial file. This method is called when a Upload-Concat header is present and when its value is "partial".
		/// </summary>
		/// <param name="uploadLength">The length of the upload in bytes</param>
		/// <param name="metadata">The Upload-Metadata request header or null if no header was provided</param>
		/// <param name="cancellationToken">Cancellation token to use when cancelling</param>
		/// <returns>The id of the newly created file</returns>
		Task<string> CreatePartialFileAsync(long uploadLength, string metadata, CancellationToken cancellationToken);

		/// <summary>
		/// Creates a final file by concatenating multiple files together. This method is called when a Upload-Concat header
		/// is present with a "final" value.
		/// </summary>
		/// <param name="partialFiles">List of file ids to concatenate</param>
		/// <param name="metadata">The Upload-Metadata request header or null if no header was provided</param>
		/// <param name="cancellationToken">Cancellation token to use when cancelling</param>
		/// <returns>The id of the newly created file</returns>
		Task<string> CreateFinalFileAsync(string[] partialFiles, string metadata, CancellationToken cancellationToken);
	}
```

## ITusCreationStore
Required: no | Tus-Extension: creation

This interface handles the creation extension of the protocol and is used for creating file references that one can later upload data to using the core protocol.

Read more: http://tus.io/protocols/resumable-upload.html#creation

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

## ITusCreationDeferLengthStore
Required: no | Tus-Extension: creation-defer-length

*Note*: This extension requires that [ITusCreationStore](#ituscreationstore) is also implemented.

Creation-defer-length is a sub extension of the creation extension that allows users to create files without knowing the size of the upload in advance. 

*Note*: Calls to `CreateFileAsync` (`ITusCreationStore`) and `CreatePartialFileAsync` (`ITusConcatenationStore`) will be invoked with `-1` as the length of the file if this interface is implemented and the user choses to use this feature.

Read more at: http://tus.io/protocols/resumable-upload.html#upload-defer-length

```csharp
public interface ITusCreationDeferLengthStore
{
	/// <summary>
	/// Set the upload length (in bytes) of the provided file.
	/// </summary>
	/// <param name="fileId">The id of the file to set the upload length for</param>
	/// <param name="uploadLength">The length of the upload in bytes</param>
	/// <param name="cancellationToken">Cancellation token to use when cancelling</param>
	/// <returns>Task</returns>
	Task SetUploadLengthAsync(string fileId, long uploadLength, CancellationToken cancellationToken);
}
```

## ITusTerminationStore
Required: no | Tus-Extension: termination

This interface adds support for the termination extension allowing clients to delete files.

Read more: http://tus.io/protocols/resumable-upload.html#termination

```csharp
public interface ITusTerminationStore
	{
		/// <summary>
		/// Delete a file from the data store.
		/// </summary>
		/// <param name="fileId">The id of the file to delete</param>
		/// <param name="cancellationToken">Cancellation token to use when cancelling</param>
		/// <returns>Task</returns>
		Task DeleteFileAsync(string fileId, CancellationToken cancellationToken);
	}
```

## ITusReadableStore
Required: no | Tus-Extension: \<none\>

ITusReadableStore is a simple interface that is not part of the tus spec. It is used to help reading data from a data store and making it easier to e.g. download files or process uploaded files. An example of how to use the interface can be found on the [Downloading files](https://github.com/smatsson/tusdotnet/wiki/Downloading-files) page.

```csharp
public interface ITusReadableStore
{
	/// <summary>
	/// Get the file with the specified id. 
	/// Returns null if the file was not found.
	/// </summary>
	/// <param name="fileId">The id of the file to get</param>
	/// <param name="cancellationToken">Cancellation token to use when cancelling</param>
	/// <returns>The file or null if the file was not found</returns>
	Task<ITusFile> GetFileAsync(string fileId, CancellationToken cancellationToken);
}
```

## ITusExpirationStore
Required: no | Tus-Extension: expiration

This interface adds support for the expiration extension allowin the server to remove incomplete files after a period of time. Files that have expired will return 404 by tusdotnet. Files are still accessible for the server using the store's methods.

Read more: http://tus.io/protocols/resumable-upload.html#expiration
Read more on the wiki on how to setup cleanup of expired files: https://github.com/smatsson/tusdotnet/wiki/Removing-expired-incomplete-files

```csharp
public interface ITusExpirationStore
{
	/// <summary>
	/// Set the expiry date of the provided file. 
	/// This method will be called once during creation if absolute expiration is used.
	/// This method will be called once per patch request if sliding expiration is used.
	/// </summary>
	/// <param name="fileId">The id of the file to update the expiry date for</param>
	/// <param name="expires">The datetime offset when the file expires</param>
	/// <param name="cancellationToken">Cancellation token to use when cancelling</param>
	/// <returns>Task</returns>
	Task SetExpirationAsync(string fileId, DateTimeOffset expires, CancellationToken cancellationToken);

	/// <summary>
	/// Get the expiry date of the provided file (set by <code>SetExpirationAsync</code>).
	/// If the datetime offset returned has passed an error will be returned to the client.
	/// If no expiry date exist for the file, this method returns null.
	/// </summary>
	/// <param name="fileId">The id of the file to get the expiry date for</param>
	/// <param name="cancellationToken">Cancellation token to use when cancelling</param>
	/// <returns></returns>
	Task<DateTimeOffset?> GetExpirationAsync(string fileId, CancellationToken cancellationToken);

	/// <summary>
	/// Returns a list of ids of incomplete files that have expired.
	/// This method can be used to do batch processing of incomplete, expired files before removing them.
	/// </summary>
	/// <param name="cancellationToken">Cancellation token to use when cancelling</param>
	/// <returns>A list of ids of incomplete files that have expired</returns>
	Task<IEnumerable<string>> GetExpiredFilesAsync(CancellationToken cancellationToken);

	/// <summary>
	/// Remove all incomplete files that have expired.
	/// </summary>
	/// <param name="cancellationToken">Cancellation token to use when cancelling</param>
	/// <returns>The number of files that were removed</returns>
	Task<int> RemoveExpiredFilesAsync(CancellationToken cancellationToken);
}
```