# Deleting processed files when upload is complete

tusdotnet does not automatically delete files after they are processed, it has to be done manually.

Deleting files requires that the store implements `ITusTerminationStore` (`TusDiskStore` does). If you are unsure whether the configured store supports it, check before casting.

## Example usage:

```csharp

OnFileCompleteAsync = async ctx =>
{
    ITusFile file = await ctx.GetFileAsync();

    using var stream = await file.GetContentAsync(ctx.CancellationToken);
    await WriteFileToOtherDisk(stream);

    if (ctx.Store is ITusTerminationStore terminationStore)
    {
        await terminationStore.DeleteFileAsync(file.Id, ctx.CancellationToken);
    }
}
```