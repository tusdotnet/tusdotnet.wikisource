## Leased file locks

A distributed file lock can be lost in the middle of a request, for example when its lease expires during a process pause or when the lock backend fails over. Another request can then take the lock while the first one is still writing. Implement `ITusLeasedFileLock` on such a lock so tusdotnet can react.

### Implementing the lock

`ITusLeasedFileLock` extends `ITusFileLock` with one property:

- `LockLostToken`: cancelled when this instance loses the lock it acquired in `Lock()`. Return `CancellationToken.None` while the lock is not held.

tusdotnet still calls `ReleaseIfHeld()` after the lock is lost. Release the lock only if this instance still owns it, for example by comparing the lease owner or fencing token. A release that ignores ownership deletes the lock that the other request now holds, and a third request can then write at the same time.

```csharp
public async Task ReleaseIfHeld()
{
    // Delete the lock only if it still carries our owner id.
    await _backend.DeleteIfOwnerAsync(_lockKey, _ownerId);
}
```

### What tusdotnet does once the lock is lost

The request body is guarded: the next read throws, a read waiting on a slow client is cancelled, and on `PipeReader` `TryRead` and `AdvanceTo` throw too. The request answers `409 Conflict`, and the client sends `HEAD` and resumes from the stored offset. A store needs no code change to get this.

**This is not full protection.** The guard stops the store from reading more of the body. It cannot undo data the store wrote before the lock was lost, and it cannot act after the store's last read. If your store writes or commits after reading, a stale write can still land.

### Fencing writes in the store

A store that must never persist a stale write checks for itself:

- `GetFileLockLostToken()` on the `Stream` or `PipeReader` passed to `AppendDataAsync` returns the lock-lost token, or `CancellationToken.None` when the lock cannot be lost. Call it on that exact object, not on a wrapper the store creates around it.
- Checking the token right before a commit is cheap, but the lock can still be lost right after the check. The reliable protection is a conditional write against the storage, such as an ETag `If-Match` or a fencing token.
- When that write is rejected, throw `TusUploadConflictException`. tusdotnet answers `409 Conflict`.

```csharp
public async Task<long> AppendDataAsync(string fileId, PipeReader reader, CancellationToken cancellationToken)
{
    var lockLost = reader.GetFileLockLostToken();
    var file = await _storage.GetAsync(fileId, cancellationToken);

    while (true)
    {
        var result = await reader.ReadAsync(cancellationToken);
        await _storage.StageAsync(file, result.Buffer, cancellationToken);
        reader.AdvanceTo(result.Buffer.End);

        if (result.IsCompleted) break;
    }

    lockLost.ThrowIfCancellationRequested();

    if (!await _storage.TryCommitAsync(file, ifMatch: file.ETag, cancellationToken))
        throw new TusUploadConflictException("The upload was modified by another request. Please try again");

    return file.Length;
}
```