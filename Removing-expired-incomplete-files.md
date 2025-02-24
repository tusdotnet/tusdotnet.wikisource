If the store being used supports `ITusExpirationStore` (`TusDiskStore` does) you can specify that incomplete files that have not been updated in a set time period should be flagged as expired. This is done automatically by tusdotnet if the Expiration-property is set on the `ITusConfiguration` and if the store supports `ITusExpirationStore`. Files are however not deleted automatically. To help with deleting expired incomplete files the `ITusExpirationStore` interface exposes two methods, `GetExpiredFilesAsync` and `DeleteExpiredFilesAsync`. The former is used to get a list of ids of files that have expired and the latter is used to remove the expired files.

tusdotnet does not automatically remove expired files so this needs to be implemented by the developer. This can be achieved by e.g. a `IHostedService` as in [this example in the .NET 6 test site](https://github.com/tusdotnet/tusdotnet/blob/master/Source/TestSites/AspNetCore_net6.0_TestApp/Services/ExpiredFilesCleanupService.cs).

> :information_source: The methods described on this page only applies to _incomplete_ expired files as this is how `TusDiskStore` is implemented. Files that are completed will not be returned by `GetExpiredFilesAsync` nor deleted by `RemoveExpiredFilesAsync`. Completed files needs to be deleted manually after any processing is done. Other stores might implement this differently.

Example usage:
```csharp

IEnumerable<string> expiredFileIds = await tusDiskStore.GetExpiredFilesAsync(cancellationToken);
// TODO: Do something with expiredFileIds.

int numberOfRemovedFiles = await tusDiskStore.RemoveExpiredFilesAsync(cancellationToken);
// TODO: Do something with numberOfRemovedFiles.
```