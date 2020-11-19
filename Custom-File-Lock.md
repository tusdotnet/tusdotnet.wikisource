tusdotnet shouldnt write to files that are already in use. To prevent this, tusdotnet uses the a file locking mechanism. In certain situations it can be useful to replace this mechanism with your own implementation.

```csharp
internal sealed class CustomFileLock : ITusFileLock
{
    /// <summary>
    /// Default constructor
    /// </summary>
    /// <param name="fileId">The file id to try to lock.</param>
    public InMemoryFileLock(string fileId)
    {
       _fileId = fileId;
       _hasLock = false;
    }

    /// <summary>
    /// Lock the file. Returns true if the file was locked or false if the file was already locked by another call.
    /// </summary>
    /// <returns>True if the file was locked or false if the file was already locked by another call.</returns>
    public bool Lock()
    {
        // lock file
    }

    /// <summary>
    /// Release the lock if held. If not held by the caller, this method is a no op.
    /// </summary>
    public void ReleaseIfHeld()
    {
        // release file lock
    }
}
```

Use the following option in `DefaultTusConfiguration` to change the file lock to the custom impelmentation
```csharp
AquireFileLock = (fileId) => new CustomFileLock(fileId);
```
