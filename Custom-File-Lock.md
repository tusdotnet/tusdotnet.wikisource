tusdotnet should not write to files that are already in use. To prevent this, tusdotnet uses a file locking mechanism via `ITusFileLockProvider`. Two built-in providers are available: `InMemoryFileLockProvider` (default) and `DiskFileLockProvider`.

To switch to disk-based locks, use the built-in `DiskFileLockProvider`:

```csharp
FileLockProvider = new DiskFileLockProvider(@"C:\tuslocks\")
```

In certain situations it can be useful to replace this mechanism with your own implementation.

```csharp
public sealed class CustomFileLock : tusdotnet.Interfaces.ITusFileLock
{
    private bool _hasLock;
    private readonly string _fileId;

    public CustomFileLock(string fileId)
    {
        _hasLock = false;
        _fileId = fileId;
    }

    /// <summary>
    /// Lock the file. Returns true if the file was locked or false if the file was already locked by another call.
    /// </summary>
    public async Task<bool> Lock()
    {
        // Acquire the lock for _fileId and set _hasLock accordingly, then return _hasLock
    }

    /// <summary>
    /// Release the lock if held. If not held by the caller, this method is a no op.
    /// </summary>
    public async Task ReleaseIfHeld()
    {
        // Release the lock for _fileId if _hasLock is true
    }
}

public sealed class CustomFileLockProvider : tusdotnet.Interfaces.ITusFileLockProvider
{
    public Task<ITusFileLock> AquireLock(string fileId)
    {
        return Task.FromResult<ITusFileLock>(new CustomFileLock(fileId));
    }
}
```

Use the following option in `DefaultTusConfiguration` to change the file lock to the custom implementation:

```csharp
FileLockProvider = new CustomFileLockProvider();
```
