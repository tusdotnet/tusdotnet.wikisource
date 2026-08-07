TusDiskStore is tusdotnet's built-in store. It saves files to a directory on disk.

## Supported extensions

- Creation
- Creation-With-Upload
- Upload-Defer-Length
- Concatenation
- Termination
- Checksum
- Checksum-Trailers
- Expiration

## Basic usage

```csharp
Store = new TusDiskStore(@"C:\tusfiles\")
```

## Options

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `directoryPath` | `string` | - | Path to the directory where files are stored |
| `deletePartialFilesOnConcat` | `bool` | `false` | Delete partial files when a final file is created using the concatenation extension |
| `bufferSize` | `TusDiskBufferSize` | `TusDiskBufferSize.Default` | Read/write buffer sizes. Use `TusDiskBufferSize.Default` or `new TusDiskBufferSize(writeBufferSizeInBytes, readBufferSizeInBytes)` |
| `fileIdProvider` | `ITusFileIdProvider` | GUID-based | Custom [file ID generation](Custom-File-Id-Provider) |

## Complete example

```csharp
Store = new TusDiskStore(
    directoryPath: @"C:\tusfiles\",
    deletePartialFilesOnConcat: true,
    bufferSize: new TusDiskBufferSize(writeBufferSizeInBytes: 1024 * 1024, readBufferSizeInBytes: 51200),
    fileIdProvider: new MyCustomFileIdProvider()
)
```
