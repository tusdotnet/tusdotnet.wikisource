`TusDiskStore` uses file id providers upon creating files to generate valid file ids. The default file id provider is `TusGuidProvider` but you can use a different file id provider by passing it to the constructor of `TusDiskStore`. Tusdotnet is shipped with  the following file id providers: 

 - `TusGuidProvider` generates GUIDs (example id: `def1853a4a72464eb8b0d357c4f6e8a5`, using guildFormat = "n")
 - `TusBase64IdProvider`: generates base64 ids (example id: `KjJA5IoF2Bdi58B7KK5pRw`, using byteLength = 16)
 
Tip: If you want to generate file ids similar to youtubes video ids, use `TusBase64IdProvider(byteLength: 8)`. This will generate file ids that look like this: `xQs3c615q5M`
 
If you want more control over the generated ids, you can create your own file id provider using the `ITusFileIdProvider` interface.
For example you can use your database as a file id provider:

```csharp
public class DatabaseFileIdProvider : ITusFileIdProvider
{
    private readonly ApplicationDbContext _db;

    /// <summary>
    /// Creates a new DatabaseFileIdProvider
    /// </summary>
    public DatabaseFileIdProvider(ApplicationDbContext db)
    {
        _db = db;
    }

    /// <inheritdoc />
    public async Task<string> CreateId(string metadata)
    {
        FileEntity file = new FileEntity();
        _db.Files.Add(file);
        await _db.Files.SaveChangesAsync();
        return file.Id;
    }

    /// <inheritdoc />
    public async Task<bool> ValidateId(string fileId)
    {
        return _db.Files.ExistsAsync(fileId);
    }
}
```

Or you can inherit from another file id provider and just add your database implementation:

```csharp
public class DatabaseGuidProvider : TusGuidProvider
{
    private readonly ApplicationDbContext _db;

    /// <summary>
    /// Creates a new DatabaseFileIdProvider
    /// </summary>
    public DatabaseGuidProvider(ApplicationDbContext db) : base()
    {
        _db = db;
    }

    /// <inheritdoc />
    public override async Task<string> CreateId(string metadata)
    {
        FileEntity file = new FileEntity();
        file.Id = base.CreateId();

        _db.Files.Add(file);
        await _db.Files.SaveChangesAsync();
        return file.Id;
    }

    /// <inheritdoc />
    public override async Task<bool> ValidateId(string fileId)
    {
        return await base.ValidateId(fileId) && await _db.Files.ExistsAsync(fileId);
    }
}
```
