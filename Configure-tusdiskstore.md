tusdotnet ships with a single built in store that saves files to disk. It is simply configured by providing a directory on disk where to store files:
```csharp
Store = new TusDiskStore(@"C:\tusfiles\", deletePartialFilesOnConcat: true)
```

Optional features provided to the constructor:

* `deletePartialFilesOnConcat` - `true` to delete all partial files when a `final` file is created using the [concatenation extension](https://tus.io/protocols/resumable-upload.html#concatenation). Defaults to `false`.
* `bufferSize` - Used to configure the read and write buffer used by the store.
* `fileIdProvider` - The file id provider to use when creating files. The provider gives support for custom file ids besides from the built in IDs based on GUIDs.