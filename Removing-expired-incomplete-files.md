If the store supports `ITusExpirationStore` (`TusDiskStore` does), you can specify that incomplete files which have not been updated within a set time period should be flagged as expired. tusdotnet handles the flagging automatically when the `Expiration` property is set on `DefaultTusConfiguration`, but does not delete the files. Deletion must be implemented by the developer.

A common approach is to use an `IHostedService` to periodically clean up expired files, as shown in [this example in the test site](https://github.com/tusdotnet/tusdotnet/blob/master/Source/TestSites/AspNetCore_net6.0_TestApp/Services/ExpiredFilesCleanupService.cs).

> :information_source: The expiration methods only apply to _incomplete_ files. Completed files will not be returned by `GetExpiredFilesAsync` or deleted by `RemoveExpiredFilesAsync`. Completed files need to be deleted manually after processing. Other stores may implement this differently.

```csharp
IEnumerable<string> expiredFileIds = await store.GetExpiredFilesAsync(cancellationToken);
// If you need control over what gets deleted, you can iterate expiredFileIds and delete
// them one by one using ITusTerminationStore.DeleteFileAsync.

int numberOfRemovedFiles = await store.RemoveExpiredFilesAsync(cancellationToken);
// TODO: Do something with numberOfRemovedFiles.
```
