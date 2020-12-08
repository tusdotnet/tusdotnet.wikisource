tusdotnet should not write to files that are already in use. To prevent this, tusdotnet uses the a file locking mechanism. In certain situations it can be useful to replace this mechanism with your own implementation.

```csharp
public sealed class CustomFileLock : ITusFileLock
{
    private bool _hasLock;
    private string _fileId;

    /// <summary>
    /// Default constructor
    /// </summary>
    /// <param name="fileId">The file id to try to lock.</param>
    public CustomFileLock(string fileId)
    {
       _hasLock = false;
       _fileId = fileId;
    }

    /// <summary>
    /// Lock the file. Returns true if the file was locked or false if the file was already locked by another call.
    /// </summary>
    /// <returns>True if the file was locked or false if the file was already locked by another call.</returns>
    public async Task<bool> Lock()
    {
        // lock file and set _hasLock
    }

    /// <summary>
    /// Release the lock if held. If not held by the caller, this method is a no op.
    /// </summary>
    public async Task ReleaseIfHeld()
    {
        // release file lock is the lock was aquired in the Lock call
    }
}
```

Use the following option in `DefaultTusConfiguration` to change the file lock to the custom implementation:

```csharp
AquireFileLock = (fileId) => new CustomFileLock(fileId);
```
