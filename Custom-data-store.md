tusdotnet ships with a single store, the TusDiskStore, which saves files in a directory on disk. You can implement your own store by implementing one or more of the following interfaces. 

tusdotnet will automatically handle requests depending on what interfaces are implemented.

* ITusStore - Support for the core protocol (required)
* ITusCreationStore - Support for the Creation extension (adds "creation" to the "Tus-Extension" header)
* ITusReadableStore - Support for reading files from the store (e.g. for downloads or processing)