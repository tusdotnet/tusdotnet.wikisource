# Deleting processed files when upload is complete

tusdotnet does not automatically delete files after they are processed, it has to be done manually.

When using `TusDiskStore`, it implements `ITusTerminationStore` which allows deleting files.

## Example usage:

```csharp

OnFileCompleteAsync = async ctx =>
{
    ITusFile file = ctx.GetFileAsync();
    var stream = await file.GetContentAsync(eventContext.CancellationToken);
    await WriteFileToOtherDisk(stream);

    // Don't forget to dispose the stream if you're using it!
    await stream.DisposeAsync();

    var terminationStore = (ITusTerminationStore)ctx.Store;
    await terminationStore.DeleteFileAsync(file.Id, ctx.CancellationToken);
}
```