`TusDiskStore` uses file ID providers upon creating files to generate valid file IDs. The default file ID provider is `GuidFileIdProvider` but you can use a different file ID provider by passing it to the constructor of `TusDiskStore`. tusdotnet ships with the following file ID providers:

 - `GuidFileIdProvider` generates GUIDs (example ID: `def1853a4a72464eb8b0d357c4f6e8a5`, using guidFormat = "n")
 - `Base64FileIdProvider`: generates base64 IDs (example ID: `KjJA5IoF2Bdi58B7KK5pRw`, using byteLength = 16)
 
Tip: If you want to generate file IDs similar to YouTube's video IDs, use `Base64FileIdProvider(byteLength: 8)`. This will generate file IDs that look like this: `xQs3c615q5M`
 
If you want more control over the generated IDs, you can create your own file ID provider using the `ITusFileIdProvider` interface.

For example you can use your database as a file id provider:

```csharp
public class DatabaseFileIdProvider : tusdotnet.Interfaces.ITusFileIdProvider
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
        await _db.SaveChangesAsync();
        return file.Id;
    }

    public async Task<bool> ValidateId(string fileId)
    {
        return await _db.Files.AnyAsync(f => f.Id == fileId);
    }
}
```

Or you can inherit from another file id provider and just add your own implementation:

```csharp
public class DatabaseGuidProvider : tusdotnet.Stores.FileIdProviders.GuidFileIdProvider
{
    private readonly ApplicationDbContext _db;

    /// <summary>
    /// Creates a new DatabaseGuidProvider
    /// </summary>
    public DatabaseGuidProvider(ApplicationDbContext db) : base()
    {
        _db = db;
    }

    /// <inheritdoc />
    public override async Task<string> CreateId(string metadata)
    {
        FileEntity file = new FileEntity();
        file.Id = await base.CreateId(metadata);

        _db.Files.Add(file);
        await _db.SaveChangesAsync();
        return file.Id;
    }

    /// <inheritdoc />
    public override async Task<bool> ValidateId(string fileId)
    {
        return await base.ValidateId(fileId) && await _db.Files.AnyAsync(f => f.Id == fileId);
    }
}
```
